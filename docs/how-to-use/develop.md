# Develop with CMake

If you are not making any changes to the files and libraries, you can just
run `cmake --build` and profit. But sometimes you need to make some additional
changes that change the configuration.

## Adding source files

In most directories you will find a file `CMakeLists.txt` with lines like:
```cmake
target_sources(Octopus_lib PRIVATE
		absorbing_boundaries.F90
		)
```

Simply add your files there and run `cmake --build`. You do not need to run
the configure stage, since this is automatically detected, and it will use
the options of the previous configuration.

:::{admonition} But what about other files?

Any other files such as those in `/share`, `testsuite` are automatically
handled, e.g. `/share` is copied to both the build-directory and the install
path.

:::

## Adding libraries

Depending on the build system that the third-party library uses, this process
can be as easy as:

:::::{tab-set}
::::{tab-item} CMake
If it's a modern cmake project, you can simply add the following:
```cmake
set(some_option_of_third_party_library "ON")
FetchContent_Declare(Library_I_want
		GIT_REPOSITORY https://github.com/owner/library
		GIT_TAG main
		)
FetchContent_MakeAvailable(Library_I_want)

...


target_link_libraries(Octopus_lib PRIVATE target_of_third_party_library)
```

To find the name of `target_of_third_party_library`, you would need to navigate
the library's documentation or top-most `CMakeLists.txt` file. Also look at the
options exposed by that library.

:::{tip}
You can set `-DFETCHCONTENT_SOURCE_DIR_LIBRARY_I_WANT=/path/to/local/git/repo`
to prototype live instances of the third-party library that you want to work
with.
:::

::::
::::{tab-item} pkg-config
In most cases the project will provide configurations using `pkg-config`. You
can then add the library for prototyping with:
```cmake
find_package(PkgConfig REQUIRED)
pkg_check_modules(local_name
		library_name IMPORTED_TARGET
		)

...


target_link_libraries(Octopus_lib PRIVATE PkgConfig::local_name)
```
where `library_name` is the name you use in CLI to find the appropriate flags:
```console
$ pkg-config --cflags --libs library_name
```

For long-term support, you should add a `FindThirdPartyPackage.cmake` file in
the `/cmake` folder as:
```cmake
#[==============================================================================================[
#                            ThirdPartyPackage compatibility wrapper                            #
]==============================================================================================]

#[===[.md
# FindThirdPartyPackage

ThirdPartyPackage compatibility module for Octopus

This file is specifically tuned for Octopus usage. See `Octopus_FindPackage` for a more general
interface.

Feel free to add any interesting documentation here. In the future this could be rendered out for
the end-user.

]===]

include(Octopus)
Octopus_FindPackage(${CMAKE_FIND_PACKAGE_NAME}
		# A dummy name that could be changed for native cmake compatibility
		NAMES ThirdPartyPackage
		PKG_MODULE_NAMES library_name)

# Create appropriate aliases
if (${CMAKE_FIND_PACKAGE_NAME}_PKGCONFIG)
	add_library(ThirdPartyPackage::SomeAlias ALIAS PkgConfig::${CMAKE_FIND_PACKAGE_NAME})
elseif (NOT TARGET ThirdPartyPackage::SomeAlias)
	# Maybe in the future there will be native CMake support
	add_library(ThirdPartyPackage::SomeAlias ALIAS FutureTargetName)
endif ()
```

::::
::::{tab-item} Makefile
This one would require the most work, but it is possible to add such libraries.

1. Create a submodule in `/third_party` with a link to the git repository of
   that project
2. Add a `ThirdPartyPackage.cmake` file and create a CMake wrapper for this
   package. In some cases it can be as simple as:
   ```cmake
   # Abstrating the path of the source (see next step for more details)
   set(lib_root ${thirdpartylibrary_SOURCE_DIR})
   
   # See the Makefile for what source files you need 
   add_library(ThirdPartyLibrary)
   target_sources(ThirdPartyLibrary PRIVATE
   		${lib_root}/path/to/src/file.F90
   		...
   		)
   target_include_directories(ThirdPartyLibrary PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
   ```
3. Add the third party library to the octopus project:
   ```cmake
   # Use FetchContent so that the user can change the version in the future
   FetchContent_Declare(ThirdPartyLibrary
   		GIT_REPOSITORY https://github.com/owner/library
   		GIT_TAG main
   		)
   FetchContent_MakeAvailable(ThirdPartyLibrary)
   
   # The source directory is set to `thirdpartylibrary_SOURCE_DIR` and is used by `ThirdPartyPackage.cmake`
   add_subdirectory(${PROJECT_SOURCE_DIR}/third_party/ThirdPartyPackage.cmake)
   ```
   :::{tip}
   You can set `-DFETCHCONTENT_SOURCE_DIR_THIRDPARTYLIBRARY=/path/to/submodule`
   to use the submodule instead of downloading the source.
   :::
4. Contact the upstream and tell them we are in 21st century now.

::::
::::{tab-item} Manual `-I`, `-L` flags
**NO**
::::

:::::

When you are done and want to make it available for the general use, make sure
you create a relevant `OCTOPUS_ThirdPartyLibrary` option to enable the support
for this.
