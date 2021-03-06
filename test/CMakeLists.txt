set(BEEHIVE_TEST_BINARY_DIR ${CMAKE_BINARY_DIR}/test)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake/")

# Source: https://github.com/google/googletest/tree/master/googletest#incorporating-into-an-existing-cmake-project

# Download and unpack googletest at configure time
configure_file(CMakeLists.txt.in googletest-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${BEEHIVE_TEST_BINARY_DIR}/googletest-download )
if(result)
  message(FATAL_ERROR "CMake step for googletest failed: ${result}")
endif()
execute_process(COMMAND ${CMAKE_COMMAND} --build .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${BEEHIVE_TEST_BINARY_DIR}/googletest-download )
if(result)
  message(FATAL_ERROR "Build step for googletest failed: ${result}")
endif()

# Prevent overriding the parent project's compiler/linker
# settings on Windows
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

# Add googletest directly to our build. This defines
# the gtest and gtest_main targets.
add_subdirectory(${CMAKE_BINARY_DIR}/googletest-src
                 ${CMAKE_BINARY_DIR}/googletest-build
                 EXCLUDE_FROM_ALL)

# The gtest/gtest_main targets carry header search path
# dependencies automatically when using CMake 2.8.11 or
# later. Otherwise we have to add them here ourselves.
if (CMAKE_VERSION VERSION_LESS 2.8.11)
  include_directories("${gtest_SOURCE_DIR}/include")
endif()

file(GLOB_RECURSE BEEHIVE_TESTS "tests/*.cpp" "../include/*.hpp" "../include/*.h")

add_executable(beehive_test
  ${BEEHIVE_TESTS}
)

target_include_directories(beehive_test
  PUBLIC ../include
)

target_link_libraries(beehive_test
  beehive
  gmock
  gmock_main
  gtest
  gtest_main
)

enable_testing()
add_test(beehive_test beehive_test)

include(CodeCoverage)
append_coverage_compiler_flags(--coverage)
setup_target_for_coverage(
  NAME beehive_test_coverage
  EXECUTABLE beehive_test
  DEPENDENCIES beehive_test
)

