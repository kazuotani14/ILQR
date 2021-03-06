cmake_minimum_required(VERSION 3.5.0)
project (iLQR)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_FLAGS "-O3") # -Wall", -Wextra")
# set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -Wall")

###
# My stuff
###

# find_package(Threads REQUIRED)

# set(CMAKE CXX_COMPILER /usr/local/bin/g++-7)
# cmake .. -DCMAKE_CXX_COMPILER=g++-7
# export CMAKE_CXX_COMPILER=g++-7
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fopenmp")

# find_package(OpenMP)
# if (OPENMP_FOUND)
#     message("success!")
# else()
#     message("fail!")
# endif()
#     # set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
#     set(CMAKE_C_FLAGS "${CMAKE_CXX_FLAGS} -fopenmp")
#     # set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
#     # set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
# endif()

# OPTION (USE_OpenMP "Use OpenMP" ON)
# IF(USE_OpenMP)
#   FIND_PACKAGE(OpenMP)
#   IF(OPENMP_FOUND)
#     SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
#     SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
#   ENDIF()
# ENDIF()

set (ILQR_SOURCES "src/ilqr_core.cpp"
                  "src/derivatives.cpp"
                  "src/boxqp.cpp")

# file(GLOB_RECURSE Foo_SOURCES "src/*.cpp")
file(GLOB_RECURSE ILQR_HEADERS "include/*.h")

set (ILQR_INCLUDE_DIRS "")
foreach (_headerFile ${ILQR_HEADERS})
    get_filename_component(_dir ${_headerFile} PATH)
    list (APPEND ILQR_INCLUDE_DIRS ${_dir})
endforeach()
list(REMOVE_DUPLICATES ILQR_INCLUDE_DIRS)

include_directories(${ILQR_INCLUDE_DIRS})
add_executable (run_iLQR "src/run_ilqr.cpp" ${ILQR_SOURCES})
# target_link_libraries(run_iLQR Threads::Threads)

###
# Use ExternalProject module to fetch GoogleTest and GoogleMock from internet and compile as third-party dependencies.
# From: http://kaizou.org/2014/11/gtest-cmake/
###

# We need thread support
find_package(Threads REQUIRED)

# Enable ExternalProject CMake module
include(ExternalProject)

# Download and install GoogleTest
ExternalProject_Add(
    gtest
    URL https://github.com/google/googletest/archive/master.zip
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/gtest
    # Disable install step
    INSTALL_COMMAND ""
)

# Get GTest source and binary directories from CMake project
ExternalProject_Get_Property(gtest source_dir binary_dir)

# Create a libgtest target to be used as a dependency by test programs
add_library(libgtest IMPORTED STATIC GLOBAL)
add_dependencies(libgtest gtest)

# Set libgtest properties
set_target_properties(libgtest PROPERTIES
    "IMPORTED_LOCATION" "${binary_dir}/googlemock/gtest/libgtest.a"
    "IMPORTED_LINK_INTERFACE_LIBRARIES" "${CMAKE_THREAD_LIBS_INIT}"
)

# Create a libgmock target to be used as a dependency by test programs
add_library(libgmock IMPORTED STATIC GLOBAL)
add_dependencies(libgmock gtest)

# Set libgmock properties
set_target_properties(libgmock PROPERTIES
    "IMPORTED_LOCATION" "${binary_dir}/googlemock/libgmock.a"
    "IMPORTED_LINK_INTERFACE_LIBRARIES" "${CMAKE_THREAD_LIBS_INIT}"
)

# I couldn't make it work with INTERFACE_INCLUDE_DIRECTORIES
include_directories("${source_dir}/googletest/include"
                    "${source_dir}/googlemock/include")

###
# My tests
###

macro(add_gtest exec_name filepath)
    set(OtherFiles ${ARGN})
    add_executable(${exec_name} ${filepath} ${OtherFiles})
    target_link_libraries(${exec_name} libgtest libgmock)
endmacro(add_gtest)

# add_gtest(gtest_test "test/test_gtest.cpp")

# add_gtest(deriv_test "test/test_finite_diff.cpp")
# add_gtest(boxqp_test "test/test_boxqp.cpp" "src/boxqp.cpp")
# add_gtest(dynamics_models_test "test/test_dynamicsmodels.cpp")
# add_gtest(ilqr_forward_test "test/test_ilqr_forward_pass.cpp" ${ILQR_SOURCES})
