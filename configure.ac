#
# Copyright (C) 2015 Alon Bar-Lev <alon.barlev@gmail.com>
#
AC_PREREQ(2.60)

define([VERSION_MAJOR], [1])
define([VERSION_MINOR], [5])
define([VERSION_FIX], [1])
define([VERSION_NUMBER], VERSION_MAJOR[.]VERSION_MINOR[.]VERSION_FIX)
define([VERSION_SUFFIX], [_rolling])

dnl Set to "1" for a first RPM release of a new version
PACKAGE_RPM_RELEASE="0.0.$(echo VERSION_SUFFIX | sed s/^_//)"

# We do not use m4_esyscmd_s to support older autoconf.
define([VERSION_STRING], m4_esyscmd(git describe 2>/dev/null | sed 's/^v//' | tr -d '\n'))
m4_ifval(VERSION_STRING, [], [define([VERSION_STRING], VERSION_NUMBER)])

AC_INIT([netdata], VERSION_STRING[]VERSION_SUFFIX)

AM_MAINTAINER_MODE([disable])
if test x"$USE_MAINTAINER_MODE" = xyes; then
AC_MSG_NOTICE(***************** MAINTAINER MODE *****************)
PACKAGE_BUILT_DATE=$(date '+%d %b %Y')
fi

PACKAGE_RPM_VERSION="VERSION_NUMBER"
AC_SUBST([PACKAGE_RPM_VERSION])
AC_SUBST([PACKAGE_RPM_RELEASE])


# -----------------------------------------------------------------------------
# autoconf initialization

AC_CONFIG_AUX_DIR([.])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_SRCDIR([src/main.c])
AM_INIT_AUTOMAKE
AC_CANONICAL_HOST
AC_PROG_CC
AC_PROG_INSTALL
PKG_PROG_PKG_CONFIG
AC_USE_SYSTEM_EXTENSIONS


# -----------------------------------------------------------------------------
# configurable options

AC_ARG_ENABLE(
    [plugin-nfacct],
    [AS_HELP_STRING([--enable-plugin-nfacct], [enable nfacct plugin, requires root])],
    ,
    [enable_plugin_nfacct="no"]
)
AC_ARG_ENABLE(
    [plugin-freeipmi],
    [AS_HELP_STRING([--enable-plugin-freeipmi], [enable freeipmi plugin])],
    ,
    [enable_plugin_freeipmi="detect"]
)
AC_ARG_ENABLE(
    [pedantic],
    [AS_HELP_STRING([--enable-pedantic], [enable pedantic compiler warnings])],
    ,
    [enable_pedantic="no"]
)
AC_ARG_WITH(
    [webdir],
    [AS_HELP_STRING([--with-webdir], [location of webdir @<:@PKGDATADIR/web@:>@])],
    [webdir="${withval}"],
    [webdir="\$(pkgdatadir)/web"]
)
AC_ARG_WITH(
    [libcap],
    [AS_HELP_STRING([--with-libcap], [build with libcap])],
    ,
    [with_libcap="detect"]
)
AC_ARG_WITH(
    [zlib],
    [AS_HELP_STRING([--with-zlib], [build with zlib])],
    ,
    [with_zlib="yes"]
)
AC_ARG_WITH(
    [math],
    [AS_HELP_STRING([--with-math], [build with math])],
    ,
    [with_math="yes"]
)
AC_ARG_WITH(
    [user],
    [AS_HELP_STRING([--with-user], [use this user to drop privilege])],
    ,
    [with_user="nobody"]
)
AC_ARG_ENABLE(
    [x86-sse],
    [AS_HELP_STRING([--disable-x86-sse], [SSE/SS2 optimizations on x86 @<:@default enabled@:>@])],
    ,
    [enable_x86_sse="yes"]
)
AC_ARG_ENABLE(
    [lto],
    [AS_HELP_STRING([--disable-lto], [Link Time Optimizations @<:@default enabled@:>@])],
    ,
    [enable_lto="detect"]
)


# -----------------------------------------------------------------------------
# netdata required checks

