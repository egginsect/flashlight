cmake_minimum_required(VERSION 3.16)

find_package(GLOG REQUIRED)
if (GLOG_FOUND)
  message(STATUS "GLOG found")
  if (FL_BUILD_STANDALONE)
    setup_install_find_module(${CMAKE_MODULE_PATH}/FindGLOG.cmake)
  endif()
else()
  message(FATAL_ERROR "GLOG not found")
endif()

# Find gflags
find_package(gflags REQUIRED)
if (gflags_FOUND)
  message(STATUS "gflags found")
  if (FL_BUILD_STANDALONE)
    setup_install_find_module(${CMAKE_MODULE_PATH}/Findgflags.cmake)
  endif()
else()
  message(FATAL_ERROR "gflags not found")
endif()

add_library(
  fl_pkg_runtime
  ""
)

target_sources(
  fl_pkg_runtime
  PRIVATE
  ${CMAKE_CURRENT_LIST_DIR}/Runtime.cpp
)

include(${CMAKE_CURRENT_LIST_DIR}/amp/CMakeLists.txt)
include(${CMAKE_CURRENT_LIST_DIR}/plugin/CMakeLists.txt)
include(${CMAKE_CURRENT_LIST_DIR}/common/CMakeLists.txt)

# flashlight-text is required
if (NOT TARGET flashlight::flashlight-text)
  find_package(flashlight-text CONFIG)
  if (NOT TARGET flashlight::flashlight-text AND FL_BUILD_STANDALONE)
    message(STATUS "Flashlight Text not found - downloading and building from source")
    include(${CMAKE_MODULE_PATH}/BuildFlashlightText.cmake)
  elseif(TARGET flashlight::flashlight-text)
    message(STATUS "Found flashlight-text")
  else()
    message(FATAL_ERROR "Could not find requirement flashlight-text.")
  endif()
endif()

target_link_libraries(
  fl_pkg_runtime
  PUBLIC
  ${GLOG_LIBRARIES}
  ${gflags_LIBRARIES}
  flashlight
  flashlight::flashlight-text
)

target_include_directories(
  fl_pkg_runtime
  PUBLIC
  ${GLOG_INCLUDE_DIRS}
  ${gflags_INCLUDE_DIRS}
)

if (FL_BUILD_TESTS)
  add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/test)
endif ()

if (FL_CODE_COVERAGE)
  add_coverage_to_target(TARGET fl_pkg_runtime)
endif()
