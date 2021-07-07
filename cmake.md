# CMake


Build logic and definitions with CMake language is written either in `CMakeLists.txt` or a file ends with `<project_name>.cmake`. But as a best practice, main script is named as `CMakeLists.txt` instead of `.cmake`

Indentation is not mandatory but suggested while writing CMake scripts. CMake doesn’t use ‘;’ to understand end of statement.

CMake commands are case insensitive(CMake Variables are case-sensitive). Some commonly used commands,

- message: prints given message
- cmake_minimum_required: sets minimum version of cmake to be used
- add_executable: adds executable target with given name
- add_library: adds a library target to be build from listed source files
- add_subdirectory: adds a subdirectory to build


### CMake Environment Variables
Some of the environment variables are overriden by predefined CMake Variables. e.g. CXXFLAGS is overriden when CMAKE_CXX_FLAGS is defined.

CMake variables can be set in CMakeLists.txt using `set(<variable> <value>`, or set through command line `cmake -D<var>=<value>`

Resources
[CMake Tutorial](https://medium.com/@onur.dundar1/cmake-tutorial-585dd180109b)


### add source files
To add source files in directories, wildcard characters can be use.
```
file(GLOB_RECURSE sources CONFIGURE_DEPENDS src/*.cpp src/*.hpp include/*.hpp)
add_executable(${PROJECT_NAME} ${sources} )
```

### apply patches
```
# Look for and apply patches to the third-party (and sub) repo.
file(GLOB PATCHES "${CMAKE_SOURCE_DIR}/patches/*.patch")
if (PATCHES)
  message(STATUS "Patches: ${PATCHES}")
  foreach(PATCH ${PATCHES})
    message(STATUS "Applying ${PATCH}")
    execute_process(
      COMMAND patch -p1 --forward --ignore-whitespace
      WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}"
      INPUT_FILE "${PATCH}"
      OUTPUT_VARIABLE OUTPUT
      RESULT_VARIABLE RESULT)
    if (RESULT EQUAL 0)
      message(STATUS "Patch applied: ${PATCH}")
    else()
      # Unfortunately although patch will recognise that a patch is already
      # applied it will still return an error.
      execute_process(
        COMMAND patch -p1 -R --dry-run
        WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}"
        INPUT_FILE "${PATCH}"
        OUTPUT_VARIABLE OUTPUT
        RESULT_VARIABLE RESULT2)
      if (RESULT2 EQUAL 0)
        message(STATUS "Patch was already applied: ${PATCH}")
      else()
        message(FATAL_ERROR "Error applying patch ${PATCH}")
      endif()
    endif()
  endforeach(PATCH)
endif()
```
[source](https://github.com/facebook/hhvm/blob/master/CMakeLists.txt)
