# DOS2UNIX build with CMake

The Makefiles for DOS2UNIX as typical often require tweaking for a system.
This is an example of how CMake streamlines platform-agnostic builds.
This CMake script auto-downloads the untouched source code and builds with CMake instead.

```sh
cmake -Bbuild
cmake --build build
```

this creates executables dos2unix and unix2dos under build/
