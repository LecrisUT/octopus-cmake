# Faster builds

This is a whole religious battle of which build system is the fastest. But
one aspect that all benchmarks agree is that Autotools is always the slowest.
Check [Meson](https://mesonbuild.com/Simple-comparison.html)'s benchmarks for
an example.

Most important factor that will affect your build speeds is the use of other
make systems, linking tools, etc. You can easily switch to using Ninja as the
build backend.
```console
$ cmake -G Ninja -B ./build
```

See for yourself the improved performance that these other modern tools have.

## Wait but what about Meson and other tools

Some benchmarks would indicate that CMake is still slower than Meson, Bazel,
etc. But one thing that they do not show is the interoperability, and the
new developments being made on CMake. CMake is a de-facto standard and as such
you have good IDE support, good support for almost all third-party libraries,
and a large community of developers.
