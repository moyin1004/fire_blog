cmake_minimum_required(VERSION 3.11)

option(BUILD_TEST "Build TEST." ON)

project(fire_blog VERSION 0.0.1)

if(BUILD_TEST)
    add_subdirectory(test)
endif()
