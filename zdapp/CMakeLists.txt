cmake_minimum_required(VERSION 3.5)
project(zdapp)

# This only works if `zdlib` as previously installed using cget or cmake
find_package(zd)

add_executable(zdapp ${CMAKE_CURRENT_SOURCE_DIR}/src/zdapp.cpp)

# Hey! It's that library we installed from `zdlib`'s CMakeFile! Remember that
# thing?
target_link_libraries(zdapp zd::zd)

install(TARGETS zdapp
    RUNTIME DESTINATION bin
)
