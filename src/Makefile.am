#############################################################################
##                          Library to build                               ##
#############################################################################
# Add libtool shared library version numbers to the bzip2 library
# See libtool versioning documentation online.
LT_REL_INTERFACE = 1
LT_REL_REVISION = 6

lib_LTLIBRARIES = libbz2.la

libbz2_la_LDFLAGS= -version-info $(LT_REL_INTERFACE):$(LT_REL_REVISION) $(AM_LDFLAGS)

CSOURCES = blocksort.c  \
      huffman.c    \
      crctable.c   \
      randtable.c  \
      compress.c   \
      decompress.c \
      bzlib.c

CHEADERS = bzlib.h bzlib_private.h

libbz2_la_SOURCES = $(CSOURCES)
include_HEADERS = $(CHEADERS)


