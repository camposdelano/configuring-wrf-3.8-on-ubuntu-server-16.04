These are some installation notes taken in the process of installing WRF version 3.8 on a computer with Ubuntu Server 16.04 LTS.

## Install required software

```console
$ sudo apt−get install build−essential csh gfortran m4
```

## System environment tests

* _A video of this part is available [here](https://www.youtube.com/watch?v=_9lBM4k7HQc)._

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

* _A video of this part is available [here](https://www.youtube.com/watch?v=Ipd8vkAj8Fk)._

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

(...)
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

(...)
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

(...)
export PATH=$DIR/mpich/bin:$PATH

$ source ~/.bashrc
$ cd ..
```

Configuring zlib: This is a compression library necessary for compiling WPS (specifically ungrib) with GRIB2 capability.
Assuming all the **export** commands from the NetCDF install are already set, you can move on to the commands to install zlib.
```console
$ cd {path_to_dir}/Build_WRF/LIBRARIES
$ sudo nano ~/.bashrc

(...)
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

## Libraries compatibility tests

* _A video of this part is available [here](https://www.youtube.com/watch?v=j205Ki84ZF4)._

Once the target machine is able to make small Fortran and C executables (what was verified in the System Environment Tests section), and after the NetCDF and MPI libraries are constructed (two of the libraries from the Building Libraries section), to emulate the WRF code's behavior, two additional small tests are required. We need to verify that the libraries are able to work with the compilers that are to be used for the WPS and WRF builds.

Move to `TESTS` directory, download the tar file that contans these tests and unpack it.
```console
$ cd {path_to_dir}/TESTS
$ wget http://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/Fortran_C_NETCDF_MPI_tests.tar
$ tar -xvf Fortran_C_NETCDF_MPI_tests.tar
```

There are 2 tests.

* Test 1: Fortran + C + NetCDF

The NetCDF-only test requires the include file from the NETCDF package be in this directory. Copy the NetCDF include here and compile the Fortran and C codes for the purpose of this test (the -c option says to not try to build an executable).
```console
$ cp ${NETCDF}/include/netcdf.inc .
$ gfortran -c 01_fortran+c+netcdf_f.f
$ gcc -c 01_fortran+c+netcdf_c.c
$ gfortran 01_fortran+c+netcdf_f.o 01_fortran+c+netcdf_c.o -L${NETCDF}/lib -lnetcdff -lnetcdf
$ ./a.out
```

The following should be displayed on your screen.
```console
C function called by Fortran
Values are xx = 2.00 and ii = 1
SUCCESS test 1 fortran + c + netcdf
```

* Test 2: Fortran + C + NetCDF + MPI

The NetCDF+MPI test requires include files from both of these packages be in this directory, but the MPI scripts automatically make the `mpif.h` file available without assistance, so no need to copy that one. Copy the NetCDF include file here and note that the MPI executables `mpif90` and `mpicc` are used below when compiling. Issue the following commands.
```console
$ cp ${NETCDF}/include/netcdf.inc .
$ mpif90 -c 02_fortran+c+netcdf+mpi_f.f
$ mpicc -c 02_fortran+c+netcdf+mpi_c.c
$ mpif90 02_fortran+c+netcdf+mpi_f.o 02_fortran+c+netcdf+mpi_c.o -L${NETCDF}/lib -lnetcdff -lnetcdf
$ mpirun ./a.out
```

The following should be displayed on your screen.
```console
C function called by Fortran
Values are xx = 2.00 and ii = 1
status = 2
SUCCESS test 2 fortran + c + netcdf + mpi
```

## Building WRFV3

* _A video of this part is available [here](https://www.youtube.com/watch?v=hkLrdlQnKTw)._

After ensuring that all libraries are compatible with the compilers, you can now prepare to build WRFV3. If you do not already have a `WRFV3` tar file, move to your `Build_WRF` directory, download that file and unpack it. Then go into the `WRFV3` directory and create a configuration file for your computer and compiler.
```console
$ cd {path_to_dir}/Build_WRF
$ wget http://www2.mmm.ucar.edu/wrf/src/WRFV3.8.TAR.gz
$ tar -zxvf WRFV3.8.TAR.gz
$ cd {path_to_dir}/Build_WRF/WRFV3
$ ./configure
```

You will see various options. Choose the option that lists the compiler you are using and the way you wish to build WRFV3 (i.e., serially or in parallel). Although there are 3 different types of parallel (smpar, dmpar, and dm+sm), it is recommend choosing dmpar option.
```console
checking for perl5... no
checking for perl... found /usr/bin/perl (perl)
Will use NETCDF in dir: /usr
HDF5 not set in environment. Will configure WRF for use without.
PHDF5 not set in environment. Will configure WRF for use without.
Will use 'time' to report timing information
$JASPERLIB or $JASPERINC not found in environment, configuring to build without grib2 I/O...
-----------------------------------------------------------------------------------------------
Please select from among the following Linux x86_64 options:

 1. (serial)  2. (smpar)  3. (dmpar)  4. (dm+sm) PGI (pgf90/gcc)
 5. (serial)  6. (smpar)  7. (dmpar)  8. (dm+sm) PGI (pgf90/pgcc): SGI MPT
 9. (serial) 10. (smpar) 11. (dmpar) 12. (dm+sm) PGI (pgf90/gcc): PGI accelerator
13. (serial) 14. (smpar) 15. (dmpar) 16. (dm+sm) INTEL (ifort/icc)
                                     17. (dm+sm) INTEL (iforticc): Xeon Phi (MIC architecture)
18. (serial) 19. (smpar) 20. (dmpar) 21. (dm+sm) INTEL (ifort/icc): Xeon (SNB with AVX mods)
22. (serial) 23. (smpar) 24. (dmpar) 25. (dm+sm) INTEL (ifort/icc): SGI MPT
26. (serial) 27. (smpar) 28. (dmpar) 29. (dm+sm) INTEL (ifort/icc): IBM POE
30. (serial)             31. (dmpar)             PATHSCALE (pathf90/pathcc)
32. (serial) 33. (smpar) 34. (dmpar) 35. (dm+sm) GNU (gfortran/gcc)
36. (serial) 37. (smpar) 38. (dmpar) 39. (dm+sm) IBM (xlf90_r/cc_r)
40. (serial) 41. (smpar) 42. (dmpar) 43. (dm+sm) PGI (ftn/gcc): Cray XC CLE
44. (serial) 45. (smpar) 46. (dmpar) 47. (dm+sm) CRAY CCE (ftn/cc): Cray XE and XC
48. (serial) 49. (smpar) 50. (dmpar) 51. (dm+sm) INTEL (ftn/icc): Cray XC
52. (serial) 53. (smpar) 54. (dmpar) 55. (dm+sm) PGI (pgf90/pgcc)
56. (serial) 57. (smpar) 58. (dmpar) 59. (dm+sm) PGI (pgf90/gcc): -f90=pgf90
60. (serial) 61. (smpar) 62. (dmpar) 63. (dm+sm) PGI (pgf90/pgcc): -f90=pgf90
64. (serial) 65. (smpar) 66. (dmpar) 67. (dm+sm) INTEL (ifort/icc): HSW/BDW
68. (serial) 69. (smpar) 70. (dmpar) 71. (dm+sm) INTEL (ifort/icc): KNL MIC

Enter selection [1-71] : 34
-----------------------------------------------------------------------------------------------
Compile for nesting? (1=basic, 2=preset moves, 3=vortex following) [default 1]: 1

Configuration successful!
-----------------------------------------------------------------------------------------------
testing for MPI_Comm_f2c and MPI_Comm_c2f 
	MPI_Comm_f2c and MPI_Comm_c2f are supported
testing for fseeko and fseeko64
fseeko64 is supported
-----------------------------------------------------------------------------------------------

(...)
```

Once your configuration is complete, you should have a `configure.wrf` file, and you are ready to compile. To compile WRFV3, you will need to decide which type of case you wish to compile. The options are listed below.
```console
em_real (3d real case)
em_quarter_ss (3d ideal case)
em_b_wave (3d ideal case)
em_les (3d ideal case)
em_heldsuarez (3d ideal case)
em_tropical_cyclone (3d ideal case)
em_hill2d_x (2d ideal case)
em_squall2d_x (2d ideal case)
em_squall2d_y (2d ideal case)
em_grav2d_x (2d ideal case)
em_seabreeze2d_x (2d ideal case)
em_scm_xy (1d ideal case)
```

For this purpose we are going to compile WRF for real cases. Compilation should take about 20-30 minutes. The ongoing compilation can be checked.
```console
$ ./compile em_real >& compile.log &
$ tail -f compile.log
```

Once the compilation completes, to check whether it was successful, you need to look for executables in the `WRFV3/main` directory.
```console
$ ls -las main/*.exe
ndown.exe (one-way nesting)
real.exe (real data initialization)
tc.exe (for tc bogusing--serial only)
wrf.exe (model executable)
```

These executables are linked to 2 different directories. You can choose to run WRF from either directory.
```console
WRFV3/run
WRFV3/test/em_real
```

## Building WPS

* _A video of this part is available [here](https://www.youtube.com/watch?v=uCImaGGCWDs)._

After the WRF model is built, the next step is building the WPS program (if you plan to run real cases, as opposed to idealized cases). The WRF model MUST be properly built prior to trying to build the WPS programs. If you do not already have the WPS source code, move to your `Build_WRF` directory, download that file and unpack it. Then go into the WPS directory and make sure the WPS directory is clean.

```console
$ cd {path_to_dir}/Build_WRF
$ wget http://www2.mmm.ucar.edu/wrf/src/WPSV3.8.TAR.gz
$ tar -zxvf WPSV3.8.TAR.gz
$ cd {path_to_dir}/Build_WRF/WPS
$ ./clean
```

The next step is to configure WPS, however, you first need to set some paths for the ungrib libraries and then you can configure.

```console
$ sudo nano ~/.bashrc

(...)
export JASPERLIB=$DIR/grib2/lib
export JASPERINC=$DIR/grib2/include

$ source ~/.bashrc
$ ./configure
```

You should be given a list of various options for compiler types, whether to compile in serial or parallel, and whether to compile ungrib with GRIB2 capability. Unless you plan to create extremely large domains, it is recommended to compile WPS in serial mode, regardless of whether you compiled WRFV3 in parallel. It is also recommended that you choose a GRIB2 option (make sure you do not choose one that states "NO_GRIB2"). You may choose a non-grib2 option, but most data is now in grib2 format, so it is best to choose this option. You can still run grib1 data when you have built with grib2.

Choose the option that lists a compiler to match what you used to compile WRFV3, serial, and grib2. **Note: The option number will likely be different than the number you chose to compile WRFV3.**

```console
Will use NETCDF in dir: /home/modelagem/Build_WRF/LIBRARIES/netcdf
Found Jasper environment variables for GRIB2 support...
  $JASPERLIB = /home/modelagem/Build_WRF/LIBRARIES/grib2/lib
  $JASPERINC = /home/modelagem/Build_WRF/LIBRARIES/grib2/include
-----------------------------------------------------------------------------------------------
Please select from among the following supported platforms:

   1. Linux x86_64, gfortran (serial)
   2. Linux x86_64, gfortran (serial_NO_GRIB2)
   3. Linux x86_64, gfortran (dmpar)
   4. Linux x86_64, gfortran (dmpar_NO_GRIB2)
   5. Linux x86_64, PGI compiler (serial)
   6. Linux x86_64, PGI compiler (serial_NO_GRIB2)
   7. Linux x86_64, PGI compiler (dmpar)
   8. Linux x86_64, PGI compiler (dmpar_NO_GRIB2)
   9. Linux x86_64, PGI compiler, SGI MPT (serial)
  10. Linux x86_64, PGI compiler, SGI MPT (serial_NO_GRIB2)
  11. Linux x86_64, PGI compiler, SGI MPT (dmpar)
  12. Linux x86_64, PGI compiler, SGI MPT (dmpar_NO_GRIB2)
  13. Linux x86_64, IA64 and Opteron (serial)
  14. Linux x86_64, IA64 and Opteron (serial_NO_GRIB2)
  15. Linux x86_64, IA64 and Opteron (dmpar)
  16. Linux x86_64, IA64 and Opteron (dmpar_NO_GRIB2)
  17. Linux x86_64, Intel compiler (serial)
  18. Linux x86_64, Intel compiler (serial_NO_GRIB2)
  19. Linux x86_64, Intel compiler (dmpar)
  20. Linux x86_64, Intel compiler (dmpar_NO_GRIB2)
  21. Linux x86_64, Intel compiler, SGI MPT (serial)
  22. Linux x86_64, Intel compiler, SGI MPT (serial_NO_GRIB2)
  23. Linux x86_64, Intel compiler, SGI MPT (dmpar)
  24. Linux x86_64, Intel compiler, SGI MPT (dmpar_NO_GRIB2)
  25. Linux x86_64, Intel compiler, IBM POE (serial)
  26. Linux x86_64, Intel compiler, IBM POE (serial_NO_GRIB2)
  27. Linux x86_64, Intel compiler, IBM POE (dmpar)
  28. Linux x86_64, Intel compiler, IBM POE (dmpar_NO_GRIB2)
  29. Linux x86_64 g95 compiler (serial)
  30. Linux x86_64 g95 compiler (serial_NO_GRIB2)
  31. Linux x86_64 g95 compiler (dmpar)
  32. Linux x86_64 g95 compiler (dmpar_NO_GRIB2)
  33. Cray XE/XC CLE/Linux x86_64, Cray compiler (serial)
  34. Cray XE/XC CLE/Linux x86_64, Cray compiler (serial_NO_GRIB2)
  35. Cray XE/XC CLE/Linux x86_64, Cray compiler (dmpar)
  36. Cray XE/XC CLE/Linux x86_64, Cray compiler (dmpar_NO_GRIB2)
  37. Cray XC CLE/Linux x86_64, Intel compiler (serial)
  38. Cray XC CLE/Linux x86_64, Intel compiler (serial_NO_GRIB2)
  39. Cray XC CLE/Linux x86_64, Intel compiler (dmpar)
  40. Cray XC CLE/Linux x86_64, Intel compiler (dmpar_NO_GRIB2)

Enter selection [1-40] : 3
-----------------------------------------------------------------------------------------------
Configuration successful. To build the WPS, type: compile
-----------------------------------------------------------------------------------------------
```

The `metgrid.exe` and `geogrid.exe` programs rely on the WRF model's I/O libraries. There is a line in the `configure.wps` file that directs the WPS build system to the location of the I/O libraries from the WRF model.

```console
(...)
WRF_DIR = ../WRFV3
(...)
```

Above is the default setting. As long as the name of the WRF model's top-level directory is "WRFV3" and the WPS and WRFV3 directories are at the same level (which they should be if you have followed exactly as instructed on this page so far), then the existing default setting is correct and there is no need to change it. If it is not correct, you must modify the configure file and then save the changes before compiling.

You can now compile WPS. Compilation should take a few minutes. The ongoing compilation can be checked.

```console
$ ./compile >& compile.log &
$ tail -f compile.log
```

Once the compilation completes, to check whether it was successful, you need to look for 3 main executables in the WPS top-level directory. Then verify that they are not zero-sized.

```console
$ ls -ls *.exe
geogrid.exe
metgrid.exe
ungrib.exe
```


## Static geography data

The WRF modeling system is able to create idealized simulations, though most users are interested in the real-data cases. To initiate a real-data case, the domain's physical location on the globe and the static information for that location must be created. This requires a data set that includes such fields as topography and land use categories. Move to your `Build_WRF` directory, download the file and unpack it. Once unpacked it will be called `geog`, rename to `WPS_GEOG`.

```console
$ cd {path_to_dir}/Build_WRF
$ wget http://www2.mmm.ucar.edu/wrf/src/wps_files/geog_complete.tar.bz2
$ tar -xvf geog_complete.tar.bz2
$ mv geog WPS_GEOG
```

The directory infomation is given to the geogrid program in the `namelist.wps` file in the `&geogrid` section. The data expands to approximately 10 GB. This data allows a user to run the geogrid.exe program.

```console
$ cd WPS
$ nano namelist.wps

(...)
geog_data_path = '{path_to_dir}/Build_WRF/WPS_GEOG'
(...)
```

## Post processing

ARWpost is a Fortran program that reads WRF-ARW input and output files, then generates GrADS output files.

Once the output files have been generated, GrADS can be used to produce horizontal or vertical cross-section plots of scalar fields (contours) or vector fields (barbs or arrows), vertical profiles and soundings.

Is recommend the use of ARWpost Version 3 or higher. This code is not dependent on the successful compilation of the WRFV3 code, and can therefore be installed anywhere, even if WRFV3 is not installed on this computer.

Move to your `Build_WRF` directory, download the file and unpack it.

```console
$ cd {path_to_dir}/Build_WRF
$ wget http://www2.mmm.ucar.edu/wrf/src/ARWpost_V3.tar.gz
$ tar -zxvf ARWpost_V3.tar.gz
```

Once unpacked, move to `ARWpost` directory and look for the following files.

```console
$ cd {path_to_dir}/Build_WRF/ARWpost
$ ls -las
arch			# A directory containing configure and
				#    compilation control
clean			# Script to clean compiled code
compile			# Script to compile the code
configure		# Script to configure the compilation for
				# your system
namelist.ARWpost	# Namelist to control the running of the code
README			# A text file containing basic information
				# on running ARWpost
src				# Directory containing all source code
script			# Directory containing some grads sample
				# scripts
util			# Directory containing some utilities
```

Assuming that the NETCDF variable is set, it is possible to configure the ARWpost.

```console
$ ./configure
```

You will see a list of options for your computer. Make sure the netCDF path is correct and pick the compile option for your machine.

```console
Will use NETCDF in dir: /home/drasousa/Build_WRF/LIBRARIES/netcdf
-----------------------------------------------------------------------------------------------
Please select from among the following supported platforms.

1.  PC Linux i486 i586 i686 x86_64, PGI compiler	
2.  PC Linux i486 i586 i686 x86_64, Intel compiler	
3.  PC Linux i486 i586 i686 x86_64, gfortran compiler 

Enter selection [1-3] : 3
-----------------------------------------------------------------------------------------------
Configuration successful. To build the ARWpost, type: compile
-----------------------------------------------------------------------------------------------
```

Edit the `Makefile` file into the `src` folder and modify the `-L\$(NETCDF)` line into the `ARWpost.exe` environment to look like:

```console
$ cd {path_to_dir}/ARWpost/src
$ nano Makefile

(...)
-L$(NETCDF)/lib -lnetcdf -lnetcdff -I$(NETCDF)/include -lnetcdf
(...)
```

Move to the `ARWpost` folder and modify the `CFLAGS` and `CPP` lines into `configure.arwp` file.

```console
$ cd {path_to_dir}/ARWpost
$ nano configure.arwp

(...)
CFLAGS = -fPIC -m64
CPP = /lib/cpp -P -traditional
(...)
```

Then compile the ARWpost. If successful, the executable ARWpost.exe will be created.

```console
$ ./compile
$ ls -ls *.exe
ARWpost.exe
```
