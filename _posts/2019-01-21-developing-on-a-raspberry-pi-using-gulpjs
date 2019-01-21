---
title:        Developing on a Raspberry Pi using Gulpjs
author:       Eric Ditter
date:         2019-1-21
tags: [Raspberry Pi, GulpJs, Dev]
excerpt_separator: <!--more-->
---

Developing on one machine and running on another is a tedious process but sometimes you need to when a library has different features for ARM vs x64 and the then there is always the Windows vs Linux issues.  This was the issue I had when I was working on a Raspberry Pi project using Python, I got everything working on Windows and then copied it over to the Pi thinking the libraries would work the same between the two but they didn't.
<!--more-->
After copying & pasting to a share a few times I realized that it was kind of annoying having to move my files to the share, kill the terminal and re-run the start command so I turned to everyone's favorite task runner, [Gulp](https://gulpjs.com/){:target="_blank"} and it was like greeting an old friend.

In order for this to work with a remote Linux machine, you need to install [Putty](https://putty.org/){:target="_blank"} which is a common SSH and telnet client for Windows.  Once you install that you get a few programs that will allow you to easily copy files and run remote commands on your connected machine through a command line.  There may be more than one way to do this same thing but this is just how I ended up doing it

**Note: If ```plink``` and ```pscp``` aren't in your PATH they are located inside ```C:\Program Files\PuTTY\```**

The folder structure of the project was pretty simple

<pre>
&#8866; gulpfile.js
&#8866; src
  &#8866; main.py
  &#8866; lib.py
</pre>

Then this is where the magic happened.

gulpfile.js
```javascript
const
  gulp = require("gulp"),
  process = require("child_process");

const
  piName = '192.168.1.5',
  piUsr = 'myUser',
  piPwd = 'securePassword';

const spawnTask = (command, doneCallback) => {
  // run the task through the command line
  const proc = process.spawn("cmd.exe", ['/c', command])
  // log the output and errors to get some insights into what's going on
  proc.stdout.on('data', (output) => console.log(output.toString()));
  proc.stderr.on('data', (output) => console.log(output.toString()));
  // when the command line is done let the calling gulp taks know
  proc.once('close', doneCallback);
}

gulp.task("start", (done) => {
  // run the python command on the Pi
  // https://the.earth.li/~sgtatham/putty/0.70/htmldoc/Chapter7.html#plink
  spawnTask(`plink -ssh -l ${piUsr} -pw ${piPwd} "sudo python /home/pi/share/my_app/main.py"`, done);
});

gulp.task("move", (done) => {
  // copy the files from src to the working directory of the pi
  // https://the.earth.li/~sgtatham/putty/0.70/htmldoc/Chapter5.html#pscp
  spawnTask(`pscp -pw ${piPwd} -r ./src/*.* ${piUsr}@${piName}:/home/pi/share/my_app`, done);
});

// gulp 4 syntax - npm install gulp@next
// if you are using gulp 3 check out the npm package run-sequence
gulp.task('default', () => {
  return gulp.watch(['./src/*.py'], gulp.series('move', 'start'))
});
```

As you can see it doesn't take much to accomplish this but child_process was something I had never used before this but it has come in handy a few times since then.  Now when I develop on the Pi I can just run the default gulp task and everything copies over and runs pretty seamlessly.
