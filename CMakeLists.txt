# Project definition
cmake_minimum_required(VERSION 3.1)
project(Boxer VERSION 1.0.0 LANGUAGES CXX)

include(GNUInstallDirs)

set(LINUX FALSE)
if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
   set(LINUX TRUE)
endif()

# Source files
set(SRC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/src")
set(INC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/include")
set(SOURCES "${INC_DIR}/boxer/boxer.h")
if (APPLE)
   list(APPEND SOURCES
      "${SRC_DIR}/boxer_mac.mm"
   )
elseif (WIN32)
   list(APPEND SOURCES
      "${SRC_DIR}/boxer_win.cpp"
   )
elseif (LINUX)
   list(APPEND SOURCES
      "${SRC_DIR}/boxer_linux.cpp"
   )
endif ()

# Library definition and features
add_library(${PROJECT_NAME} ${SOURCES})
target_include_directories(${PROJECT_NAME} PUBLIC
   $<BUILD_INTERFACE:${INC_DIR}>
   $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)
target_compile_features(${PROJECT_NAME} PUBLIC cxx_strong_enums cxx_nullptr)

# Platform-specific dependencies
if (APPLE)
   find_library(COCOA_LIBRARY Cocoa)
   target_link_libraries(${PROJECT_NAME} PUBLIC ${COCOA_LIBRARY})
elseif (LINUX)
   find_package(PkgConfig REQUIRED)
   pkg_check_modules(GTK3 REQUIRED gtk+-3.0)
   target_link_libraries(${PROJECT_NAME} PUBLIC ${GTK3_LIBRARIES})
   target_include_directories(${PROJECT_NAME} PRIVATE ${GTK3_INCLUDE_DIRS})
endif ()

# Symbol exporting
if (BUILD_SHARED_LIBS)
   target_compile_definitions(${PROJECT_NAME} PRIVATE "BOXER_BUILD_DLL")
   target_compile_definitions(${PROJECT_NAME} INTERFACE "BOXER_DLL")
endif ()

# Build options
option(BOXER_BUILD_EXAMPLES "Build example programs" OFF)
if (BOXER_BUILD_EXAMPLES)
   add_subdirectory("examples")
endif()

# Install rules
option(BOXER_INSTALL "Generate installation target" ON)
if (BOXER_INSTALL)
   install(TARGETS Boxer
           EXPORT BoxerTargets)

   install(DIRECTORY include/boxer DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
           FILES_MATCHING PATTERN boxer.h)

   set(BOXER_CONFIG_PATH "${CMAKE_INSTALL_LIBDIR}/cmake/Boxer")
   install(EXPORT BoxerTargets
           FILE "BoxerTargets.cmake"
           NAMESPACE Boxer::
           EXPORT_LINK_INTERFACE_LIBRARIES
           DESTINATION "${BOXER_CONFIG_PATH}")

   include(CMakePackageConfigHelpers)
   write_basic_package_version_file(
      "${BOXER_CONFIG_PATH}/BoxerConfigVersion.cmake"
      VERSION ${PROJECT_VERSION}
      COMPATIBILITY AnyNewerVersion
   )
endif()
