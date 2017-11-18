# zd - This shows how to create a very simple C++ project that
# adheres to best practice. Unlike most projects this one has zero
# dependencies, making it a good starting case.

# Use Modern CMake, and force your will onto others!
cmake_minimum_required(VERSION 3.5)
# Name of the project
project(zd)

include(CMakePackageConfigHelpers)

add_library(zd
    ${CMAKE_CURRENT_SOURCE_DIR}/include/zd.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/zd.cpp
)

target_include_directories(zd PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
)

# Normally, target_link_libraries would be used here, but as this project
# has zero dependencies that's not the case.


# This actually installs the library.

install(DIRECTORY include/ DESTINATION include)

install(TARGETS zd EXPORT zd-targets
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    INCLUDES DESTINATION include
)

# This installs usage requirements. Again since there are no dependencies
# this is uninteresting.

# This creates a CMakeFile like this one which will create `zd::zd` and
# all the other goodies we want dependent projects to use. There's a lot
# that goes on in this file though so it's good this install command
# will just do it for us.

add_library(zd::zd ALIAS zd)
install(EXPORT zd-targets
    FILE zd-targets.cmake
    NAMESPACE zd::
    DESTINATION lib/cmake/zd
)

# Installs the config file. This is what CMake will try to find when we
# run `find_package` elsewhere.
# Normally this file would try to find dependencies, but since this lib is
# all about zero deps we skip that.
# Note it includes the zd-targets file we create above. If we don't do
# this, find_package won't do anything!
file(WRITE "${PROJECT_BINARY_DIR}/zd-config.cmake" "
include(CMakeFindDependencyMacro)
include(\"\${CMAKE_CURRENT_LIST_DIR}/zd-targets.cmake\")
")

write_basic_package_version_file("${PROJECT_BINARY_DIR}/zd-config-version.cmake"
    VERSION 1.0
    COMPATIBILITY AnyNewerVersion
)

install(FILES
    "${PROJECT_BINARY_DIR}/zd-config.cmake"
    "${PROJECT_BINARY_DIR}/zd-config-version.cmake"
    DESTINATION lib/cmake/zd
)
