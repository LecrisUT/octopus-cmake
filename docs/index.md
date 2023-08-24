# Octopus with CMake

Octopus now supports building with CMake. Until it is [merged](https://gitlab.com/octopus-code/octopus/-/merge_requests/1827),
you can find it at [gitlab.com/LecrisUT/octopus#cmake](https://gitlab.com/LecrisUT/octopus/-/tree/cmake?ref_type=heads).

```console
$ git clone https://gitlab.com/octopus-code/octopus
$ cd octopus
$ git remote add LecrisUT https://gitlab.com/LecrisUT/octopus
$ git fetch
$ git checkout LecrisUT/cmake
```

:::{toctree}
---
maxdepth: 2
titlesonly: true
glob: true
---
./why-cmake/index.md
./how-to-use/index.md
:::
