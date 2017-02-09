# Copyright (c) 2014 Jarryd Beck
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.

IF(BIICODE)
    include(biicode/cmake/tools)
    ADD_BIICODE_TARGETS()
    ACTIVATE_CPP11(INTERFACE ${BII_BLOCK_TARGET})
RETURN()

ENDIF()

cmake_minimum_required(VERSION 3.1)
project(cxxopts VERSION 1.0.0)

enable_testing()

option(CXXOPTS_BUILD_EXAMPLES "Set to ON to build examples" ON)
option(CXXOPTS_BUILD_TESTS "Set to ON to build tests" OFF)

set(CXXOPTS_LINKER_LIBRARIES "")
option(CXXOPTS_USE_UNICODE_HELP "Use ICU Unicode library" OFF)
if(CXXOPTS_USE_UNICODE_HELP)

  find_package(PkgConfig)

  pkg_check_modules(ICU REQUIRED icu-uc)

  set(CXXOPTS_LINKER_LIBRARIES "${ICU_LDFLAGS}")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${ICU_CFLAGS} -DCXXOPTS_USE_UNICODE")
endif()

add_library(cxxopts INTERFACE)
target_include_directories(
  cxxopts INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
  )
target_link_libraries(
  cxxopts
  INTERFACE ${CXXOPTS_LINKER_LIBRARIES}
  )

add_subdirectory(src)
add_subdirectory(test)

# Install {

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Use variables:
#   * targets_export_name
#   * PROJECT_NAME
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    TARGETS cxxopts
    EXPORT "${targets_export_name}"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(FILES include/cxxopts.hpp DESTINATION ${include_install_dir})

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

# }
