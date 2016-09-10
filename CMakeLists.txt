cmake_minimum_required(VERSION 2.8.2)

project(rapidcheck_test1 CXX)

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
if(NOT COMPILER_SUPPORTS_CXX11)
  message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

if (CMAKE_VERSION VERSION_LESS 3.2)
    set(UPDATE_DISCONNECTED_IF_AVAILABLE "")
else()
    set(UPDATE_DISCONNECTED_IF_AVAILABLE "UPDATE_DISCONNECTED 1")
endif()

include(DownloadProject.cmake)
download_project(PROJ                googletest
                 GIT_REPOSITORY      https://github.com/google/googletest.git
                 GIT_TAG             master
                 ${UPDATE_DISCONNECTED_IF_AVAILABLE}
)
add_subdirectory(${googletest_SOURCE_DIR} ${googletest_BINARY_DIR})
download_project(PROJ                rapidcheck
                 GIT_REPOSITORY      https://github.com/emil-e/rapidcheck
                 GIT_TAG             master
                 ${UPDATE_DISCONNECTED_IF_AVAILABLE}
)
add_subdirectory(${rapidcheck_SOURCE_DIR} ${rapidcheck_BINARY_DIR})

# When using CMake 2.8.11 or later, header path dependencies
# are automatically added to the gtest and gmock targets.
# For earlier CMake versions, we have to explicitly add the
# required directories to the header search path ourselves.
if (CMAKE_VERSION VERSION_LESS 2.8.11)
    include_directories("${gtest_SOURCE_DIR}/include"
                        "${gmock_SOURCE_DIR}/include")
endif()

###########################################################

include(CTest)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(rapidcheck_test1 src/main.cpp)
target_include_directories(rapidcheck_test1 PRIVATE ${rapidcheck_SOURCE_DIR}/include)
target_include_directories(rapidcheck_test1 PRIVATE ${rapidcheck_SOURCE_DIR}/extras/gtest/include)
target_link_libraries(rapidcheck_test1 gtest gmock_main rapidcheck)

add_test(NAME rapidcheck_test1_always_the_same_values COMMAND rapidcheck_test1)
