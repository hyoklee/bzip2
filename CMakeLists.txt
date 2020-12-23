cmake_minimum_required (VERSION 3.2.2)
PROJECT (BZ2 C)

#-----------------------------------------------------------------------------
# Basic BZ2 stuff here
#-----------------------------------------------------------------------------

#-----------------------------------------------------------------------------
# Define some CMake variables for use later in the project
#-----------------------------------------------------------------------------
set (BZ2_RESOURCES_DIR          ${BZ2_SOURCE_DIR}/config/cmake)
set (BZ2_SRC_DIR                ${BZ2_SOURCE_DIR}/src)
set (BZ2_TESTS_SRC_DIR          ${BZ2_SOURCE_DIR}/tests)
set (BZ2_UTILS_SRC_DIR          ${BZ2_SOURCE_DIR}/utils)

#-----------------------------------------------------------------------------
# Set the core names of all the libraries
#-----------------------------------------------------------------------------
set (BZ2_LIB_CORENAME              "bz2")

#-----------------------------------------------------------------------------
# Set the true names of all the libraries if customized by external project
#-----------------------------------------------------------------------------
set (BZ2_LIB_NAME              "${BZ2_EXTERNAL_LIB_PREFIX}${BZ2_LIB_CORENAME}")

#-----------------------------------------------------------------------------
# Set the target names of all the libraries
#-----------------------------------------------------------------------------
set (BZ2_LIB_TARGET              "${BZ2_LIB_CORENAME}-static")
set (BZ2_LIBSH_TARGET            "${BZ2_LIB_CORENAME}-shared")

if (NOT BZ2_INSTALL_BIN_DIR)
  set (BZ2_INSTALL_BIN_DIR bin)
endif ()
if (NOT BZ2_INSTALL_LIB_DIR)
  set (BZ2_INSTALL_LIB_DIR lib)
endif ()
if (NOT BZ2_INSTALL_INCLUDE_DIR)
  set (BZ2_INSTALL_INCLUDE_DIR include)
endif ()
if (NOT BZ2_INSTALL_DATA_DIR)
  if (NOT WIN32)
    if (APPLE)
      if (BZ2_BUILD_FRAMEWORKS)
        set (BZ2_INSTALL_EXTRA_DIR ../SharedSupport)
      else ()
        set (BZ2_INSTALL_EXTRA_DIR share)
      endif ()
      set (BZ2_INSTALL_FWRK_DIR ${CMAKE_INSTALL_FRAMEWORK_PREFIX})
    endif ()
    set (BZ2_INSTALL_DATA_DIR share)
    set (BZ2_INSTALL_CMAKE_DIR share/cmake)
  else ()
    set (BZ2_INSTALL_DATA_DIR ".")
    set (BZ2_INSTALL_CMAKE_DIR cmake)
  endif ()
endif ()

#-----------------------------------------------------------------------------
# parse the full version number from bzlib.h and include in BZ2LIB_VERSION
#-----------------------------------------------------------------------------
file (READ ${BZ2_SRC_DIR}/bzlib.h _bzlib_h_contents)
string (REGEX REPLACE ".*#define[ \t]+BZ2LIB_VERSION[ \t]+\"([0-9]*.[0-9]*.[0-9]*)\".*"
    "\\1" BZ2_FULL_VERSION ${_bzlib_h_contents})
string (REGEX REPLACE ".*#define[ \t]+BZ2LIB_VERSION[ \t]+\"([0-9]*).*$"
    "\\1" BZ2_VERS_MAJOR ${_bzlib_h_contents})
string (REGEX REPLACE ".*#define[ \t]+BZ2LIB_VERSION[ \t]+\"[0-9]*.([0-9]*).*$"
    "\\1" BZ2_VERS_MINOR ${_bzlib_h_contents})
string (REGEX REPLACE ".*#define[ \t]+BZ2LIB_VERSION[ \t]+\"[0-9]*.[0-9]*.([0-9]*)\".*$"
    "\\1" BZ2_VERS_RELEASE ${_bzlib_h_contents})