# fails on centos6
#AX_CHECK_ENABLE_DEBUG()

AX_GCC_FUNC_ATTRIBUTE([returns_nonnull])
AX_GCC_FUNC_ATTRIBUTE([malloc])
AX_GCC_FUNC_ATTRIBUTE([noreturn])
AX_GCC_FUNC_ATTRIBUTE([format])
AX_GCC_FUNC_ATTRIBUTE([warn_unused_result])

AC_CHECK_FUNCS_ONCE(accept4)
AC_CHECK_TYPES([struct timespec, clockid_t], [], [], [[#include <time.h>]])
AC_SEARCH_LIBS([clock_gettime], [rt posix4])
AC_CHECK_FUNCS([clock_gettime])
AC_CHECK_FUNCS([sched_setscheduler sched_get_priority_min sched_get_priority_max nice])

AC_TYPE_INT8_T
AC_TYPE_INT16_T
AC_TYPE_INT32_T
AC_TYPE_INT64_T
AC_TYPE_UINT8_T
AC_TYPE_UINT16_T
AC_TYPE_UINT32_T
AC_TYPE_UINT64_T
AC_C_INLINE
AC_FUNC_STRERROR_R
AC_C__GENERIC
AC_C___ATOMIC
# AC_C_STMT_EXPR
AC_CHECK_SIZEOF([void *])
AC_CANONICAL_HOST
AC_HEADER_MAJOR
AC_HEADER_RESOLV

AC_CHECK_HEADERS_ONCE([sys/prctl.h])


# -----------------------------------------------------------------------------
# operating system detection

AC_MSG_CHECKING([operating system])
case "$host_os" in
freebsd*)
    build_target=freebsd
    ;;
darwin*)
    build_target=macos
    LDFLAGS="${LDFLAGS} -framework CoreFoundation -framework IOKit"
    ;;
*)
    build_target=linux
    ;;
esac

AM_CONDITIONAL([FREEBSD], [test "${build_target}" = "freebsd"])
AM_CONDITIONAL([MACOS], [test "${build_target}" = "macos"])
AM_CONDITIONAL([LINUX], [test "${build_target}" = "linux"])
AC_MSG_RESULT([${build_target}])


# -----------------------------------------------------------------------------
# pthreads

ACX_PTHREAD(, [AC_MSG_ERROR([Cannot initialize pthread environment])])
LIBS="${PTHREAD_LIBS} ${LIBS}"
CFLAGS="${CFLAGS} ${PTHREAD_CFLAGS}"
CC="${PTHREAD_CC}"


# -----------------------------------------------------------------------------
# libm

AC_ARG_VAR([MATH_CFLAGS], [C compiler flags for math])
AC_ARG_VAR([MATH_LIBS], [linker flags for math])
if test -z "${MATH_LIBS}"; then
    AC_CHECK_LIB(
        [m],
        [sin],
        [MATH_LIBS="-lm"]
    )
fi
test "${with_math}" = "yes" -a -z "${MATH_LIBS}" && AC_MSG_ERROR([math required but not found])

AC_MSG_CHECKING([if libm should be used])
if test "${with_math}" != "no" -a ! -z "${MATH_LIBS}"; then
    with_math="yes"
    AC_DEFINE([STORAGE_WITH_MATH], [1], [math usability])
    OPTIONAL_MATH_CFLAGS="${MATH_CFLAGS}"
    OPTIONAL_MATH_LIBS="${MATH_LIBS}"
else
    with_math="no"
fi
AC_MSG_RESULT([${with_math}])


# -----------------------------------------------------------------------------
# zlib

PKG_CHECK_MODULES(
    [ZLIB],
    [zlib],
    [have_zlib=yes],
    [have_zlib=no]
)
test "${with_zlib}" = "yes" -a "${have_zlib}" != "yes" && AC_MSG_ERROR([zlib required but not found. Try installing 'zlib1g-dev' or 'zlib-devel'.])

