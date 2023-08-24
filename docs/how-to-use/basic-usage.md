# Basic usage

## Configure, Build, Test, Install

For the user, the basics of how to configure, build, test and install in
cmake is as follows:

::::{tab-set}
:::{tab-item} CMake

For any `CMake >= 3.13` you can do the following to build octopus in
`./build` directory and install in `/path/to/octopus/install`

```console
$ cmake -B ./build -DCMAKE_INSTALL_PREFIX=/path/to/octopus/install
$ cmake --build ./build -- -j $(nproc)
$ ctest --test-dir ./build
$ cmake --install ./build
```
:::
:::{tab-item} CMake w/ presets

This requires `CMake >= 3.23` and it is the preferred approach moving forward.
The build directory is defined in the preset, e.g. `cmake-build-release`.
You can change the configuration to your liking directly in the CLI.

```console
$ cmake --preset default -DCMAKE_INSTALL_PREFIX=/path/to/octopus/install
$ cmake --build --preset default -- -j $(nproc)
$ ctest --preset default
$ cmake --install ./cmake-build
```

If you want some of these options to be more permanent you can define a file
`/CMakeUserPresets.json` with the configurations you want to override, e.g.
the settings above correspond to:
```json
{
  "version": 6,
  "configurePresets": [
    {
      "name": "with-install",
      "displayName": "With custom install path",
      "inherits": [
        "default"
      ],
      "binaryDir": "build",
      "cacheVariables": {
        "CMAKE_INSTALL_PREFIX": {
          "type": "FILEPATH",
          "value": "/path/to/octopus/install"
        }
      }
    }
  ]
}
```

:::
:::{tab-item} CMake all-in-one
This requires CMake >= 3.25
```console
$ cmake --workflow --preset default
```

Note that you cannot edit the configure, build, test stages manually. But
for almost all cases, you will find an equivalent option in the
`CMakePresets.json` files, and you can create your own development environment.

:::
::::

## How do I find the configuration options?

Some cmake options are defined dynamically (they might be in an if statement),
so these cannot be queried just by running `cmake --help`. But there are of
course built-in methods of navigating the options using `ccmake` and other
GUI interfaces
```console
$ ccmake -B ./build
OCTOPUS_FFTW                    *OFF
OCTOPUS_INSTALL                 *ON
OCTOPUS_MAX_DIM                 *3
```

But, you shouldn't be afraid of just opening the `CMakeLists.txt` files and
see directly what the options are and what do they do:
```cmake
#[==============================================================================================[
#                                            Options                                            #
]==============================================================================================]

set(OCTOPUS_MAX_DIM "3" CACHE STRING
		"Octopus: Maximum number of dimensions Octopus can use; [default=3;must be>=3]")
option(OCTOPUS_MPI "Octopus: Build with MPI support" OFF)
option(OCTOPUS_OpenMP "Octopus: Build with OpenMP support" OFF)
option(OCTOPUS_ELPA "Octopus: Build with ELPA support" OFF)
option(OCTOPUS_netCDF "Octopus: Build with netCDF support" OFF)
option(OCTOPUS_INSTALL "Octopus: Install project" ${PROJECT_IS_TOP_LEVEL})
option(OCTOPUS_FFTW "Octopus: Build with FFTW support" OFF)
option(OCTOPUS_MKL "Octopus: Build with MKL support" OFF)
option(OCTOPUS_ScaLAPACK "Octopus: Build with ScaLAPACK support" OFF)

#[==============================================================================================[
#                                     Project configuration                                     #
]==============================================================================================]
```

To use these options simply prefix the option with a `-D` and specify the value
you want, e.g. `-DOCTOPUS_MPI=True`.

Some options are not included here as they are either CMake native or part
of third-party libraries, e.g. `CMAKE_INSTALL_PREFIX` is the built-in option
to specify the install path root. Modern CMake design encourages the options
to be prefixed by the project name, so you should be able to guess where to
find further documentation of those options.
