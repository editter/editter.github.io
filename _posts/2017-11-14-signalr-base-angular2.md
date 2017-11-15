---
title:        Using base classes in SignalR 2 and Angular 2
author:       Eric Ditter
date:         2017-11-15
categories: typescript
excerpt_separator: <!--more-->
---

On one of my projects I used SignalR pretty extensivly in an Angular 1.6 application and I was using the [angular-signalr-hub](https://github.com/justmaier/angular-signalr-hub) library to integrate it into my application. It worked very well but I am moving to Angular 2 so I needed to find a way to do it without having to use the library and I was hoping to get to a more object oriented way of doing it.

With Typescript you can use base classes so I ended up coming up with the following and overall I really like how it came out. Everything that comes from the server passes through here so I can intercept whatever I need which actually came in handy. For date values I had some instances where they came through as strings and arrays came through as [array-like objects](http://www.nfriedly.com/techblog/2009/06/advanced-javascript-objects-arrays-and-array-like-objects/). I just parsed them internally and my calling code know the difference.
<!--more-->
## SignalrBase.ts

``` typescript
/// <reference types="signalr" />

import { NgZone } from '@angular/core';
import { Subject } from 'rxjs/Subject';
import { Observable } from 'rxjs/Observable';
import { BehaviorSubject } from 'rxjs/BehaviorSubject';
import * as moment from 'moment';

export enum ConnectionState {
  Connecting = 0,
  Connected = 1,
  Reconnecting = 2,
  Disconnected = 4
}

export abstract class SignalrBase {

  // static so I can have a shared connection
  private static connections: { [url: string]: SignalR.Hub.Connection } = {};
  [propertyName: string]: any;

  // These are used to feed the public observables
  public error$ = new Observable<SignalR.ConnectionError>();
  private connectionState$ = new Observable<ConnectionState>();

  // These are used to track the internal SignalR state
  private callbackQueue: (() => void)[] = [];
  private connection: SignalR.Hub.Connection;
  private proxy: SignalR.Hub.Proxy;
  private connectionStateSubject = new BehaviorSubject<ConnectionState>(ConnectionState.Disconnected);
  private errorSubject = new Subject<SignalR.ConnectionError>();

  // tslint:disable-next-line:max-line-length
  private datePattern: RegExp = new RegExp(/(\d{4}-[01]\d-[0-3]\dT[0-2]\d:[0-5]\d:[0-5]\d\.\d+)|(\d{4}-[01]\d-[0-3]\dT[0-2]\d:[0-5]\d:[0-5]\d)|(\d{4}-[01]\d-[0-3]\dT[0-2]\d:[0-5]\d)/);

  protected constructor(
    private hubName: string,
    private listenerNames: string[],
    private methodNames: string[],
    private ngZone: NgZone,
  ) {

    // Set up our observables
    this.connectionState$ = this.connectionStateSubject.asObservable();
    this.error$ = this.errorSubject.asObservable();

    this.connection = SignalrService.InitConnection('/MySite/SignalR');
    this.proxy = this.connection.createHubProxy(hubName);

    // Define handlers for the connection state events
    //
    this.connection.stateChanged(state => {

      switch (state.newState) {
        case ConnectionState.Connected:
          while (this.callbackQueue.length > 0) {
            const callback = this.callbackQueue.pop();
            if (callback instanceof Function) {
              callback();
            }
          }

          break;
        case ConnectionState.Disconnected:
          const timeoutKey = setInterval(async (intervalState: { attempt: number }) => {
            if (intervalState.attempt > 30) {
              clearInterval(timeoutKey);
              console.error(`[SignalR][${this.hubName}] Clearing Timeout for disconnect`);
              return;
            }

            try {
              await this.connection.start();
            } catch (err) {
              console.error(`[SignalR][${this.hubName}]`, err);

            }

            intervalState.attempt++;

          }, 10000, { attemptNum: 0 }); // Try to restart connection after 10 seconds

          break;

      }

      this.connectionStateSubject.next(state.newState);

    });

    // Define handlers for any errors
    //
    this.connection.error(error => {
      console.error(`[SignalR][${this.hubName}]`, error);
      this.errorSubject.next(error);

    });

    this.listenerNames.forEach(listenerName => this.buildListener(listenerName));
    this.methodNames.forEach(methodName => this.buildMethod(methodName));

  }

  private static InitConnection(url: string) {
    const jq = (window as any).jQuery as SignalR;
    if (typeof jq === 'undefined') {
      throw new Error(`The variable "jQuery" is not defined...please check that jQuery has been loaded properly`);

    } else if (!(jq.hubConnection instanceof Function)) {
      throw new Error(`The 'jQuery.hubConnection()' function is not defined...please check that SignalR has been loaded properly`);

    }

    // we do this so the connection can be shared per URL
    if (typeof this.connections[url] === 'undefined') {
      this.connections[url] = jq.hubConnection(url);
      this.connections[url].start()
        .done(d => {
        })
        .fail(error => {
          console.error(error);

        });

    }

    return this.connections[url];

  }

  private buildMethod(methodName: string) {

    this[methodName] = (...args: any[]) => {
      const results = new Subject<any>();
      const invokeCall = () => {

        this.proxy.invoke(methodName, ...args)
          .done((params: any) => {
            this.ngZone.run(() => results.next(this.convertProperties(params)));
          })
          .fail((error: any) => {
            this.ngZone.run(() => results.error(error));
          })
          .always(() => {
            results.complete();
          });

      };

      if (this.connectionState$.getValue() !== ConnectionState.Connected) {
        this.callbackQueue.push(invokeCall);
      } else {
        invokeCall();
      }

      return results.asObservable();
    };

  }

  private buildListener(listenerName: string) {
    const methodName = 'on' + listenerName.charAt(0).toUpperCase() + listenerName.slice(1);
    const results = new Subject<any>();

    this.proxy.on(listenerName, resultData => {
      resultData = this.convertProperties(resultData);
      this.ngZone.run(() => results.next(resultData));
    });

    this[methodName] = results.asObservable();
  }

  private convertProperties = (params: any): any => {
    // this logic is here in an attempt to automatically parse dates and array like values to arrays
    // if you are wondering what array like values are then just know that it
    // isn't an instance of array but it is Array.isArray (stupid IE)
    if (!(params instanceof Array) && Array.isArray(params)) {
      params = Object.keys(params).map((key: any) => params[key]);
    }

    for (const propertyName in params) {
      if (params.hasOwnProperty(propertyName)) {
        let propertyValue = params[propertyName];

        if (!propertyValue) {
          continue;
        } else if (propertyValue instanceof Array || Array.isArray(propertyValue)
          || propertyValue instanceof Object || typeof propertyValue === 'object') {
          // pass this in so we can check the array values or object properties
          propertyValue = this.convertProperties(propertyValue);

        } else {

          try {

            // check that the string is a match for the date regex
            // we need to use the regex because momentjs will look at integers as valid dates
            // and we only want to parse the string versions
            const isMatch = this.datePattern.exec(propertyValue);
            const parsedDate = moment(propertyValue);
            if (isMatch && isMatch.length > 0 && parsedDate.isValid()) {
              params[propertyName] = parsedDate.toDate();

            }

          } catch (err) {
            console.warn(`Error trying to auto parse date result. Method: '${name}',Property: '${propertyName}', Value: '${propertyValue}'`, err);

          }

        }

      }
    }

    return params;

  }

}

```

This is my implementation class which is pretty barebone. The only thing I wasn't able to figure out was getting NgZone injected directly into the base class so I pass it from the impementing class. It isn't perfect but it is pretty minor to have one extra parameter. If you aren't using Angular 2 then you can remove all references to NgZone and be fine, it is only used to trigger the UI to update.

## data.service.ts

``` typescript
import { Injectable, NgZone } from '@angular/core';
import { Observable } from 'rxjs/Observable';
import { SignalrService } from './signalr.service.base';

@Injectable()
export class DataService extends SignalrService {
  // Listeners
  public onMessage: Observable<{ username: string, text: string }>;

  // Getters
  public getData: () => Observable<string[]>;

  constructor(ngZone: NgZone) {
    super('CrewHub',
      [
        // this will be converted to onMessage in the base class
        'message',
      ],
      [
        'getData',
      ], ngZone);

  }

}
```

I don't know a lot of the nuances in Angular 2 so maybe this isn't the "correct" way to do it but in my case it worked pretty well.