message (STATUS "VERSION: ${BZ2_FULL_VERSION}")
set (BZ2_SOVERS_MAJOR 1)
set (BZ2_SOVERS_MINOR 0)
set (BZ2_SOVERS_RELEASE 6)

#-----------------------------------------------------------------------------
set (BZ2_PACKAGE "bz2")
set (BZ2_PACKAGE_NAME "bz2")
set (BZ2_PACKAGE_VERSION "${BZ2_VERS_MAJOR}.${BZ2_VERS_MINOR}.${BZ2_VERS_RELEASE}")
set (BZ2_PACKAGE_VERSION_STRING "${BZ2_PACKAGE_VERSION}")
set (BZ2_PACKAGE_VERSION_MAJOR "${BZ2_VERS_MAJOR}")
set (BZ2_PACKAGE_VERSION_MINOR "${BZ2_VERS_MINOR}")
set (BZ2_PACKAGE_STRING "${BZ2_PACKAGE_NAME} ${BZ2_PACKAGE_VERSION}")
set (BZ2_PACKAGE_TARNAME "${BZ2_PACKAGE_NAME}{BZ2_PACKAGE_EXT}")
set (BZ2_PACKAGE_URL "http://www.hdfgroup.org")
set (BZ2_PACKAGE_BUGREPORT "help@hdfgroup.org")
set (BZ2_PACKAGE_SOVERSION "${BZ2_SOVERS_MAJOR}.${BZ2_SOVERS_MINOR}.${BZ2_SOVERS_RELEASE}")
set (BZ2_PACKAGE_SOVERSION_MAJOR "${BZ2_SOVERS_MAJOR}")

#-----------------------------------------------------------------------------
# Include some macros for reusable code
#-----------------------------------------------------------------------------
include (${BZ2_RESOURCES_DIR}/BZ2Macros.cmake)
SET_HDF_BUILD_TYPE()

#-----------------------------------------------------------------------------
# Setup output Directories
#-----------------------------------------------------------------------------
if (NOT BZ2_EXTERNALLY_CONFIGURED)
  set (CMAKE_RUNTIME_OUTPUT_DIRECTORY
      ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single Directory for all Executables."
  )
  set (CMAKE_LIBRARY_OUTPUT_DIRECTORY
      ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single Directory for all Libraries"
  )
  set (CMAKE_ARCHIVE_OUTPUT_DIRECTORY
      ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single Directory for all static libraries."
  )
  if (WIN32)
    set (CMAKE_TEST_OUTPUT_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${HDF_CFG_NAME})
    set (CMAKE_PDB_OUTPUT_DIRECTORY
        ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single Directory for all pdb files."
    )
  else ()
    set (CMAKE_TEST_OUTPUT_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${HDF_CFG_NAME})
  endif ()
else ()
  # if we are externally configured, but the project uses old cmake scripts
  # this may not be set
  if (NOT CMAKE_RUNTIME_OUTPUT_DIRECTORY)
    set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${EXECUTABLE_OUTPUT_PATH})
  endif ()
endif ()

#-----------------------------------------------------------------------------
# Targets built within this project are exported at Install time for use
# by other projects
#-----------------------------------------------------------------------------
if (NOT BZ2_EXPORTED_TARGETS)
  set (BZ2_EXPORTED_TARGETS "bz2-targets")
endif ()

#-----------------------------------------------------------------------------
# To include a library in the list exported by the project AT BUILD TIME,
# add it to this variable. This is NOT used by Make Install, but for projects
# which include bzip2 as a sub-project within their build tree
#-----------------------------------------------------------------------------
set_global_variable (BZ2_LIBRARIES_TO_EXPORT "")

#-----------------------------------------------------------------------------
# Run all the CMake configuration tests for our build environment
#-----------------------------------------------------------------------------
include (${BZ2_RESOURCES_DIR}/ConfigureChecks.cmake)

