cmake_minimum_required(VERSION 3.11)

# set(ENV{http_proxy} "http://127.0.0.1:33211")
# set(ENV{https_proxy} "http://127.0.0.1:33211")
if(${CMAKE_VERSION} VERSION_LESS 3.14)
    macro(FetchContent_MakeAvailable NAME)
        FetchContent_GetProperties(${NAME})

        if(NOT ${NAME}_POPULATED)
            FetchContent_Populate(${NAME})

            # add_subdirectory(${${NAME}_SOURCE_DIR})
        endif()
    endmacro()
endif()

project(test_blog LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)

include(FetchContent)
FetchContent_Declare(brpc
    GIT_REPOSITORY https://github.com/apache/brpc.git
    GIT_TAG 1.8.0
    GIT_SHALLOW TRUE
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
)
FetchContent_MakeAvailable(brpc)

find_path(LEVELDB_INCLUDE_PATH NAMES leveldb/db.h)
find_library(LEVELDB_LIB NAMES leveldb)

if((NOT LEVELDB_INCLUDE_PATH) OR(NOT LEVELDB_LIB))
    message(FATAL_ERROR "Fail to find leveldb")
endif()

include_directories()

find_package(PkgConfig REQUIRED)
find_package(Protobuf REQUIRED)

include(../cmake/util.cmake)
compile_proto(PROTO_HDRS PROTO_SRCS ${CMAKE_SOURCE_DIR}/test ${CMAKE_SOURCE_DIR}/test ${CMAKE_SOURCE_DIR}/test http.proto)
add_library(TEST_PROTO_LIB OBJECT ${PROTO_SRCS} ${PROTO_HDRS})

if(PkgConfig_FOUND)
    message("found pkgconfig")
else()
    message(FATAL_ERROR "no pkgconfig")
endif()

pkg_check_modules(brpc REQUIRED IMPORTED_TARGET openssl gflags protobuf grpc)

add_executable(test_blog main.cc $<TARGET_OBJECTS:TEST_PROTO_LIB>)
add_dependencies(test_blog PROTO_LIB brpc-static)

target_link_libraries(test_blog
    PRIVATE brpc
    PRIVATE PkgConfig::brpc
    PRIVATE leveldb
    PRIVATE pthread
)
target_include_directories(test_blog
    PRIVATE ${brpc_BINARY_DIR}/output/include
    PRIVATE ${LEVELDB_INCLUDE_PATH}
)
target_link_directories(test_blog
    PRIVATE ${brpc_BINARY_DIR}/output/lib
)