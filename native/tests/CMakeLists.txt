# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT license.

cmake_minimum_required(VERSION 3.12)

project(SEALTest VERSION 3.6.0 LANGUAGES CXX C)

# If not called from root CMakeLists.txt
if(NOT DEFINED SEAL_BUILD_TESTS)
    set(SEAL_BUILD_TESTS ON)

    # Import Microsoft SEAL
    find_package(SEAL 3.6.0 EXACT REQUIRED)

    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/${OUTLIB_PATH})
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/bin)
    set(THIRDPARTY_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/thirdparty)
else()
    set(THIRDPARTY_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/../../thirdparty)
endif()

if(NOT DEFINED SEAL_BUILD_DEPS)
    # [option] SEAL_BUILD_DEPS (default: ON)
    # Download and build missing dependencies, throw error if disabled.
    set(SEAL_BUILD_DEPS_OPTION_STR "Automatically download and build unmet dependencies")
    option(SEAL_BUILD_DEPS ${SEAL_BUILD_DEPS_OPTION_STR} ON)
endif()

# if SEAL_BUILD_TESTS is ON, use GoogleTest
if(SEAL_BUILD_TESTS)
    if(SEAL_BUILD_DEPS)
        list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/../../cmake)
        set(SEAL_THIRDPARTY_DIR ${CMAKE_CURRENT_LIST_DIR}/../../thirdparty)
        seal_fetch_thirdparty_content(ExternalGTest)
        add_library(GTest::gtest ALIAS gtest)
    else()
        find_package(GTest 1.10.0 CONFIG)
        if(NOT GTest_FOUND)
            message(FATAL_ERROR "GoogleTest: not found")
        else()
            message(STATUS "GoogleTest: found")
        endif()
    endif()

    add_executable(sealtest "")

    add_subdirectory(seal)

    if(TARGET SEAL::seal)
        target_link_libraries(sealtest PRIVATE SEAL::seal GTest::gtest)
    elseif(TARGET SEAL::seal_shared)
        target_link_libraries(sealtest PRIVATE SEAL::seal_shared GTest::gtest)
    else()
        message(FATAL_ERROR "Cannot find target SEAL::seal or SEAL::seal_shared")
    endif()
endif()
