These are some installation notes taken in the process of installing WRF version 3.8 on a computer with Ubuntu Server 16.04 LTS.

## Install required software
```console
$ sudo apt−get install build−essential csh gfortran m4
```

## System environment tests

First and foremost, it is very important to have a gfortran compiler, as well as gcc and cpp. If you have these installed, you should be given a path for the location of each.
```console
$ which gfortran
/user/bin/gfortran
$ which cpp
/user/bin/cpp
$ which gcc
/user/bin/gcc
```

Check your gcc version. It is recommend using version 4.4.0 or later.
```console
$ gcc −−version
gcc (Ubuntu 5.3.1−14ubuntu2.1) 5.3.1 20160413
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions. There is NO
warranty; not even for MERCHANTABILITY of FITNESS FOR A PARTICULAR PURPOSE.
```

Create a new, clean directory called `Build_WRF`, and another one called `TESTS`.

There are a few simple tests that can be run to verify that the fortran compiler is built properly, and that it is compatible with the C compiler. Download the tar file that contains the tests into the `TESTS` directory and unpack the tar file.
```console
$ cd {path_to_dir}/TESTS
$ wget http ://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/Fortran_C_tests.tar
$ tar −xvf Fortran_C_tests.tar
```

There are 7 tests available, so start at the top and run through them, one at a time.

* Test 1: Fixed Format Fortran Test.
```console
$ gfortran TEST_1_fortran_only_fixed.f
$ ./a.out
SUCCESS test 1 fortran only fixed format
```

* Test 2: Free Format Fortran.
```console
$ gfortran TEST_2_fortran_only_free.f90
$ ./a.out
Assume Fortran 2003: has FLUSH, ALLOCATABLE, derived type, and ISO C Binding
SUCCESS test 2 fortran only free format
```

* Test 3: C.
```console
$ gcc TEST_3_c_only.c
$ ./a.out
SUCCESS test 3 c only
```

* Test 4: Fortran Calling a C Function (our gcc and gfortran have different defaults, so we force both to always use 64 bit [-m64] when combining them).
```console
$ gcc −c −m64 TEST_4_fortran+c_c.c
$ gfortran −c −m64 TEST_4_fortran+c_f.f90
$ gfortran −m64 TEST_4_fortran+c_f.o TEST_4_fortran+c_c.o
$ ./a.out
C function called by Fortran Values are xx = 2.00 and ii = 1
SUCCESS test 4 fortran calling c
```

In addition to the compilers required to manufacture the WRF executables, the WRF build system has scripts as the top level for the user interface. The WRF scripting system uses, and therefore is necessary having csh, perl and sh.
To test whether these scripting languages are working properly on the system, there are 3 tests to run. These tests were included in the "Fortran and C Tests Tar File".

* Test 5: csh.
```console
$ csh ./TEST_csh.csh
SUCCESS csh test
```

* Test 6: perl.
```console
$ ./TEST_perl.pl
SUCCESS perl test
```

* Test 7: sh.
```console
$ ./TEST_sh.sh
SUCCESS sh test
```

## Building libraries

Before getting started, you need to make another directory. Go inside your `Build_WRF` directory and then make a directory called `LIBRARIES`.

```console
$ cd {path_to_dir}/Build_WRF
$ mkdir LIBRARIES
```

Depending on the type of run you wish to make, there are various libraries that should be installed. Go inside your `LIBRARIES` directory and then download all 5 tar files.

```console
$ cd {path_to_dir}/Build_WRF/LIBRARIES
$ wget http://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/mpich−3.0.4.tar.gz
$ wget http://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/netcdf−4.1.3.tar.gz
$ wget http://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/jasper−1.900.1.tar.gz
$ wget http://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/libpng −1.2.50.tar.gz
$ wget http://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/zlib −1.2.7.tar.gz
```

**It is important to note that these libraries must all be installed with the same compilers as will be used to install WRFV3 and WPS.**

Configuring NetCDF library: This library is always necessary! Modify the `.bashrc` file in the home directory of current user to set the environment variables.