#-----------------------------------------------------------------------------
# Option to Build Shared and Static libs, default is shared
#-----------------------------------------------------------------------------
option (BUILD_SHARED_LIBS "Build Shared Libraries" ON)
set (CMAKE_POSITION_INDEPENDENT_CODE ON)

#-----------------------------------------------------------------------------
# bzip2 Library Settings
#-----------------------------------------------------------------------------
if (WIN32)
  set (BUILD_BZ2 1)
  add_definitions (-D_BIND_TO_CURRENT_VCLIBS_VERSION=1)
  add_definitions (-D_CRT_SECURE_NO_WARNINGS)
  add_definitions (-D_CONSOLE)
endif ()

if (MSVC)
  set (CMAKE_MFC_FLAG 0)
endif ()

set (MAKE_SYSTEM)
if (CMAKE_BUILD_TOOL MATCHES "make")
  set (MAKE_SYSTEM 1)
endif ()

set (CFG_INIT "/${CMAKE_CFG_INTDIR}")
if (MAKE_SYSTEM)
  set (CFG_INIT "")
endif ()

#-----------------------------------------------------------------------------
# Compiler specific flags : Shouldn't there be compiler tests for these
#-----------------------------------------------------------------------------
if (CMAKE_COMPILER_IS_GNUCC)
  set (CMAKE_C_FLAGS "${CMAKE_ANSI_CFLAGS} ${CMAKE_C_FLAGS} -std=c99 -fomit-frame-pointer -finline-functions -fno-common")
endif ()
if (CMAKE_COMPILER_IS_GNUCXX AND CMAKE_CXX_COMPILER_LOADED)
  set (CMAKE_CXX_FLAGS "${CMAKE_ANSI_CFLAGS} ${CMAKE_CXX_FLAGS} -fomit-frame-pointer -finline-functions -fno-common")
endif ()

#-----------------------------------------------------------------------------
# This is in here to help some of the GCC based IDES like Eclipse
# and code blocks parse the compiler errors and warnings better.
#-----------------------------------------------------------------------------
if (CMAKE_COMPILER_IS_GNUCC)
  set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fmessage-length=0")
endif ()
if (CMAKE_COMPILER_IS_GNUCXX AND CMAKE_CXX_COMPILER_LOADED)
  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fmessage-length=0")
endif ()

#-----------------------------------------------------------------------------
# Include user macros
#-----------------------------------------------------------------------------
include (UserMacros.cmake)

#-----------------------------------------------------------------------------
# All libs/tests/utils need the main include directories
#-----------------------------------------------------------------------------
INCLUDE_DIRECTORIES (${BZ2_BINARY_DIR} ${BZ2_SOURCE_DIR}/src ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})

add_subdirectory (src)

#-----------------------------------------------------------------------------
# Dashboard and Testing Settings
#-----------------------------------------------------------------------------
option (BUILD_TESTING "Build bzip2 Unit Testing" OFF)
if (BUILD_TESTING)
  set (DART_TESTING_TIMEOUT 1200 CACHE INTEGER
       "Timeout in seconds for each test (default 1200=20minutes)")
  enable_testing ()
  include (CTest)
endif ()

#-----------------------------------------------------------------------------
# Option to build utils
#-----------------------------------------------------------------------------
option (BZ2_BUILD_UTILS  "Build bzip2 Library Utilities" OFF)
if (BZ2_BUILD_UTILS)
  add_subdirectory (utils)
endif ()

#-----------------------------------------------------------------------------
# Add file(s) to CMake Install
#-----------------------------------------------------------------------------

include (CMakePackageConfigHelpers)

#-----------------------------------------------------------------------------
# Add Target(s) to CMake Install for import into other projects
#-----------------------------------------------------------------------------
if (NOT BZ2_EXTERNALLY_CONFIGURED)
  install (
      EXPORT ${BZ2_EXPORTED_TARGETS}
      DESTINATION ${BZ2_INSTALL_CMAKE_DIR}
      FILE ${BZ2_PACKAGE}${BZ2_PACKAGE_EXT}-targets.cmake
      COMPONENT configinstall
  )
