cmake_minimum_required(VERSION 3.10)
include(CheckCXXCompilerFlag)

project(opencbdc-tx)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_EXTENSIONS False)

try_compile(USE_STDFS_LIB ${CMAKE_BINARY_DIR} "${CMAKE_SOURCE_DIR}/cmake-tests/test-libstdc++fs.cpp" LINK_LIBRARIES "stdc++fs")
check_cxx_compiler_flag(-Wshadow-all W_SHADOW_ALL)
check_cxx_compiler_flag(-Wnewline-eof W_NEWLINE_EOF)


set(SSE41_CXXFLAGS "-msse4.1")
set(AVX2_CXXFLAGS  "-mavx -mavx2")
set(SHANI_CXXFLAGS "-msse4 -msha")

try_compile(USE_SSE41 ${CMAKE_BINARY_DIR} "${CMAKE_SOURCE_DIR}/cmake-tests/test-sse41.cpp" COMPILE_DEFINITIONS ${SSE41_CXXFLAGS})
try_compile(USE_AVX2  ${CMAKE_BINARY_DIR} "${CMAKE_SOURCE_DIR}/cmake-tests/test-avx2.cpp"  COMPILE_DEFINITIONS ${AVX2_CXXFLAGS})
try_compile(USE_SHANI ${CMAKE_BINARY_DIR} "${CMAKE_SOURCE_DIR}/cmake-tests/test-shani.cpp" COMPILE_DEFINITIONS ${SHANI_CXXFLAGS})

set(CMAKE_CXX_OUTPUT_EXTENSION_REPLACE ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

find_package(Threads)

if(USE_STDFS_LIB)
    link_libraries(-lstdc++fs)
endif()

link_libraries(-lprotobuf)

find_library(LEVELDB_LIBRARY leveldb REQUIRED)
find_library(NURAFT_LIBRARY nuraft REQUIRED)

include_directories(3rdparty 3rdparty/secp256k1/include)

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    add_compile_options(-fprofile-arcs -ftest-coverage)
endif()


if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    add_link_options(--coverage)
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Profiling")
    add_link_options(-Wl,--build-id)
    add_compile_options(-O3
                        -fno-omit-frame-pointer
                        -g)
endif()

add_subdirectory(3rdparty)

add_compile_options(-Wall
                    -Wextra
                    -pedantic
                    -Werror
                    -fno-exceptions
                    -Wno-class-memaccess)

add_compile_definitions(_NO_EXCEPTION)

if(W_SHADOW_ALL)
    add_compile_options(-Wshadow-all)
else()
    add_compile_options(-Wshadow)
endif()

if(W_NEWLINE_EOF)
    add_compile_options(-Wnewline-eof)
endif()

add_subdirectory(src)
add_subdirectory(tests)
add_subdirectory(tools/bench)
add_subdirectory(tools/shard-seeder)