```console
$ sudo nano ~/.bashrc
```

At the bottom of the file add these lines so that they will be set for future logins.

```console
export DIR={path_to_dir}/Build_WRF/LIBRARIES
export CC=gcc
export CXX=g++
export FC=gfortran
export FCFLAGS=−m64
export F77=gfortran
export FFLAGS=−m64
```

Then source the file to make these settings active for current session.

```console
$ source ~/.bashrc
```

Unpack the `netcdf-4.1.3.tar.gz` file.

```console
$ tar −zxvf netcdf−4.1.3.tar.gz
```

Go into the `netcdf-4.1.3` directory and run the configure script with the parameters presented below, make and make install.

```console
$ cd {path_to_dir}/Build_WRF/LIBRARIES/netcdf-4.1.3
$ ./configure --prefix=$DIR/netcdf --disable-dap --disable-netcdf-4 --disable-shared
$ make
$ make install
```

Modify again the `.bashrc` file and set two new environment variables at the bottom. Then source the file to make these settings active for current session and leave the directory.

```console
$ sudo nano ~/.bashrc

export PATH=$DIR/netcdf/bin:$PATH
export NETCDF=$DIR/netcdf

$ source ~/.bashrc
$ cd ..
```

Configuring MPICH library: This library is necessary if you are planning to build WRF in parallel. If your machine does not have more than 1 processor, or if you have no need to run WRF with multiple processors, you can skip installing MPICH.

In principle, any implementation of the MPI-2 standard should work with WRF; however, we have the most experience with MPICH, and therefore, that is what will be described here.

Assuming all the **export** commands were already issued while setting up NetCDF, you can continue on to install MPICH, issuing each of the following commands.

```console
$ cd {path_to_dir}/Build_WRF/LIBRARIES
$ tar -zxvf mpich-3.0.4.tar.gz
$ cd {path_to_dir}/Build_WRF/LIBRARIES/mpich-3.0.4
$ ./configure --prefix=$DIR/mpich
$ make
$ make install
$ sudo nano ~/.bashrc

export PATH=$DIR/mpich/bin:$PATH

$ source ~/.bashrc
$ cd ..
```

Configuring zlib: This is a compression library necessary for compiling WPS (specifically ungrib) with GRIB2 capability.
Assuming all the **export** commands from the NetCDF install are already set, you can move on to the commands to install zlib.

```console
$ cd {path_to_dir}/Build_WRF/LIBRARIES
$ sudo nano ~/.bashrc

export LDFLAGS=-L$DIR/grib2/lib 
export CPPFLAGS=-I$DIR/grib2/include 

$ source ~/.bashrc
$ tar -zxvf zlib-1.2.7.tar.gz
$ cd {path_to_dir}/Build_WRF/LIBRARIES/zlib-1.2.7
$ ./configure --prefix=$DIR/grib2
$ make
$ make install
$ cd ..
```

Configuring libpng: This is a compression library necessary for compiling WPS (specifically ungrib) with GRIB2 capability.
Assuming all the **export** commands from the NetCDF install are already set, you can move on to the commands to install libpng.

```console
$ cd {path_to_dir}/Build_WRF/LIBRARIES
$ tar -zxvf libpng-1.2.50.tar.gz
$ cd {path_to_dir}/Build_WRF/LIBRARIES/libpng-1.2.50
$ ./configure --prefix=$DIR/grib2
$ make
$ make install
$ cd ..
```

Configuring JasPer: This is a compression library necessary for compiling WPS (specifically ungrib) with GRIB2 capability.
Assuming all the **export** commands from the NetCDF install are already set, you can move on to the commands to install jasper.

```console
$ cd {path_to_dir}/Build_WRF/LIBRARIES
$ tar -zxvf jasper-1.900.1.tar.gz
$ cd {path_to_dir}/Build_WRF/LIBRARIES/jasper-1.900.1
$ ./configure --prefix=$DIR/grib2
$ make
$ make install
$ cd ..
```
