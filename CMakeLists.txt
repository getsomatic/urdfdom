cmake_minimum_required( VERSION 2.8 FATAL_ERROR )
project (urdfdom CXX C)

set (URDF_MAJOR_VERSION 1)
set (URDF_MINOR_VERSION 0)
set (URDF_PATCH_VERSION 3)

set (URDF_VERSION ${URDF_MAJOR_VERSION}.${URDF_MINOR_VERSION}.${URDF_PATCH_VERSION})
set (URDF_MAJOR_MINOR_VERSION ${URDF_MAJOR_VERSION}.${URDF_MINOR_VERSION})

message (STATUS "${PROJECT_NAME} version ${URDF_VERSION}")

include(GNUInstallDirs)

# set the default build type
if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

# If compiler support symbol visibility, enable it.
include(CheckCCompilerFlag)
check_c_compiler_flag(-fvisibility=hidden HAS_VISIBILITY)
if (HAS_VISIBILITY)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden")
endif()

# This shouldn't be necessary, but there has been trouble
# with MSVC being set off, but MSVCXX ON.
if(MSVC OR MSVC90 OR MSVC10)
  set(MSVC ON)
endif (MSVC OR MSVC90 OR MSVC10)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

find_package(TinyXML REQUIRED)
include_directories(SYSTEM ${TinyXML_INCLUDE_DIRS})

find_package(urdfdom_headers 1.0.3 REQUIRED PATHS /home/somatic/Development/projects/robot_cleaner_ws/build/urdfdom_headers/cmake)
include_directories(SYSTEM ${urdfdom_headers_INCLUDE_DIRS})
if (NOT MSVC)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
endif()

find_package(console_bridge 0.3 REQUIRED)
include_directories(SYSTEM ${console_bridge_INCLUDE_DIRS})
link_directories(${console_bridge_LIBRARY_DIRS})

#In Visual Studio a special postfix for 
#libraries compiled in debug is used
if(MSVC)
set(CMAKE_DEBUG_POSTFIX "d")
endif(MSVC)

# Control where libraries and executables are placed during the build
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")

enable_testing()

add_subdirectory(urdf_parser)

if(WIN32 AND NOT CYGWIN)
  set(CMAKE_CONFIG_INSTALL_DIR CMake)
else()
    set(CMAKE_CONFIG_INSTALL_DIR ${CMAKE_INSTALL_FULL_LIBDIR}/${PROJECT_NAME}/cmake/)
endif()

set(PKG_NAME ${PROJECT_NAME})
set(PKG_LIBRARIES urdfdom_sensor urdfdom_model_state urdfdom_model urdfdom_world)
set(PKG_DEPENDS urdfdom_headers console_bridge)
set(cmake_conf_file "cmake/urdfdom-config.cmake")
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/${cmake_conf_file}.in" "${CMAKE_BINARY_DIR}/${cmake_conf_file}" @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/${cmake_conf_file}
  DESTINATION ${CMAKE_CONFIG_INSTALL_DIR} COMPONENT cmake)

# Make the package config file
if (NOT MSVC)
  set(PKG_DESC "Unified Robot Description Format")
  set(PKG_DEPENDS "urdfdom_headers console_bridge") # make the list separated by spaces instead of ;
  set(PKG_URDF_LIBS "-lurdfdom_sensor -lurdfdom_model_state -lurdfdom_model -lurdfdom_world")
  set(pkg_conf_file "cmake/pkgconfig/urdfdom.pc")
  configure_file("${CMAKE_CURRENT_SOURCE_DIR}/${pkg_conf_file}.in" "${CMAKE_BINARY_DIR}/${pkg_conf_file}" @ONLY)
  install(FILES ${CMAKE_BINARY_DIR}/${pkg_conf_file}
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig/ COMPONENT pkgconfig)
endif()

# Add uninstall target
# Ref: http://www.cmake.org/Wiki/CMake_FAQ#Can_I_do_.22make_uninstall.22_with_CMake.3F
configure_file("${PROJECT_SOURCE_DIR}/cmake/uninstall.cmake.in" "${PROJECT_BINARY_DIR}/uninstall.cmake" IMMEDIATE @ONLY)
add_custom_target(uninstall "${CMAKE_COMMAND}" -P "${PROJECT_BINARY_DIR}/uninstall.cmake")

message(STATUS "Configuration successful. Type make to compile urdfdom")

SET_DIRECTORY_PROPERTIES(PROPERTIES
  ADDITIONAL_MAKE_CLEAN_FILES ${CMAKE_BINARY_DIR}/cmake/urdfdom-config.cmake
  ADDITIONAL_MAKE_CLEAN_FILES ${CMAKE_BINARY_DIR}/cmake/pkgconfig/urdfdom.pc)
