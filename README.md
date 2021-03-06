pidusage
========

[![Build Status](https://travis-ci.org/soyuka/pidusage.svg?branch=master)](https://travis-ci.org/soyuka/pidusage)
[![Build status](https://ci.appveyor.com/api/projects/status/dqs82fp92pf2rey5)](https://ci.appveyor.com/project/soyuka/pidusage)

Process cpu % and memory use of a PID

Ideas from https://github.com/arunoda/node-usage/ but with no C-bindings

# API

```
var pusage = require('pidusage')

pusage.stat(process.pid, function(err, stat) {

	expect(err).to.be.null
	expect(stat).to.be.an('object')
	expect(stat).to.have.property('cpu')
	expect(stat).to.have.property('memory')

	console.log('Pcpu: %s', stat.cpu)
	console.log('Mem: %s', stat.memory) //those are bytes

})

// Unmonitor process
pusage.unmonitor(18902);
```

# What do this script do?

A check on the `os.platform` is done to determine the method to use.

### Linux
We use `/proc/{pid}/stat` in addition to the the `PAGE_SIZE` and the `CLK_TCK` direclty from `getconf()` command. Uptime comes from `proc/uptime` file because it's more accurate than the nodejs `os.uptime()`.

/!\ As stated in [#17](https://github.com/soyuka/pidusage/issues/17), memory will increase when using `pidusage.stat` in an interval because of `readFile`. Use `--expose-gc` and release the garbage collector to avoid such leaking. 

### On darwin, freebsd, solaris (tested on 10/11)
We use a fallback with the `ps -o pcpu,rss -p PID` command to get the same informations.

### On AIX
AIX is tricky because I have no AIX test environement, at the moment we use: `ps -o pcpu,rssize -p PID` but `/proc` results should be more accurate! If you're familiar with the AIX environment and know how to get the same results as we've got with Linux systems, please help.
[#4](https://github.com/soyuka/pidusage/issues/4)

### Windows
Windows is really tricky, atm it uses the `wmic.exe`, feel free to share ideas on how to improve this.
More specifically, thanks to [@crystaldust](https://github.com/crystaldust) we replaced `wmic PROCESS` by `wmic path Win32_PerfFormattedData_PerfProc_Process` to get more accurated data ([PR](https://github.com/soyuka/pidusage/pull/16), [commit](https://github.com/soyuka/pidusage/commit/2c8d47d2365590684e5998be33d9ce05af5ab8f3)).

# Licence

MIT