AC_MSG_CHECKING([if zlib should be used])
if test "${with_zlib}" != "no" -a "${have_zlib}" = "yes"; then
    with_zlib="yes"
    AC_DEFINE([NETDATA_WITH_ZLIB], [1], [zlib usability])
    OPTIONAL_ZLIB_CLFAGS="${ZLIB_CFLAGS}"
    OPTIONAL_ZLIB_LIBS="${ZLIB_LIBS}"
else
    with_zlib="no"
fi
AC_MSG_RESULT([${with_zlib}])


# -----------------------------------------------------------------------------
# libuuid

PKG_CHECK_MODULES(
    [UUID],
    [uuid],
    [have_uuid=yes],
    [AC_MSG_ERROR([libuuid required but not found. Try installing 'uuid-dev' or 'libuuid-devel'.])]
)
AC_DEFINE([NETDATA_WITH_UUID], [1], [uuid usability])
OPTIONAL_UUID_CLFAGS="${UUID_CFLAGS}"
OPTIONAL_UUID_LIBS="${UUID_LIBS}"


# -----------------------------------------------------------------------------
# compiler options

AC_ARG_VAR([SSE_CANDIDATE], [C compiler flags for SSE])
AS_CASE([$host_cpu],
    [i?86], [SSE_CANDIDATE="yes"]
)
AC_SUBST([SSE_CANDIDATE])
if test "${SSE_CANDIDATE}" = "yes" -a "${enable_x86_sse}" = "yes"; then
    opt="-msse2 -mfpmath=sse"
    AX_CHECK_COMPILE_FLAG(${opt}, [CFLAGS="${CFLAGS} ${opt}"], [])
fi

if test "${GCC}" = "yes"; then
    AC_DEFINE_UNQUOTED([likely(x)], [__builtin_expect(!!(x), 1)], [gcc branch optimization])
    AC_DEFINE_UNQUOTED([unlikely(x)], [__builtin_expect(!!(x), 0)], [gcc branch optimization])
else
    AC_DEFINE_UNQUOTED([likely(x)], [(x)], [gcc branch optimization])
    AC_DEFINE_UNQUOTED([unlikely(x)], [(x)], [gcc branch optimization])
fi

if test "${enable_pedantic}" = "yes"; then
    enable_strict="yes"
    CFLAGS="${CFLAGS} -pedantic -Wall -Wextra -Wno-long-long"
fi


# -----------------------------------------------------------------------------
# memory allocation library

AC_MSG_CHECKING([for memory allocator])
TS_CHECK_JEMALLOC
if test "$has_jemalloc" = "1"; then
    AC_DEFINE([ENABLE_JEMALLOC], [1], [compile and link with jemalloc])
    AC_MSG_RESULT([jemalloc])
else
    TS_CHECK_TCMALLOC
    if test "$has_tcmalloc" = "1"; then
        AC_DEFINE([ENABLE_TCMALLOC], [1], [compile and link with tcmalloc])
        AC_MSG_RESULT([tcmalloc])
    else
        AC_MSG_RESULT([system])
        AC_C_MALLOPT
        AC_C_MALLINFO
    fi
fi


# -----------------------------------------------------------------------------
# libcap

PKG_CHECK_MODULES(
    [LIBCAP],
    [libcap],
    [AC_CHECK_LIB([cap], [cap_get_proc, cap_set_proc],
        [AC_CHECK_HEADER(
            [sys/capability.h],
            [have_libcap=yes],
            [have_libcap=no]
        )],
        [have_libcap=no]
    )],
    [have_libcap=no]
)
test "${with_libcap}" = "yes" -a "${have_libcap}" != "yes" && AC_MSG_ERROR([libcap required but not found.])

AC_MSG_CHECKING([if libcap should be used])
if test "${with_libcap}" != "no" -a "${have_libcap}" = "yes"; then
    with_libcap="yes"
    AC_DEFINE([HAVE_CAPABILITY], [1], [libcap usability])
    OPTIONAL_LIBCAP_CLFAGS="${LIBCAP_CFLAGS}"
    OPTIONAL_LIBCAP_LIBS="${LIBCAP_LIBS}"
else
    with_libcap="no"
