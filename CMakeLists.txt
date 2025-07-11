# LICENSE: Apache License 2.0
# Copyright (c) Hardy Simpson
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#/

cmake_minimum_required(VERSION 3.12)

project(zlog)

include(GNUInstallDirs)

set(CMAKE_MODULE_PATH ${zlog_SOURCE_DIR}/cmake)

SET(CPACK_PACKAGE_VERSION_MAJOR "1")
SET(CPACK_PACKAGE_VERSION_MINOR "2")
SET(CPACK_PACKAGE_VERSION_PATCH "18")
SET(CPACK_RPM_PACKAGE_RELEASE 1) # release version.
SET(CPACK_PACKAGE_VERSION "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")
SET(ZLOG_VERSION ${CPACK_PACKAGE_VERSION})
SET(ZLOG_SO_VERSION ${CPACK_PACKAGE_VERSION_MAJOR})

message(STATUS "platform : ${CMAKE_SYSTEM}")

add_definitions("-g -Wall -Wstrict-prototypes")
set(CMAKE_C_FLAGS "-std=c99 -pedantic -D_DEFAULT_SOURCE")
set(CMAKE_C_FLAGS_DEBUG "-ggdb3 -DDEBUG")
set(CMAKE_C_FLAGS_RELEASE "-O2")

if (WIN32)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DWINVER=0x0500 -D_WIN32_WINNT=0x0500 ")
endif ()

cmake_policy(SET CMP0015 NEW)
# set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${zlog_BINARY_DIR}/lib")
# set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${zlog_BINARY_DIR}/lib")

# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${zlog_BINARY_DIR}/bin")

link_directories(${CMAKE_LIBRARY_OUTPUT_DIRECTORY})

set(Need_THREAD 1)

if (WIN32)
    set(Need_UNIXEM 1)
endif ()

include(cmake/LoadLibraries.cmake)

add_subdirectory(src)
add_subdirectory(cpack)

if (UNIT_TEST)
    enable_testing()
    add_subdirectory(test)
endif ()

# ====== add for vcpkg install ======

# 假设 src/CMakeLists.txt 定义了 add_library(zlog ...) 并设置了 install
# 安装头文件
install(FILES src/zlog.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# 根据 vcpkg 的安装路径动态设置配置文件安装目录
if(DEFINED VCPKG_INSTALLATION_ROOT)
    set(INSTALL_CONF_DIR "${VCPKG_INSTALLATION_ROOT}/share/${PROJECT_NAME}/config" CACHE PATH "Installation directory for configuration files")
else()
    option(INSTALL_CONF_DIR "Installation directory for configuration files" "/etc/zlog")
endif()

install(FILES doc/zlog.conf DESTINATION ${INSTALL_CONF_DIR})

# 生成并安装 CMake Config 文件
include(CMakePackageConfigHelpers)
configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/zlogConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/zlogConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/zlog
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/zlogConfig.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/zlog
)

install(EXPORT zlogTargets
    FILE zlogTargets.cmake
    NAMESPACE zlog::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/zlog
)


# 安装 target 和导出文件
install(TARGETS zlog
    EXPORT zlogTargets
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
