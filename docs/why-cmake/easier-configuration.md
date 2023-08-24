# Easier configuration

On the user side, you might not see much difference when configuring for cmake or autotools.

::::{tab-set}
:::{tab-item} Autotools
```console
$ ./configure --enable-mpi
$ make
$ make check
```
:::
:::{tab-item} CMake
```console
$ cmake -B ./build -DOCTOPUS_MPI=ON
$ cmake --build ./build
$ ctest --test-dir ./build
```
:::
:::{tab-item} CMake w/ presets
```console
$ cmake --workflow --preset default
```
:::
::::

But things get complicated once you need to configure your Octopus program.

::::{tab-set}
:::{tab-item} Autotools

Autotools does not have built-in support for finding packages so a lot of
the configuration has to be done by the user.

```console
$ module load gcc netcdf-serial gsl
$ ./configure --with-netcdf-prefix=${NETCDF_HOME} --with-gsl-prefix=${GSL_HOME}
```
:::
:::{tab-item} CMake

CMake has many built-in functionality to search for the necessary packages
automatically. Most operating systems, package managers, and HPC systems
are designed with this built-in support in mind

```console
$ module load gcc netcdf-serial gsl
$ cmake -B ./build -DOCTOPUS_netCDF=ON
```
:::
::::

And under the hood things are a lot simpler.

::::{tab-set}
:::{tab-item} Autotools
See `/configure.ac`

```m4
dnl check whether mpi is enabled
AC_ARG_ENABLE(mpi, AS_HELP_STRING([--enable-mpi(=PATH)], [Parallel version. For ancient versions of MPI, you may need to specify PATH for MPI libs.]))
case $enable_mpi in
  yes) ;;
  no | "") enable_mpi=no ;;
  -* | */* | *.a | *.so | *.so.* | *.o)
    LIBS_MPI="$enable_mpi"
    enable_mpi=yes
    ;;
  *)
    LIBS_MPI="-l$enable_mpi"
    enable_mpi=yes
    ;;
esac
AM_CONDITIONAL(USE_MPI, test "$enable_mpi" = "yes")

if test x"$enable_mpi" == x"yes"; then
  octopus_default_cc=mpicc
else
  octopus_default_cc=gcc
fi

...

dnl check for Fortran MPI support
if test x"$enable_mpi" != x"no"; then
  ACX_MPI([], AC_MSG_ERROR([could not compile an MPI test program]))
  ACX_MPI_FC_MODULE
  ACX_MPI2
fi
```
:::
:::{tab-item} CMake
```cmake
option(OCTOPUS_MPI "Octopus: Build with MPI support" OFF)

if (OCTOPUS_MPI)
	find_package(MPI REQUIRED)
endif ()

if (TARGET MPI::MPI_Fortran)
	target_link_libraries(Octopus_lib PUBLIC MPI::MPI_Fortran)
endif ()
```
:::
::::

A lot of the complications of Autotools is abstracted away by CMake. The only
configurations that the developer needs to worry about is:
- What are the source files for the library/executable `target_sources()`
- What do I link to the library/executable `target_link_libraries()`