fi
AC_MSG_RESULT([${with_libcap}])
AM_CONDITIONAL([ENABLE_CAPABILITY], [test "${with_libcap}" = "yes"])


# -----------------------------------------------------------------------------
# apps.plugin

AC_MSG_CHECKING([if apps.plugin should be enabled])
if test "${build_target}" != "macos"; then
    enable_plugin_apps="yes"
else
    enable_plugin_apps="no"
fi
AC_MSG_RESULT([${enable_plugin_apps}])
AM_CONDITIONAL([ENABLE_PLUGIN_APPS], [test "${enable_plugin_apps}" = "yes"])


# -----------------------------------------------------------------------------
# freeipmi.plugin - libipmimonitoring

PKG_CHECK_MODULES(
    [IPMIMONITORING],
    [libipmimonitoring],
    [have_ipmimonitoring=yes],
    [have_ipmimonitoring=no]
)
test "${enable_plugin_freeipmi}" = "yes" -a "${have_ipmimonitoring}" != "yes" && \
    AC_MSG_ERROR([ipmimonitoring required but not found. Try installing 'libipmimonitoring-dev' or 'libipmimonitoring-devel'])

AC_MSG_CHECKING([if freeipmi.plugin should be enabled])
if test "${enable_plugin_freeipmi}" != "no" -a "${have_ipmimonitoring}" = "yes"; then
    enable_plugin_freeipmi="yes"
    AC_DEFINE([HAVE_FREEIPMI], [1], [ipmimonitoring usability])
    OPTIONAL_IPMIMONITORING_CLFAGS="${IPMIMONITORING_CFLAGS}"
    OPTIONAL_IPMIMONITORING_LIBS="${IPMIMONITORING_LIBS}"
else
    enable_plugin_freeipmi="no"
fi
AC_MSG_RESULT([${enable_plugin_freeipmi}])
AM_CONDITIONAL([ENABLE_PLUGIN_FREEIPMI], [test "${enable_plugin_freeipmi}" = "yes"])


# -----------------------------------------------------------------------------
# nfacct.plugin - libmnl, libnetfilter_acct

AC_CHECK_HEADERS_ONCE([linux/netfilter/nfnetlink_conntrack.h])

PKG_CHECK_MODULES(
    [NFACCT],
    [libnetfilter_acct],
    [have_libnetfilter_acct=yes],
    [have_libnetfilter_acct=no]
)

PKG_CHECK_MODULES(
    [LIBMNL],
    [libmnl],
    [have_libmnl=yes],
    [have_libmnl=no]
)

test "${enable_plugin_nfacct}" = "yes" -a "${have_libnetfilter_acct}" != "yes" && \
    AC_MSG_ERROR([netfilter_acct required but not found])

test "${enable_plugin_nfacct}" = "yes" -a "${have_libmnl}" != "yes" && \
    AC_MSG_ERROR([libmnl required but not found. Try installing 'libmnl-dev' or 'libmnl-devel'])

AC_MSG_CHECKING([if nfacct.plugin should be enabled])
if test "${enable_plugin_nfacct}" != "no" -a "${have_libnetfilter_acct}" = "yes" -a "${have_libmnl}" = "yes"; then
    enable_plugin_nfacct="yes"
    AC_DEFINE([HAVE_LIBMNL], [1], [libmnl usability])
    AC_DEFINE([HAVE_LIBNETFILTER_ACCT], [1], [libnetfilter_acct usability])
    AC_DEFINE([INTERNAL_PLUGIN_NFACCT], [1], [nfacct plugin usability])
    OPTIONAL_NFACCT_CLFAGS="${NFACCT_CFLAGS} ${LIBMNL_CFLAGS}"
    OPTIONAL_NFACCT_LIBS="${NFACCT_LIBS} ${LIBMNL_LIBS}"
else
    enable_plugin_nfacct="no"
fi
AC_MSG_RESULT([${enable_plugin_nfacct}])
AM_CONDITIONAL([ENABLE_PLUGIN_NFACCT], [test "${enable_plugin_nfacct}" = "yes"])