endif ()

#-----------------------------------------------------------------------------
# Export all exported targets to the build tree for use by parent project
#-----------------------------------------------------------------------------
if (NOT BZ2_EXTERNALLY_CONFIGURED)
  export (
      TARGETS ${BZ2_LIBRARIES_TO_EXPORT} ${BZ2_LIB_DEPENDENCIES}
      FILE ${BZ2_PACKAGE}${BZ2_PACKAGE_EXT}-targets.cmake
  )
  export (PACKAGE ${BZ2_PACKAGE}${BZ2_PACKAGE_EXT})
endif ()

#-----------------------------------------------------------------------------
# Set includes needed for build
#-----------------------------------------------------------------------------
set (BZ2_INCLUDES_BUILD_TIME
    ${BZ2_SRC_DIR} ${BZ2_BINARY_DIR}
)

#-----------------------------------------------------------------------------
# Set variables needed for installation
#-----------------------------------------------------------------------------
set (BZ2_VERSION_STRING ${BZ2_PACKAGE_VERSION})
set (BZ2_VERSION_MAJOR  ${BZ2_PACKAGE_VERSION_MAJOR})
set (BZ2_VERSION_MINOR  ${BZ2_PACKAGE_VERSION_MINOR})

#-----------------------------------------------------------------------------
# Configure the bzip2-config.cmake file for the build directory
#-----------------------------------------------------------------------------
set (INCLUDE_INSTALL_DIR ${BZ2_INSTALL_INCLUDE_DIR} )
set (SHARE_INSTALL_DIR "${CMAKE_CURRENT_BINARY_DIR}/${BZ2_INSTALL_CMAKE_DIR}" )
set (CURRENT_BUILD_DIR "${CMAKE_CURRENT_BINARY_DIR}" )
configure_package_config_file (
    ${BZ2_RESOURCES_DIR}/bzip2-config.cmake.in
    "${BZ2_BINARY_DIR}/${BZ2_PACKAGE}${BZ2_PACKAGE_EXT}-config.cmake"
    INSTALL_DESTINATION "${BZ2_INSTALL_CMAKE_DIR}/bzip2"
    PATH_VARS INCLUDE_INSTALL_DIR SHARE_INSTALL_DIR CURRENT_BUILD_DIR
    INSTALL_PREFIX "${CMAKE_CURRENT_BINARY_DIR}"
)

