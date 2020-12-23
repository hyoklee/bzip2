cmake_minimum_required (VERSION 3.2.2)
project (BZ2_UTILS C)

#-----------------------------------------------------------------------------
# Build BZ2 executables
#-----------------------------------------------------------------------------
add_executable (bzip2 bzip2.c)
target_include_directories(bzip2 PRIVATE "${BZ2_UTILS_SOURCE_DIR};${CMAKE_BINARY_DIR}")
TARGET_C_PROPERTIES (bzip2 STATIC)
target_link_libraries (bzip2 PRIVATE ${BZ2_LIB_TARGET})

add_executable (bzip2recover bzip2recover.c)
target_include_directories(bzip2recover PRIVATE "${BZ2_UTILS_SOURCE_DIR};${CMAKE_BINARY_DIR}")
TARGET_C_PROPERTIES (bzip2recover STATIC)
target_link_libraries (bzip2recover PRIVATE ${BZ2_LIB_TARGET})

##############################################################################
##############################################################################
###           T E S T I N G                                                ###
##############################################################################
##############################################################################

if (BUILD_TESTING)

  set (BZ2_REFERENCE_FILES
      sample1.bz2
      sample2.bz2
      sample3.bz2
      sample1.ref
      sample2.ref
      sample3.ref
  )

  #
  # copy test files from source dir to test dir
  #
  foreach (tst_file ${BZ2_REFERENCE_FILES})
    set (dest "${PROJECT_BINARY_DIR}/${tst_file}")
    #message (STATUS "... Copying ${BZ2_TESTS_SRC_DIR}/${tst_file} ${dest}")
    add_custom_command (
        TARGET     bzip2
        POST_BUILD
        COMMAND    ${CMAKE_COMMAND}
        ARGS       -E copy_if_different ${BZ2_TESTS_SRC_DIR}/${tst_file} ${dest}
    )
  endforeach ()

#-----------------------------------------------------------------------------
# Add BZ2 tests
#-----------------------------------------------------------------------------

  MACRO (ADD_BZ2_TEST testfile compressarg decompressarg)
    add_test (
        NAME BZ2-${testfile}
        COMMAND "${CMAKE_COMMAND}"
            -D "TEST_PROGRAM=$<TARGET_FILE:bzip2>"
            -D "TEST_ARGS:STRING=${compressarg}"
            -D "TEST_FOLDER=${PROJECT_BINARY_DIR}"
            -D "TEST_INPUT=${testfile}.ref"
            -D "TEST_OUTPUT=${testfile}.rb2"
            -D "TEST_EXPECT=0"
            -P "${BZ2_RESOURCES_DIR}/runTest.cmake"
    )
    if (NOT "${last_test}" STREQUAL "")
      set_tests_properties (BZ2-${testfile} PROPERTIES DEPENDS ${last_test})
    endif ()
    add_test (NAME BZ2-${testfile}-CMP COMMAND ${CMAKE_COMMAND} -E compare_files ${testfile}.bz2 ${testfile}.rb2)
    set_tests_properties (BZ2-${testfile}-CMP PROPERTIES DEPENDS BZ2-${testfile})
    add_test (
        NAME BZ2-${testfile}-D
        COMMAND "${CMAKE_COMMAND}"
            -D "TEST_PROGRAM=$<TARGET_FILE:bzip2>"
            -D "TEST_ARGS:STRING=${decompressarg}"
            -D "TEST_FOLDER=${PROJECT_BINARY_DIR}"
            -D "TEST_INPUT=${testfile}.bz2"
            -D "TEST_OUTPUT=${testfile}.tst"
            -D "TEST_EXPECT=0"
            -P "${BZ2_RESOURCES_DIR}/runTest.cmake"
    )
    set_tests_properties (BZ2-${testfile}-D PROPERTIES DEPENDS BZ2-${testfile}-CMP)
    add_test (NAME BZ2-${testfile}-D-CMP COMMAND ${CMAKE_COMMAND} -E compare_files ${testfile}.ref ${testfile}.tst)
    set_tests_properties (BZ2-${testfile}-D-CMP PROPERTIES DEPENDS BZ2-${testfile}-D)
  endmacro ()

  # Remove any output file left over from previous test run
  add_test (
      NAME BZ2-clear-objects
      COMMAND    ${CMAKE_COMMAND}
          -E remove
          sample1.rb2
          sample1.rb2.err
          sample2.rb2
          sample2.rb2.err
          sample3.rb2
          sample3.rb2.err
          sample1.tst
          sample1.tst.err
          sample2.tst
          sample2.tst.err
          sample3.tst
          sample3.tst.err
  )
  if (NOT "${last_test}" STREQUAL "")
    set_tests_properties (BZ2-clear-objects PROPERTIES DEPENDS ${last_test})
  endif ()
  set (last_test "BZ2-clear-objects")

  ADD_BZ2_TEST (sample1 -1 -d)
  ADD_BZ2_TEST (sample2 -2 -d)
  ADD_BZ2_TEST (sample3 -3 -ds)

endif ()

#-----------------------------------------------------------------------------
# Add Target(s) to CMake Install for import into other projects
#-----------------------------------------------------------------------------
if (BZ2_EXPORTED_TARGETS)
  install (
      TARGETS
          bzip2
          bzip2recover
      RUNTIME DESTINATION
          ${BZ2_INSTALL_BIN_DIR}
      COMPONENT
          applications
  )
endif ()
