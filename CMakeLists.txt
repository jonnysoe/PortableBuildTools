cmake_minimum_required(VERSION 3.22)

if(NOT DEFINED CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

# Version is hardcoded in pbt.rc
project(PortableBuildTools VERSION 2.10.2 LANGUAGES C)
set(CMAKE_C_STANDARD 11)

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/source
)

set(SOURCE_FILES
    source/miniz/miniz.c
    source/pbt.c
    source/pbt.rc
)

add_compile_definitions(
    _CRT_DECLARE_NONSTDC_NAMES=0
    NOMINMAX
    UNICODE
    MINIZ_NO_STDIO
    MINIZ_NO_TIME
    MINIZ_NO_DEFLATE_APIS
    MINIZ_NO_ZLIB_APIS
    MINIZ_NO_ZLIB_COMPATIBLE_NAME
    MINIZ_NO_MALLOC
)

if(MSVC)
    add_compile_options(
        /utf-8
        /INCREMENTAL:NO
        /wd4244
        /wd4267
    )

    add_link_options(
        /entry:_start
    )
else()
    add_compile_options(
        -finput-charset=UTF-8
        -Wno-inconsistent-dllimport
        -Wno-incompatible-pointer-types-discards-qualifiers
        -Wno-incompatible-function-pointer-types
    )
    add_link_options(
        --entry=_start
    )
endif()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)
set(CMAKE_PDB_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})

# Can't turn on IPO because this codebase has tons of standard function redefinition
set(CMAKE_INTERPROCEDURAL_OPTIMIZATION OFF)
# set(CMAKE_INTERPROCEDURAL_OPTIMIZATION_RELEASE ON)

add_executable(${PROJECT_NAME} ${SOURCE_FILES})
target_link_libraries(${PROJECT_NAME}
    ComCtl32
    ShLwApi
    wininet
    # Doesn't sound right as CMake should be handling the /MD or /MDd flags properly,
    # so shouldn't require explicit libraries for debugging
    # https://learn.microsoft.com/en-us/cpp/c-runtime-library/crt-library-features?view=msvc-170
    $<$<CONFIG:Debug>:vcruntimed>
    $<$<CONFIG:Debug>:ucrtd>
)