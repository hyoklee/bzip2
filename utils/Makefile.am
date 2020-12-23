#############################################################################
##                      Setup for building programs                        ##
#############################################################################

AM_CPPFLAGS=-I$(top_srcdir)/src  -I$(top_srcdir)/test

#############################################################################
##                              Testing                                    ##
#############################################################################

bin_PROGRAMS = bzip2 bzip2recover

bzip2_SOURCES = bzip2.c
bzip2_LDADD = ../src/libbz2.la
bzip2_DEPENDENCIES = ../src/libbz2.la


bzip2recover_SOURCES = bzip2recover.c
bzip2recover_LDADD = ../src/libbz2.la
bzip2recover_DEPENDENCIES = ../src/libbz2.la

#############################################################################
##                          Testing example                                ##
#############################################################################
check: $(bin_PROGRAMS) 
	@cat $(top_srcdir)/tests/words1
	./bzip2 -1  < $(top_srcdir)/tests/sample1.ref > sample1.rb2
	./bzip2 -2  < $(top_srcdir)/tests/sample2.ref > sample2.rb2
	./bzip2 -3  < $(top_srcdir)/tests/sample3.ref > sample3.rb2
	./bzip2 -d  < $(top_srcdir)/tests/sample1.bz2 > sample1.tst
	./bzip2 -d  < $(top_srcdir)/tests/sample2.bz2 > sample2.tst
	./bzip2 -ds < $(top_srcdir)/tests/sample3.bz2 > sample3.tst
	cmp $(top_srcdir)/tests/sample1.bz2 sample1.rb2 
	cmp $(top_srcdir)/tests/sample2.bz2 sample2.rb2
	cmp $(top_srcdir)/tests/sample3.bz2 sample3.rb2
	cmp sample1.tst $(top_srcdir)/tests/sample1.ref
	cmp sample2.tst $(top_srcdir)/tests/sample2.ref
	cmp sample3.tst $(top_srcdir)/tests/sample3.ref
	@cat $(top_srcdir)/tests/words3

#############################################################################
##                          And the cleanup                                ##
#############################################################################

DISTCLEANFILES = sample1.rb2 sample2.rb2 sample3.rb2 \
	sample1.tst sample2.tst sample3.tst
