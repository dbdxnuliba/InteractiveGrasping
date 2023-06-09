cmake_minimum_required(VERSION 3.0)
project(test_installed_embed CXX)

set(CMAKE_MODULE_PATH "")
find_package(pybind11 CONFIG REQUIRED)
message(STATUS "Found pybind11 v${pybind11_VERSION}: ${pybind11_INCLUDE_DIRS}")

add_executable(test_cmake_build ../embed.cpp)
target_link_libraries(test_cmake_build PRIVATE pybind11::embed)


# Do not treat includes from IMPORTED target as SYSTEM (Python headers in pybind11::embed).
# This may be needed to resolve header conflicts, e.g. between Python release and debug headers.
set_target_properties(test_cmake_build PROPERTIES NO_SYSTEM_FROM_IMPORTED ON)

add_custom_target(check $<TARGET_FILE:test_cmake_build> ${PROJECT_SOURCE_DIR}/../test.py)

if(CMAKE_CXX_STANDARD AND CMAKE_CXX_COMPILER_ID MATCHES "Clang" AND PYTHON_VERSION VERSION_LESS 3.0)
  if(CMAKE_CXX_STANDARD LESS 17)
    target_compile_options(test_cmake_build PUBLIC -Wno-deprecated-register)
  else()
    target_compile_options(test_cmake_build PUBLIC -Wno-register)
  endif()
endif()
