cmake_minimum_required(VERSION 3.0)
project(test_subdirectory_target CXX)

add_subdirectory(${PYBIND11_PROJECT_DIR} pybind11)

add_library(test_cmake_build MODULE ../main.cpp)

target_link_libraries(test_cmake_build PRIVATE pybind11::module)

# make sure result is, for example, test_installed_target.so, not libtest_installed_target.dylib
set_target_properties(test_cmake_build PROPERTIES PREFIX "${PYTHON_MODULE_PREFIX}"
                                                  SUFFIX "${PYTHON_MODULE_EXTENSION}")

add_custom_target(check ${CMAKE_COMMAND} -E env PYTHONPATH=$<TARGET_FILE_DIR:test_cmake_build>
                  ${PYTHON_EXECUTABLE} ${PROJECT_SOURCE_DIR}/../test.py ${PROJECT_NAME})

if(CMAKE_CXX_STANDARD AND CMAKE_CXX_COMPILER_ID MATCHES "Clang" AND PYTHON_VERSION VERSION_LESS 3.0)
  if(CMAKE_CXX_STANDARD LESS 17)
    target_compile_options(test_cmake_build PUBLIC -Wno-deprecated-register)
  else()
    target_compile_options(test_cmake_build PUBLIC -Wno-register)
  endif()
endif()