# -----------------------------------------------------------------------------
# Link-Time-Optimization

if test "${enable_lto}" != "no"; then
    opt="-flto"
    AX_CHECK_COMPILE_FLAG(${opt}, [have_lto=yes], [have_lto=no])
fi
if test "${have_lto}" = "yes"; then
    oCFLAGS="${CFLAGS}"
    CFLAGS="${CFLAGS} -flto ${OPTIONAL_MATH_CLFAGS} ${OPTIONAL_NFACCT_CLFAGS} ${OPTIONAL_ZLIB_CLFAGS} ${OPTIONAL_UUID_CLFAGS} ${OPTIONAL_LIBCAP_CFLAGS} ${OPTIONAL_IPMIMONITORING_CFLAGS}"
    ac_cv_c_lto_cross_compile="${enable_lto}"
    test "${ac_cv_c_lto_cross_compile}" != "yes" && ac_cv_c_lto_cross_compile="no"
    AC_C_LTO
    CFLAGS="${oCFLAGS}"
    test "${ac_cv_c_lto}" != "yes" && have_lto="no"
fi
test "${enable_lto}" = "yes" -a "${have_lto}" != "yes" && \
    AC_MSG_ERROR([LTO is required but is not available.])
AC_MSG_CHECKING([if LTO should be enabled])
if test "${enable_lto}" != "no" -a "${have_lto}" = "yes"; then
    enable_lto="yes"
    CFLAGS="${CFLAGS} -flto"
else
    enable_lto="no"
fi
AC_MSG_RESULT([${enable_lto}])


# -----------------------------------------------------------------------------

AC_DEFINE_UNQUOTED([NETDATA_USER], ["${with_user}"], [use this user to drop privileged])

AC_SUBST([varlibdir], ["\$(localstatedir)/lib/netdata"])
AC_SUBST([registrydir], ["\$(localstatedir)/lib/netdata/registry"])
AC_SUBST([cachedir], ["\$(localstatedir)/cache/netdata"])
AC_SUBST([chartsdir], ["\$(libexecdir)/netdata/charts.d"])
AC_SUBST([nodedir], ["\$(libexecdir)/netdata/node.d"])
AC_SUBST([pythondir], ["\$(libexecdir)/netdata/python.d"])
AC_SUBST([configdir], ["\$(sysconfdir)/netdata"])
AC_SUBST([logdir], ["\$(localstatedir)/log/netdata"])
AC_SUBST([pluginsdir], ["\$(libexecdir)/netdata/plugins.d"])
AC_SUBST([webdir])

AC_SUBST([OPTIONAL_MATH_CLFAGS])
AC_SUBST([OPTIONAL_MATH_LIBS])
AC_SUBST([OPTIONAL_NFACCT_CLFAGS])
AC_SUBST([OPTIONAL_NFACCT_LIBS])
AC_SUBST([OPTIONAL_ZLIB_CLFAGS])
AC_SUBST([OPTIONAL_ZLIB_LIBS])
AC_SUBST([OPTIONAL_UUID_CLFAGS])
AC_SUBST([OPTIONAL_UUID_LIBS])
AC_SUBST([OPTIONAL_LIBCAP_CFLAGS])
AC_SUBST([OPTIONAL_LIBCAP_LIBS])
AC_SUBST([OPTIONAL_IPMIMONITORING_CFLAGS])
AC_SUBST([OPTIONAL_IPMIMONITORING_LIBS])

AC_CONFIG_FILES([
    Makefile
    charts.d/Makefile
    conf.d/Makefile
    netdata.spec
    python.d/Makefile
    node.d/Makefile
    plugins.d/Makefile
    src/Makefile
    system/Makefile
    web/Makefile
    contrib/Makefile
])
AC_OUTPUT

test "${with_math}" != "yes" && AC_MSG_WARN([You are building without math. math allows accurate calculations. It should be enabled.]) || :
test "${with_zlib}" != "yes" && AC_MSG_WARN([You are building without zlib. zlib allows netdata to transfer a lot less data with web clients. It should be enabled.]) || :
