# External libraries

Linking to external libraries is a nightmare using autotools, and this has been
a major deterrent for Octopus to be able to make use of external packages. This
is greatly simplified in CMake, and we can more easily develop integrations
with any libraries


Let's take netCDF as an example. Things get a lot more complicated for other
packages on the Autotools side, while on the CMake side, things remain the
same.

::::{tab-set}
:::{tab-item} Autotools
`/m4/netcdf.m4`
```m4
AC_DEFUN([ACX_NETCDF], [
acx_netcdf_ok=no

dnl Check if the library was given in the command line
AC_ARG_WITH(netcdf-prefix, [AS_HELP_STRING([--with-netcdf-prefix=DIR], [Directory where netcdf was installed.])])
case $with_netcdf_prefix in
  no ) acx_netcdf_ok=disabled ;;
  "") if test "x$FCFLAGS_NETCDF" == x; then
    FCFLAGS_NETCDF="$ax_cv_f90_modflag/usr/include"
  fi;;
  *) LIBS_NETCDF="-L$with_netcdf_prefix/lib"; FCFLAGS_NETCDF="$ax_cv_f90_modflag$with_netcdf_prefix/include" ;;
esac

AC_ARG_WITH(netcdf-include, [AS_HELP_STRING([--with-netcdf-include=DIR], [Directory where netcdf Fortran headers were installed.])])
case $with_netcdf_include in
  "") ;;
  *)  FCFLAGS_NETCDF="$ax_cv_f90_modflag$with_netcdf_include" ;;
esac

dnl Backup LIBS and FCFLAGS
acx_netcdf_save_LIBS="$LIBS"
acx_netcdf_save_FCFLAGS="$FCFLAGS"

dnl The tests
AC_MSG_CHECKING([for netcdf])
if test "$acx_netcdf_ok" != disabled; then
  netcdf_fcflags="$FCFLAGS_NETCDF"
  FCFLAGS="$netcdf_fcflags $acx_netcdf_save_FCFLAGS"
  for netcdf_libsl in "" -lnetcdf "-lnetcdff -lnetcdf"; do
    netcdf_libs="$LIBS_NETCDF $netcdf_libsl"
    LIBS="$netcdf_libs $acx_netcdf_save_LIBS"
    AC_LINK_IFELSE(AC_LANG_PROGRAM([],[
      use netcdf
      integer :: ncid
      integer :: status
      status = nf90_close(ncid)
    ]), [acx_netcdf_ok=yes; FCFLAGS_NETCDF="$netcdf_fcflags"; LIBS_NETCDF="$netcdf_libs"], [])
    if test "$acx_netcdf_ok" == yes; then
      LIBS_NETCDF=$netcdf_libs
      break
    fi
  done
fi
AC_MSG_RESULT([$acx_netcdf_ok ($FCFLAGS_NETCDF $LIBS_NETCDF)])

dnl Finally, execute ACTION-IF-FOUND/ACTION-IF-NOT-FOUND:
if test x"$acx_netcdf_ok" = xyes; then
  AC_DEFINE(HAVE_NETCDF,1,[Defined if you have NETCDF library.])
else
  AC_MSG_WARN([Could not find NetCDF library.
              *** Will compile without NetCDF and ETSF I/O support])
  FCFLAGS_NETCDF=""
  LIBS_NETCDF=""
fi

AC_SUBST(FCFLAGS_NETCDF)
AC_SUBST(LIBS_NETCDF)
FCFLAGS="$acx_netcdf_save_FCFLAGS"
LIBS="$acx_netcdf_save_LIBS"
])dnl ACX_NETCDF
```
:::

:::{tab-item} CMake

In the `CMakeLists.txt` files you will find:

```cmake
option(OCTOPUS_netCDF "Octopus: Build with netCDF support" OFF)

...

if (OCTOPUS_netCDF)
	find_package(netCDF-Fortran REQUIRED MODULE)
	set(HAVE_NETCDF 1)
endif ()

...

if (TARGET netCDF::Fortran)
	target_link_libraries(Octopus_lib PRIVATE netCDF::Fortran)
endif ()
```

This takes advantage of the built-in support for CMake packages. But sometimes
the package might not be correctly installed and this support might be missing.

Not to worry, this was already considered, see `/cmake/FindnetCDF-Fortran.cmake`

```cmake
include(Octopus)
Octopus_FindPackage(${CMAKE_FIND_PACKAGE_NAME}
		# Try to import via native CMake support
		NAMES netCDF-Fortran
		# Otherwise use `pkg-config` to find the necessary files
		PKG_MODULE_NAMES netcdf-fortran)

# Create appropriate aliases
if (${CMAKE_FIND_PACKAGE_NAME}_PKGCONFIG)
	add_library(netCDF::Fortran ALIAS PkgConfig::${CMAKE_FIND_PACKAGE_NAME})
elseif (NOT TARGET netCDF::Fortran)
	if (TARGET netCDF::netcdff)
		add_library(netCDF::Fortran ALIAS netCDF::netcdff)
	else ()
		add_library(netCDF::Fortran ALIAS netcdff)
	endif ()
endif ()
```

:::

::::

One aspect that is only present in CMake is the ability to link to git
repositories directly so that you can more easily prototype. For example,
see the `Spglib` integration.

```cmake
FetchContent_Declare(Spglib
		GIT_REPOSITORY https://github.com/spglib/spglib
		GIT_TAG v2.1.0-rc2
		FIND_PACKAGE_ARGS MODULE REQUIRED COMPONENTS Fortran
		)
FetchContent_MakeAvailable(Spglib)

...


target_link_libraries(Octopus_lib PRIVATE Spglib::Fortran)
```

