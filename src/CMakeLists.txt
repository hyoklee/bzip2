cmake_minimum_required (VERSION 3.2.2)
project (BZ2_SRC C)

#-----------------------------------------------------------------------------
# Define BZ2 Library
#-----------------------------------------------------------------------------
set (BZ2_SRCS
      ${BZ2_SRC_SOURCE_DIR}/blocksort.c
      ${BZ2_SRC_SOURCE_DIR}/huffman.c
      ${BZ2_SRC_SOURCE_DIR}/crctable.c
      ${BZ2_SRC_SOURCE_DIR}/randtable.c
      ${BZ2_SRC_SOURCE_DIR}/compress.c
      ${BZ2_SRC_SOURCE_DIR}/decompress.c
      ${BZ2_SRC_SOURCE_DIR}/bzlib.c
)
set (BZ2_PUBLIC_HEADERS
      ${BZ2_SRC_SOURCE_DIR}/bzlib.h
)

add_library (${BZ2_LIB_TARGET} STATIC ${BZ2_SRCS} ${BZ2_PUBLIC_HEADERS})
target_include_directories(${BZ2_LIB_TARGET} PRIVATE "${BZ2_SRC_SOURCE_DIR};${CMAKE_BINARY_DIR}")
TARGET_C_PROPERTIES (${BZ2_LIB_TARGET} STATIC)
target_link_libraries (${BZ2_LIB_TARGET} PRIVATE ${LINK_LIBS})
BZ2_SET_LIB_OPTIONS (${BZ2_LIB_TARGET} ${BZ2_LIB_NAME} STATIC)
set_target_properties(${BZ2_LIB_TARGET} PROPERTIES
    PUBLIC_HEADER "${BZ2_PUBLIC_HEADERS}"
    LINKER_LANGUAGE C
    INTERFACE_INCLUDE_DIRECTORIES "$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/include>"
)
set_global_variable (BZ2_LIBRARIES_TO_EXPORT ${BZ2_LIB_TARGET})
set (install_targets ${BZ2_LIB_TARGET})

if (BUILD_SHARED_LIBS)
  add_library (${BZ2_LIBSH_TARGET} SHARED ${BZ2_SRCS} ${BZ2_PUBLIC_HEADERS})
  target_include_directories(${BZ2_LIBSH_TARGET} PRIVATE "${BZ2_SRC_SOURCE_DIR};${CMAKE_BINARY_DIR}")
  TARGET_C_PROPERTIES (${BZ2_LIBSH_TARGET} SHARED)
  target_link_libraries (${BZ2_LIBSH_TARGET} PRIVATE ${LINK_LIBS})
  BZ2_SET_LIB_OPTIONS (${BZ2_LIBSH_TARGET} ${BZ2_LIB_NAME} SHARED)
  set_target_properties(${BZ2_LIBSH_TARGET} PROPERTIES
      PUBLIC_HEADER "${BZ2_PUBLIC_HEADERS}"
      LINKER_LANGUAGE C
      INTERFACE_INCLUDE_DIRECTORIES "$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/include>"
      INTERFACE_COMPILE_DEFINITIONS BZ_IMPORT=1
  )
  set_global_variable (BZ2_LIBRARIES_TO_EXPORT "${BZ2_LIBRARIES_TO_EXPORT};${BZ2_LIBSH_TARGET}")
  set (install_targets ${install_targets} ${BZ2_LIBSH_TARGET})
endif ()

#-----------------------------------------------------------------------------
# Add Target(s) to CMake Install for import into other projects
#-----------------------------------------------------------------------------
if (BZ2_EXPORTED_TARGETS)
  if (BUILD_SHARED_LIBS)
    INSTALL_TARGET_PDB (${BZ2_LIBSH_TARGET} ${BZ2_INSTALL_BIN_DIR} libraries)
  endif (BUILD_SHARED_LIBS)
  INSTALL_TARGET_PDB (${BZ2_LIB_TARGET} ${BZ2_INSTALL_BIN_DIR} libraries)

  install (
      TARGETS
          ${install_targets}
      EXPORT
          ${BZ2_EXPORTED_TARGETS}
      LIBRARY DESTINATION ${BZ2_INSTALL_LIB_DIR} COMPONENT libraries
      ARCHIVE DESTINATION ${BZ2_INSTALL_LIB_DIR} COMPONENT libraries
      RUNTIME DESTINATION ${BZ2_INSTALL_BIN_DIR} COMPONENT libraries
      FRAMEWORK DESTINATION ${BZ2_INSTALL_FWRK_DIR} COMPONENT libraries
      PUBLIC_HEADER DESTINATION ${BZ2_INSTALL_INCLUDE_DIR} COMPONENT headers
  )
endif ()
