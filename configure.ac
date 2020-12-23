#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_INIT([bz2], [1.0.6], [help@hdfgroup.org])
AC_CONFIG_SRCDIR([src/bzlib.h])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_AUX_DIR([bin])

## AM_INIT_AUTOMAKE takes a list of options that should be applied to
## every Makefile.am when automake is run.
AM_INIT_AUTOMAKE([foreign])

## AM_MAINTAINER_MODE turns off "rebuild rules" that contain dependencies
## for Makefiles, configure, etc.  If AM_MAINTAINER_MODE
## is *not* included here, these files will be rebuilt if out of date.
## This is a problem because if users try to build on a machine with
## the wrong versions of autoconf and automake, these files will be
## rebuilt with the wrong versions and bad things can happen.
## Also, CVS doesn't preserve dependencies between timestamps, so
## Makefiles will often think rebuilding needs to occur when it doesn't.
## Developers should './configure --enable-maintainer-mode' to turn on
## rebuild rules.
AM_MAINTAINER_MODE

AC_CANONICAL_HOST
AC_PREFIX_DEFAULT([`pwd`/bz2])

LT_INIT

## Source any special files that we need. These files normally aren't
## present but can be used by the maintainers to fine tune things like
## turning on debug or profiling flags for the compiler. The search order
## is:
##
##         CPU-VENDOR-OS
##         VENDOR-OS
##         CPU-OS
##         CPU-VENDOR
##         OS
##         VENDOR
##         CPU
##
## If the `OS' ends with a version number then remove it. For instance,
## `freebsd3.1' would become `freebsd'
##
case "$host_os" in
  aix4.*)       host_os_novers="aix4.x"     ;;
  aix5.*)       host_os_novers="aix5.x"     ;;
  freebsd*)     host_os_novers="freebsd"    ;;
  irix5.*)      host_os_novers="irix5.x"    ;;
  irix6.*)      host_os_novers="irix6.x"    ;;
  osf4.*)       host_os_novers="osf4.x"     ;;
  osf5.*)       host_os_novers="osf5.x"     ;;
  solaris2.*)   host_os_novers="solaris2.x" ;;
  *)            host_os_novers="$host_os"   ;;
esac

host_config="none"
for f in $host_cpu-$host_vendor-$host_os        \
         $host_cpu-$host_vendor-$host_os_novers \
         $host_vendor-$host_os                  \
         $host_vendor-$host_os_novers           \
         $host_cpu-$host_os                     \
         $host_cpu-$host_os_novers              \
         $host_cpu-$host_vendor                 \
         $host_os                               \
         $host_os_novers                        \
         $host_vendor                           \
         $host_cpu ; do
  AC_MSG_CHECKING([for config $f])
  if test -f "$srcdir/config/$f"; then
    host_config=$srcdir/config/$f
    AC_MSG_RESULT([found])
    break
  fi
  AC_MSG_RESULT([no])
done
if test "X$host_config" != "Xnone"; then
  CC_BASENAME="`echo $CC | cut -f1 -d' ' | xargs basename 2>/dev/null`"
  . $host_config
fi

## Checks for programs.
AC_PROG_CC
AC_PROG_CPP
AC_PROG_INSTALL

## Fix up the INSTALL macro if it's a relative path. We want the
## full-path to the binary instead.
case "$INSTALL" in
  *install-sh*)
    INSTALL='\${top_srcdir}/bin/install-sh -c'
    ;;
esac

## ----------------------------------------------------------------------
## Check which archiving tool to use. This needs to be done before
## the AC_PROG_LIBTOOL macro.
##
if test -z "$AR"; then
  AC_CHECK_PROGS([AR], [ar xar], [:], [$PATH])
fi

AC_SUBST([AR])

## Export the AR macro so that it will be placed in the libtool file
## correctly.
export AR

AC_PROG_MAKE_SET
AC_LIBTOOL_DLOPEN
AC_PROG_LIBTOOL

## Post processing to patch up some deficiencies in libtool
case $host_os in
  linux*)
    # If gcc is not used, need to set $wl to use "-Wl,"
    if $CC -v 2>&1 | grep '^gcc' > /dev/null ; then
      : using gcc
    else
      echo 'fixing $wl in' $ofile
ed - $ofile <<EOF 2> /dev/null
/^wl=""/s//wl="-Wl,"/
w
q
EOF
    fi
    ;;
esac


AC_SUBST([LD_LIBRARY_PATH]) LD_LIBRARY_PATH="$LD_LIBRARY_PATH"


## Are we building this in debug or production mode? (Remove the -g flag
## in production mode.)
AC_MSG_CHECKING(for build mode)
AC_ARG_ENABLE(production,
              [AC_HELP_STRING([--enable-production],
                              [Determines how to run the compiler.])])

case "X-$enable_production" in
  X-|X-yes)
    AC_MSG_RESULT([production])

    ## Remove the "-g" flag from compile line if it's in there.
    CFLAGS_temp=""
    if test -n "$CFLAGS"; then
      for d in $CFLAGS ; do
        if test "X$d" != "X-g"; then
          CFLAGS_temp="$CFLAGS_temp $d"
        fi
      done
      CFLAGS=$CFLAGS_temp
    fi


    CFLAGS="$CFLAGS $PROD_CFLAGS"
    CPPFLAGS="$CPPFLAGS $PROD_CPPFLAGS"
    ;;
  X-no)
    AC_MSG_RESULT([development])
    CFLAGS="$CFLAGS $DEBUG_CFLAGS"
    CPPFLAGS="$CPPFLAGS $DEBUG_CPPFLAGS"
    ;;
  *)
    AC_MSG_RESULT([user-defined])
    ;;
esac

## ======================================================================
## Checks for library functions
## ======================================================================

AC_MSG_CHECKING([for math library support])
AC_TRY_LINK([#include <math.h>], [volatile x = 37.927; k=1.2; sinh(x*k/cosh(x/k)],
            [AC_MSG_RESULT([yes])],
            [AC_MSG_RESULT([no]); LIBS="$LIBS -lm"])


## Checks for libraries.

## Checks for header files.
AC_HEADER_STDC
AC_CHECK_HEADERS([fcntl.h stdlib.h string.h])

## Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_TYPE_SIZE_T

## Checks for library functions.
AC_FUNC_ERROR_AT_LINE
##AC_FUNC_MALLOC
AC_FUNC_MEMCMP
AC_FUNC_VPRINTF
AC_CHECK_FUNCS([memset])

LT_OUTPUT

## Are we compiling static libraries, shared libraries, or both?  This
## is only used for the libhdf5.settings file. We can't just look at
## $enable_static and $enable_shared because if they're yes the ltconfig
## might have decided that one or the other is simply not possible.
## Therefore we have to ask the generated `libtool' shell script
## which 'features' it has enabled.
if (./libtool --features | grep '^enable shared libraries' > /dev/null); then
  enable_shared=yes
else
  enable_shared=no
fi

if (./libtool --features | grep '^enable static libraries' > /dev/null); then
  enable_static=yes
else
  enable_static=no
fi

if test "X$enable_static" = "Xyes" && test "X$enable_shared" = "Xyes"; then
  STATIC_SHARED="static, shared"
elif test "X$enable_static" = "Xyes"; then
  STATIC_SHARED="static"
elif test "X$enable_shared" = "Xyes"; then
  STATIC_SHARED="shared"
else
  STATIC_SHARED="none"
fi

AC_CONFIG_FILES([Makefile
                 utils/Makefile
                 src/Makefile])
AC_OUTPUT
