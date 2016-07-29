These are some installation notes taken in the process of installing WRF version 3.8 on a computer with Ubuntu Server 16.04 LTS.

## Install required software

```ShellSession
$ sudo apt−get install build−essential csh gfortran m4
```

## System environment tests

First and foremost, it is very important to have a gfortran compiler, as well as gcc and cpp. If you have these installed, you should be given a path for the location of each.

```ShellSession
$ which gfortran
/user/bin/gfortran
$ which cpp
/user/bin/cpp
$ which gcc
/user/bin/gcc
```

Check your gcc version. It is recommend using version 4.4.0 or later.

```ShellSession
$ gcc −−version
gcc (Ubuntu 5.3.1−14ubuntu2.1) 5.3.1 20160413
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY of FITNESS FOR A PARTICULAR PURPOSE.
```

Create a new, clean directory called `Build_WRF`, and another one called `TESTS`.