#-----------------------------------------------------------------------------
# Configure the bzip2-config.cmake file for the install directory
#-----------------------------------------------------------------------------
set (INCLUDE_INSTALL_DIR ${BZ2_INSTALL_INCLUDE_DIR})
set (SHARE_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/${BZ2_INSTALL_CMAKE_DIR}" )
set (CURRENT_BUILD_DIR "${CMAKE_INSTALL_PREFIX}" )
configure_package_config_file (
    ${BZ2_RESOURCES_DIR}/bzip2-config.cmake.in
    "${BZ2_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${BZ2_PACKAGE}${BZ2_PACKAGE_EXT}-config.cmake"
    INSTALL_DESTINATION "${BZ2_INSTALL_CMAKE_DIR}/zip2"
    PATH_VARS INCLUDE_INSTALL_DIR SHARE_INSTALL_DIR CURRENT_BUILD_DIR
)
if (NOT BZ2_EXTERNALLY_CONFIGURED)
  install (
      FILES ${BZ2_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${BZ2_PACKAGE}${BZ2_PACKAGE_EXT}-config.cmake
      DESTINATION ${BZ2_INSTALL_CMAKE_DIR}
      COMPONENT configinstall
  )
endif ()

#-----------------------------------------------------------------------------
# Configure the bzip2-config-version.cmake file for the install directory
#-----------------------------------------------------------------------------
if (NOT BZ2_EXTERNALLY_CONFIGURED)
  configure_file (
      ${BZ2_RESOURCES_DIR}/bzip2-config-version.cmake.in
      ${BZ2_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${BZ2_PACKAGE}${BZ2_PACKAGE_EXT}-config-version.cmake @ONLY
  )
  install (
      FILES ${BZ2_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${BZ2_PACKAGE}${BZ2_PACKAGE_EXT}-config-version.cmake
      DESTINATION ${BZ2_INSTALL_CMAKE_DIR}/bzip2
      COMPONENT configinstall
  )
endif ()

#-----------------------------------------------------------------------------
# Configure the README.txt file for the binary package
#-----------------------------------------------------------------------------
BZ2_README_PROPERTIES ()

#-----------------------------------------------------------------------------
# Configure the LICENSE.txt file for the windows binary package
#-----------------------------------------------------------------------------
if (WIN32)
  configure_file (${BZ2_SOURCE_DIR}/LICENSE ${BZ2_BINARY_DIR}/LICENSE.txt @ONLY)
endif ()

#-----------------------------------------------------------------------------
# Check for Installation Utilities
#-----------------------------------------------------------------------------
if (WIN32)
  set (PF_ENV_EXT "(x86)")
  find_program (NSIS_EXECUTABLE NSIS.exe PATHS "$ENV{ProgramFiles}\\NSIS" "$ENV{ProgramFiles${PF_ENV_EXT}}\\NSIS")
  if(NOT CPACK_WIX_ROOT)
    file(TO_CMAKE_PATH "$ENV{WIX}" CPACK_WIX_ROOT)
  endif()
  find_program (WIX_EXECUTABLE candle  PATHS "${CPACK_WIX_ROOT}/bin")
endif ()

#-----------------------------------------------------------------------------
# Set the cpack variables
#-----------------------------------------------------------------------------
if (NOT BZ2_EXTERNALLY_CONFIGURED)
  set (CPACK_PACKAGE_VENDOR "HDF_Group")
  set (CPACK_PACKAGE_NAME "${BZ2_PACKAGE_NAME}")
  if (CDASH_LOCAL)
    set (CPACK_PACKAGE_VERSION "${BZ2_PACKAGE_VERSION}")
  else ()
    set (CPACK_PACKAGE_VERSION "${BZ2_PACKAGE_VERSION_STRING}")
  endif ()
  set (CPACK_PACKAGE_VERSION_MAJOR "${BZ2_PACKAGE_VERSION_MAJOR}")
  set (CPACK_PACKAGE_VERSION_MINOR "${BZ2_PACKAGE_VERSION_MINOR}")
  set (CPACK_PACKAGE_VERSION_PATCH "")
  set (CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
  set (CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/README.HDF")
  set (CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.HDF")
  set (CPACK_PACKAGE_RELOCATABLE TRUE)
  set (CPACK_PACKAGE_DESCRIPTION_SUMMARY "bzip2 Installation")
  set (CPACK_PACKAGE_INSTALL_DIRECTORY "${CPACK_PACKAGE_VENDOR}/${CPACK_PACKAGE_NAME}/${CPACK_PACKAGE_VERSION}")

  set (CPACK_GENERATOR "TGZ")
  if (WIN32)
    set (CPACK_GENERATOR "ZIP")

    if (NSIS_EXECUTABLE)
      list (APPEND CPACK_GENERATOR "NSIS")
    endif ()
    # Installers for 32- vs. 64-bit CMake:
    #  - Root install directory (displayed to end user at installer-run time)
    #  - "NSIS package/display name" (text used in the installer GUI)
    #  - Registry key used to store info about the installation
    set (CPACK_NSIS_PACKAGE_NAME "${BZ2_PACKAGE_STRING}")
    if (CMAKE_CL_64)
      set (CPACK_NSIS_INSTALL_ROOT "$PROGRAMFILES64")
      set (CPACK_PACKAGE_INSTALL_REGISTRY_KEY "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION} (Win64)")
    else ()
      set (CPACK_NSIS_INSTALL_ROOT "$PROGRAMFILES")
      set (CPACK_PACKAGE_INSTALL_REGISTRY_KEY "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}")
    endif ()
    # set the install/unistall icon used for the installer itself
    # There is a bug in NSI that does not handle full unix paths properly.
    #set (CPACK_NSIS_MUI_ICON "${BZ2_RESOURCES_DIR}\\\\BZ2.ico")
    #set (CPACK_NSIS_MUI_UNIICON "${BZ2_RESOURCES_DIR}\\\\BZ2.ico")
    # set the package header icon for MUI
    #set (CPACK_PACKAGE_ICON "${BZ2_RESOURCES_DIR}\\\\BZ2.bmp")
    set (CPACK_NSIS_DISPLAY_NAME "${CPACK_NSIS_PACKAGE_NAME}")
    set (CPACK_PACKAGE_INSTALL_DIRECTORY "${CPACK_PACKAGE_VENDOR}\\\\${CPACK_PACKAGE_NAME}\\\\${CPACK_PACKAGE_VERSION}")
    set (CPACK_NSIS_CONTACT "${BZ2_PACKAGE_BUGREPORT}")
    set (CPACK_NSIS_MODIFY_PATH ON)

    if (WIX_EXECUTABLE)
      list (APPEND CPACK_GENERATOR "WIX")
    endif ()
#WiX variables
    set (CPACK_WIX_UNINSTALL "1")
# .. variable:: CPACK_WIX_LICENSE_RTF
#  RTF License File
#
#  If CPACK_RESOURCE_FILE_LICENSE has an .rtf extension it is used as-is.
#
#  If CPACK_RESOURCE_FILE_LICENSE has an .txt extension it is implicitly
#  converted to RTF by the WiX Generator.
#  The expected encoding of the .txt file is UTF-8.
#
#  With CPACK_WIX_LICENSE_RTF you can override the license file used by the
#  WiX Generator in case CPACK_RESOURCE_FILE_LICENSE is in an unsupported
#  format or the .txt -> .rtf conversion does not work as expected.
    set (CPACK_RESOURCE_FILE_LICENSE "${BZ2_BINARY_DIR}/LICENSE.txt")
# .. variable:: CPACK_WIX_PRODUCT_ICON
#  The Icon shown next to the program name in Add/Remove programs.
#    set(CPACK_WIX_PRODUCT_ICON "${BZ2_RESOURCES_DIR}\\\\hdf.ico")
#
# .. variable:: CPACK_WIX_UI_BANNER
#
#  The bitmap will appear at the top of all installer pages other than the
#  welcome and completion dialogs.
#
#  If set, this image will replace the default banner image.
#
#  This image must be 493 by 58 pixels.
#
# .. variable:: CPACK_WIX_UI_DIALOG
#
#  Background bitmap used on the welcome and completion dialogs.
#
#  If this variable is set, the installer will replace the default dialog
#  image.
#
#  This image must be 493 by 312 pixels.
#
  elseif (APPLE)
    list (APPEND CPACK_GENERATOR "DragNDrop")
    set (CPACK_COMPONENTS_ALL_IN_ONE_PACKAGE ON)
    set (CPACK_PACKAGING_INSTALL_PREFIX "/${CPACK_PACKAGE_INSTALL_DIRECTORY}")
    #set (CPACK_PACKAGE_ICON "${BZ2_RESOURCES_DIR}/hdf.icns")

    option (BZ2_PACK_MACOSX_FRAMEWORK  "Package the BZIP2 Library in a Bundle" OFF)
    if (BZ2_PACK_MACOSX_FRAMEWORK AND BZ2_BUILD_FRAMEWORKS)
      set (CPACK_BUNDLE_NAME "${BZ2_PACKAGE_STRING}")
      set (CPACK_BUNDLE_LOCATION "/")    # make sure CMAKE_INSTALL_PREFIX ends in /
      set (CMAKE_INSTALL_PREFIX "/${CPACK_BUNDLE_NAME}.framework/Versions/${CPACK_PACKAGE_VERSION}/${CPACK_PACKAGE_NAME}/")
      set (CPACK_SHORT_VERSION_STRING "${CPACK_PACKAGE_VERSION}")
      #-----------------------------------------------------------------------------
      # Configure the Info.plist file for the install bundle
      #-----------------------------------------------------------------------------
      configure_file (
          ${BZ2_RESOURCES_DIR}/CPack.Info.plist.in
          ${BZ2_BINARY_DIR}/CMakeFiles/Info.plist @ONLY
      )
      configure_file (
          ${BZ2_RESOURCES_DIR}/PkgInfo.in
          ${BZ2_BINARY_DIR}/CMakeFiles/PkgInfo @ONLY
      )
      install (
          FILES ${BZ2_BINARY_DIR}/CMakeFiles/PkgInfo
          DESTINATION ..
      )
    endif ()
  else ()
    list (APPEND CPACK_GENERATOR "STGZ")
    set (CPACK_PACKAGING_INSTALL_PREFIX "/${CPACK_PACKAGE_INSTALL_DIRECTORY}")
    set (CPACK_COMPONENTS_ALL_IN_ONE_PACKAGE ON)

    set (CPACK_DEBIAN_PACKAGE_SECTION "Libraries")
    set (CPACK_DEBIAN_PACKAGE_MAINTAINER "${BZ2_PACKAGE_BUGREPORT}")

#      list (APPEND CPACK_GENERATOR "RPM")
    set (CPACK_RPM_PACKAGE_RELEASE "1")
    set (CPACK_RPM_COMPONENT_INSTALL ON)
    set (CPACK_RPM_PACKAGE_RELOCATABLE ON)
  endif ()

  # By default, do not warn when built on machines using only VS Express:
  if (NOT DEFINED CMAKE_INSTALL_SYSTEM_RUNTIME_LIBS_NO_WARNINGS)
    set (CMAKE_INSTALL_SYSTEM_RUNTIME_LIBS_NO_WARNINGS ON)
  endif ()
  include (InstallRequiredSystemLibraries)

  set (CPACK_INSTALL_CMAKE_PROJECTS "${BZ2_BINARY_DIR};bzip2;libraries;/")
  set (CPACK_INSTALL_CMAKE_PROJECTS "${BZ2_BINARY_DIR};bzip2;headers;/")
  set (CPACK_INSTALL_CMAKE_PROJECTS "${BZ2_BINARY_DIR};bzip2;configinstall;/")

  set (CPACK_ALL_INSTALL_TYPES Full Developer User)
  set (CPACK_INSTALL_TYPE_FULL_DISPLAY_NAME "Everything")

  set(CPACK_COMPONENTS_ALL libraries headers bz2documents configinstall)

  include (CPack)

  cpack_add_component_group(Runtime)

  cpack_add_component_group(Documents
      EXPANDED
      DESCRIPTION "Release notes for bzip2"
  )

  cpack_add_component_group(Development
      EXPANDED
      DESCRIPTION "All of the tools you'll need to develop applications"
  )

  cpack_add_component (libraries
      DISPLAY_NAME "bzip2 Libraries"
      GROUP Runtime
      INSTALL_TYPES Full Developer User
  )
  cpack_add_component (headers
      DISPLAY_NAME "bzip2 Headers"
      DEPENDS libraries
      GROUP Development
      INSTALL_TYPES Full Developer
  )
  cpack_add_component (bz2documents
      DISPLAY_NAME "bzip2 Documents"
      GROUP Documents
      INSTALL_TYPES Full Developer
  )
  cpack_add_component (configinstall
      DISPLAY_NAME "bzip2 CMake files"
      DEPENDS libraries
      GROUP Development
      INSTALL_TYPES Full Developer User
  )

endif ()
