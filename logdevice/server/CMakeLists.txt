# Copyright (c) 2017-present, Facebook, Inc. and its affiliates.
# All rights reserved.
#
# This source code is licensed under the BSD-style license found in the
# LICENSE file in the root directory of this source tree.

auto_sources(hfiles "*.h" RECURSE "${LOGDEVICE_SERVER_DIR}")
auto_sources(files "*.cpp" RECURSE "${LOGDEVICE_SERVER_DIR}")

REMOVE_MATCHES_FROM_LISTS(files hfiles
  MATCHES
    "/test/"
    "/locallogstore/test/"
    "main.cpp"
)

add_library(logdevice_server STATIC ${hfiles} ${files})

target_link_libraries(logdevice_server
  common
  ldclient_static
  ${LOGDEVICE_EXTERNAL_DEPS})

add_executable(logdeviced ${LOGDEVICE_SERVER_DIR}/main.cpp)
target_link_libraries(logdeviced logdevice_server)

install(TARGETS logdeviced
  COMPONENT runtime
  DESTINATION bin)

add_subdirectory(locallogstore)

# Tests
file(GLOB test_hfiles "${LOGDEVICE_SERVER_DIR}/test/*.h")
file(GLOB test_files  "${LOGDEVICE_SERVER_DIR}/test/*.cpp")

add_executable(server_test
  ${test_hfiles}
  ${test_files}
)

enable_testing()
if(HAVE_CMAKE_GTEST)
gtest_discover_tests(server_test
                     WORKING_DIRECTORY ${CMAKE_BINARY_DIR})
else()
add_test(
  NAME ServerTest
  COMMAND ${UNIT_TEST_OUTPUT_DIRECTORY}/server_test
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR})
endif()

target_link_libraries(server_test
  common
  common_test_util
  test_util
  logdevice_server
  locallogstore_test_util
  ${LOGDEVICE_EXTERNAL_DEPS}
  ${GMOCK_LIBRARY}
  ${GTEST_LIBRARY}
  ${LIBGFLAGS_LIBRARY})

set_target_properties(server_test
  PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY ${UNIT_TEST_OUTPUT_DIRECTORY})

target_compile_definitions(server_test
  PRIVATE
  GTEST_USE_OWN_TR1_TUPLE=0
)

file(COPY ${LOGDEVICE_SERVER_DIR}/test/configs DESTINATION
  ${CMAKE_BINARY_DIR}/logdevice/server/test/)
