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


Resources
[CMake Tutorial](https://medium.com/@onur.dundar1/cmake-tutorial-585dd180109b)
