# This file is part of Cockpit.
#
# Copyright (C) 2013 Red Hat, Inc.
#
# Cockpit is free software; you can redistribute it and/or modify it
# under the terms of the GNU Lesser General Public License as published by
# the Free Software Foundation; either version 2.1 of the License, or
# (at your option) any later version.
#
# Cockpit is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
# Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public License
# along with Cockpit; If not, see <http://www.gnu.org/licenses/>.

AC_INIT([Cockpit],
        [m4_esyscmd_s([if [ -f .tarball ]; then cat .tarball; else echo $(git describe | tr - . ); fi])],
        [devel@lists.cockpit-project.org],
        [cockpit],
        [https://cockpit-project.org/])

# The above command needs to run in a git repository or `.tarball` file needs to be
# present. If that didn't work bail early. Cockpit needs to know its own version number
AC_MSG_CHECKING(that PACKAGE_VERSION is properly set)
if test -z "$PACKAGE_VERSION"; then
  AC_MSG_RESULT(no)
  AC_MSG_ERROR(Please build from a .git checkout or .tar.xz tarball)
else
  AC_MSG_RESULT(yes)
fi

AC_CONFIG_SRCDIR([src])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_AUX_DIR([tools])
AC_CONFIG_MACRO_DIR([tools])

AM_INIT_AUTOMAKE([1.9 foreign dist-xz no-dist-gzip subdir-objects tar-pax])
AM_MAINTAINER_MODE([enable])

AC_USE_SYSTEM_EXTENSIONS

AM_PROG_AR
AC_PROG_CC
AC_PROG_CC_C99
AC_PROG_RANLIB
AC_ISC_POSIX
AC_HEADER_STDC
AC_PROG_LN_S

# HACK - Prevent valgrind false-positives due to GHashTable trickery
AC_CHECK_SIZEOF([void *])
AM_CONDITIONAL([VALGRIND_MINADDR_HACK], [test "$ac_cv_sizeof_void_p" -eq 8])

# Unrelated to above: enable more valgrind workarounds for other problems.
AC_CHECK_HEADERS([valgrind/valgrind.h])

m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

AC_MSG_CHECKING([whether to install to prefix only])
AC_ARG_ENABLE([prefix-only],
              [AS_HELP_STRING([--enable-prefix-only], [Whether to install to prefix only])],
              [], [enable_prefix_only=no])
AC_MSG_RESULT($enable_prefix_only)

# Initialization
#

GLIB_VERSION="2.50"
LIBSSH_VERSION="0.8.5"

GIO_REQUIREMENT="gio-unix-2.0 >= $GLIB_VERSION gio-2.0 >= $GLIB_VERSION"
LIBSYSTEMD_REQUIREMENT="libsystemd >= 235"
JSON_GLIB_REQUIREMENT="json-glib-1.0 >= 1.4"
POLKIT_REQUIREMENT="polkit-agent-1 >= 0.105"
GNUTLS_REQUIREMENT="gnutls >= 3.6.0"
KRB5_REQUIREMENT="krb5-gssapi >= 1.11 krb5 >= 1.11"

PKG_CHECK_MODULES(GIO, [$GIO_REQUIREMENT])
GLIB_VERSION_DEF="GLIB_VERSION_$(echo $GLIB_VERSION | tr '.' '_')"
GIO_CFLAGS="$GIO_CFLAGS -DGLIB_VERSION_MIN_REQUIRED=GLIB_VERSION_2_44" # GSimpleAsyncResult
GIO_CFLAGS="$GIO_CFLAGS -DGLIB_VERSION_MAX_ALLOWED=$GLIB_VERSION_DEF"

PKG_CHECK_MODULES(LIBSYSTEMD, [$LIBSYSTEMD_REQUIREMENT])
PKG_CHECK_MODULES(JSON_GLIB, [$JSON_GLIB_REQUIREMENT])
PKG_CHECK_MODULES(GNUTLS, [$GNUTLS_REQUIREMENT])
PKG_CHECK_MODULES(KRB5, [$KRB5_REQUIREMENT])

COCKPIT_CFLAGS="$GIO_CFLAGS $JSON_GLIB_CFLAGS $LIBSYSTEMD_CFLAGS"
COCKPIT_LIBS="$GIO_LIBS $JSON_GLIB_LIBS $LIBSYSTEMD_LIBS -lutil -lm"
AC_SUBST(COCKPIT_CFLAGS)
AC_SUBST(COCKPIT_LIBS)

# bridge with optional polkit
AC_ARG_ENABLE(polkit, AC_HELP_STRING([--disable-polkit], [Disable usage of polkit]),,enable_polkit=yes)

if test "$enable_polkit" = "yes"; then
  PKG_CHECK_MODULES(POLKIT, [$POLKIT_REQUIREMENT])
  COCKPIT_BRIDGE_CFLAGS="$COCKPIT_CFLAGS $POLKIT_CFLAGS"
  COCKPIT_BRIDGE_LIBS="$COCKPIT_LIBS $POLKIT_LIBS"
  AC_DEFINE_UNQUOTED([WITH_POLKIT], [], [Build with polkit])
else
  COCKPIT_BRIDGE_CFLAGS="$COCKPIT_CFLAGS"
  COCKPIT_BRIDGE_LIBS="$COCKPIT_LIBS"
fi
AM_CONDITIONAL(WITH_POLKIT, test "$enable_polkit" = "yes")
AC_SUBST(COCKPIT_BRIDGE_CFLAGS)
AC_SUBST(COCKPIT_BRIDGE_LIBS)

COCKPIT_PCP_CFLAGS="$COCKPIT_CFLAGS"
COCKPIT_PCP_LIBS="$COCKPIT_LIBS"
AC_SUBST(COCKPIT_PCP_CFLAGS)
AC_SUBST(COCKPIT_PCP_LIBS)

COCKPIT_SESSION_CFLAGS="$KRB5_CFLAGS $LIBSYSTEMD_CFLAGS"
COCKPIT_SESSION_LIBS="$KRB5_LIBS $LIBSYSTEMD_LIBS"
AC_SUBST(COCKPIT_SESSION_CFLAGS)
AC_SUBST(COCKPIT_SESSION_LIBS)

PAM_COCKPIT_CERT_CFLAGS="$COCKPIT_CFLAGS"
PAM_COCKPIT_CERT_LIBS=""
AC_SUBST(PAM_COCKPIT_CERT_CFLAGS)
AC_SUBST(PAM_COCKPIT_CERT_LIBS)

COCKPIT_WS_CFLAGS="$COCKPIT_CFLAGS"
COCKPIT_WS_LIBS="$COCKPIT_LIBS"
AC_SUBST(COCKPIT_WS_CFLAGS)
AC_SUBST(COCKPIT_WS_LIBS)

COCKPIT_TLS_CFLAGS="$GNUTLS_CFLAGS"
COCKPIT_TLS_LIBS="$GNUTLS_LIBS"
AC_SUBST(COCKPIT_TLS_CFLAGS)
AC_SUBST(COCKPIT_TLS_LIBS)

# whether to build cockpit-ssh
AC_ARG_ENABLE(ssh, AC_HELP_STRING([--disable-ssh], [Disable cockpit-ssh build and libssh dependency]))
AC_MSG_CHECKING([build with cockpit-ssh])
if test "$enable_ssh" != "no"; then
  enable_ssh="yes"
  AC_MSG_RESULT([$enable_ssh])
  PKG_CHECK_MODULES(LIBSSH, [libssh >= $LIBSSH_VERSION])
  COCKPIT_SSH_SESSION_CFLAGS="$COCKPIT_CFLAGS $LIBSSH_CFLAGS $KRB5_CFLAGS"
  COCKPIT_SSH_SESSION_LIBS="$COCKPIT_LIBS $LIBSSH_LIBS $KRB5_LIBS"
  AC_SUBST(COCKPIT_SSH_SESSION_LIBS)
  AC_SUBST(COCKPIT_SSH_SESSION_CFLAGS)
else
  enable_ssh="no"
  key_auth="no"
  AC_MSG_RESULT([$enable_ssh])
fi

AM_CONDITIONAL(WITH_COCKPIT_SSH, test "$enable_ssh" = "yes")

# pam
AC_CHECK_HEADER([security/pam_appl.h], ,
  [AC_MSG_ERROR([Couldn't find PAM headers. Try installing pam-devel])]
)
PAM_LIBS="-lpam"
COCKPIT_SESSION_LIBS="$COCKPIT_SESSION_LIBS $PAM_LIBS"
COCKPIT_WS_LIBS="$COCKPIT_WS_LIBS $PAM_LIBS"

# pam module directory
AC_ARG_WITH([pamdir],
            [AC_HELP_STRING([--with-pamdir=DIR],
                             [directory to install pam modules in])],
             [], [with_pamdir='${libdir}/security'])
pamdir=$with_pamdir
AC_SUBST(pamdir)

# crypt
AC_CHECK_HEADER([crypt.h], ,
  [AC_MSG_ERROR([Couldn't find crypt headers. Try installing glibc-headers])]
)
AC_CHECK_LIB(crypt, crypt_r, [ true ],
  [AC_MSG_ERROR([Couldn't find crypt library. Try installing glibc-devel])]
)
COCKPIT_WS_LIBS="$COCKPIT_WS_LIBS -lcrypt"

# pcp
AC_MSG_CHECKING([whether to build with PCP])
AC_ARG_ENABLE(pcp, AC_HELP_STRING([--disable-pcp], [Disable usage of PCP]))

if test "$enable_pcp" = "no"; then
  AC_MSG_RESULT($enable_pcp)

else
  if test "$enable_pcp" = ""; then
    disable_msg="(perhaps --disable-pcp)"
  fi

  enable_pcp="yes"
  AC_MSG_RESULT($enable_pcp)

  AC_CHECK_HEADER([pcp/pmapi.h], ,
    [AC_MSG_ERROR([Couldn't find pcp headers $disable_msg])]
  )
  AC_CHECK_HEADERS([pcp/import.h pcp/pmda.h], ,
    [AC_MSG_ERROR([Couldn't find pcp headers $disable_msg])],
    [#include <pcp/pmapi.h>]
  )
  AC_CHECK_LIB(pcp, pmNewContext, [ true ],
    [AC_MSG_ERROR([Couldn't find pcp library $disable_msg])]
  )
  AC_CHECK_LIB(pcp_pmda, pmdaCacheLookup, [ true ],
    [AC_MSG_ERROR([Couldn't find pcp_pmda library $disable_msg])]
  )
  AC_CHECK_LIB(pcp_import, pmiStart, [ true ],
    [AC_MSG_ERROR([Couldn't find pcp_import library $disable_msg])]
  )
  COCKPIT_PCP_LIBS="$COCKPIT_PCP_LIBS -lpcp -lm"
fi

AM_CONDITIONAL([ENABLE_PCP], [test "$enable_pcp" = "yes"])

# systemd
AC_ARG_WITH([systemdunitdir], [AC_HELP_STRING([--with-systemdunitdir=DIR],
                                              [directory to install systemd unit files in])])

if test ! -z "$with_systemdunitdir"; then
  systemdunitdir=$with_systemdunitdir
elif test "$enable_prefix_only" = "yes"; then
    systemdunitdir='${prefix}/lib/systemd/system'
else
  PKG_CHECK_MODULES(SYSTEMD, [systemd])
  AC_MSG_CHECKING(for systemd unit dir)
  systemdunitdir=$($PKG_CONFIG systemd --variable=systemdsystemunitdir)
  if test "$systemdunitdir" = ""; then
    AC_MSG_ERROR([systemd's pkg-config file doesn't contain 'systemdsystemunitdir' variable])
  fi
  AC_MSG_RESULT($systemdunitdir)
fi
AC_SUBST([systemdunitdir], [$systemdunitdir])

# Internationalization

GETTEXT_PACKAGE=cockpit
AC_SUBST([GETTEXT_PACKAGE])
AM_GLIB_GNU_GETTEXT
AC_DEFINE_UNQUOTED([GETTEXT_PACKAGE],["$GETTEXT_PACKAGE"],[gettext domain])

AC_PATH_PROG(MSGCAT, msgcat)
AC_PATH_PROG(MSGFMT, msgfmt)
AC_PATH_PROG(MSGGREP, msggrep)
AC_PATH_PROG(MSGMERGE, msgmerge)
AC_PATH_PROG(XGETTEXT, xgettext)
AC_MSG_CHECKING(for gettext binaries)
if test -x "$MSGCAT" -a -x "$MSGFMT" -a -x "$MSGMERGE" -a -x "$MSGGREP" -a -x "$XGETTEXT"; then
  AC_SUBST(MSGCAT)
  AC_SUBST(MSGFMT)
  AC_SUBST(MSGGREP)
  AC_SUBST(MSGMERGE)
  AC_SUBST(XGETTEXT)
  AC_MSG_RESULT(yes)
else
  AC_MSG_ERROR([no. Please install gettext tools])
fi

# usermod
#
# usermod might not be world-executable, so AC_PATH_PROG might not
# find it.  In that case, we fall back to /usr/sbin/usermod.

AC_PATH_PROG([USERMOD], [usermod], [/usr/sbin/usermod], [$PATH:/usr/local/sbin:/usr/sbin:/sbin])
AC_DEFINE_UNQUOTED([PATH_USERMOD],["$USERMOD"],[Location of usermod binary])

# newusers
AC_PATH_PROG([NEWUSERS], [newusers], [/usr/sbin/newusers], [$PATH:/usr/local/sbin:/usr/sbin:/sbin])
AC_DEFINE_UNQUOTED([PATH_NEWUSERS],["$NEWUSERS"],[Location of newusers binary])
AC_MSG_CHECKING(whether $NEWUSERS understands --crypt-method)
if newusers --help | grep -q -e --crypt-method; then
  AC_DEFINE_UNQUOTED([HAVE_NEWUSERS_CRYPT_METHOD],[1],[whether newusers understands --crypt-method])
  AC_MSG_RESULT(yes)
else
  AC_MSG_RESULT(no)
fi

# chpasswd
AC_PATH_PROG([CHPASSWD], [chpasswd], [/usr/sbin/chpasswd], [$PATH:/usr/local/sbin:/usr/sbin:/sbin])
AC_DEFINE_UNQUOTED([PATH_CHPASSWD],["$CHPASSWD"],[Location of chpasswd binary])

# ssh-add
AC_PATH_PROG([SSH_ADD], [ssh-add], [/usr/bin/ssh-add], [$PATH:/usr/local/sbin:/usr/sbin:/sbin])
AC_DEFINE_UNQUOTED([PATH_SSH_ADD], ["$SSH_ADD"], [Location of ssh-add binary])

# ssh-agent
AC_PATH_PROG([SSH_AGENT], [ssh-agent], [/usr/bin/ssh-agent], [$PATH:/usr/local/bin:/usr/bin:/bin])
AC_DEFINE_UNQUOTED([PATH_SSH_AGENT], ["$SSH_AGENT"], [Location of ssh-agent binary])

# chcon
AC_PATH_PROG([CHCON], [chcon], [/usr/bin/chcon], [$PATH:/usr/local/bin:/usr/bin:/bin])
AC_DEFINE_UNQUOTED([PATH_CHCON], ["$CHCON"], [Location of chcon binary])

changequote(,)dnl
if test "x$GCC" = "xyes"; then
  CFLAGS="-Wall \
          -Werror=strict-prototypes -Werror=missing-prototypes \
          -Werror=implicit-function-declaration \
          -Werror=pointer-arith -Werror=init-self \
          -Werror=format=2 \
          -Werror=return-type \
          -Werror=missing-include-dirs $CFLAGS"
fi
changequote([,])dnl

# System functions

AC_CHECK_FUNCS(fdwalk)

# Address sanitizer

AC_MSG_CHECKING([for asan flags])
AC_ARG_ENABLE(asan,
              AC_HELP_STRING([--enable-asan=no/yes],
                             [Turn the Address Sanitizer on or off])
             )

if test "$enable_asan" = "yes"; then
    CFLAGS="$CFLAGS -fsanitize=address -O1 -fno-omit-frame-pointer -g"
    asan_status="yes"
else
    asan_status="no"
fi
AM_CONDITIONAL(WITH_ASAN, test "$enable_asan" = "yes")
AC_MSG_RESULT($asan_status)

# User and group for running cockpit web server (cockpit-tls or -ws in customized setups)

AC_ARG_WITH(cockpit_user,
	    AS_HELP_STRING([--with-cockpit-user=<user>],
			   [User for running cockpit (root)]
                          )
           )
AC_ARG_WITH(cockpit_group,
	    AS_HELP_STRING([--with-cockpit-group=<group>],
			   [Group for running cockpit]
                          )
           )
if test -z "$with_cockpit_user"; then
    COCKPIT_USER=root
    COCKPIT_GROUP=
else
    COCKPIT_USER=$with_cockpit_user
    if test -z "$with_cockpit_group"; then
        COCKPIT_GROUP=$with_cockpit_user
    else
	COCKPIT_GROUP=$with_cockpit_group
    fi
fi

AC_SUBST(COCKPIT_USER)
AC_SUBST(COCKPIT_GROUP)

# User for running cockpit-ws instances from cockpit-tls

AC_ARG_WITH(cockpit_ws_instance_user,
	    AS_HELP_STRING([--with-cockpit-ws-instance-user=<user>],
			   [User for running cockpit-ws instances from cockpit-tls (root)]
                          )
           )
AC_ARG_WITH(cockpit_ws_instance_group,
	    AS_HELP_STRING([--with-cockpit-ws-instance-group=<group>],
			   [Group for running cockpit-ws instances from cockpit-tls]
                          )
           )
if test -z "$with_cockpit_ws_instance_user"; then
    if test "$COCKPIT_USER" != "root"; then
        AC_MSG_ERROR([--with-cockpit-ws-instance-user is required when setting --with-cockpit-user])
    fi
    COCKPIT_WSINSTANCE_USER=root
else
    COCKPIT_WSINSTANCE_USER=$with_cockpit_ws_instance_user
    if test -z "$with_cockpit_ws_instance_group"; then
        COCKPIT_WSINSTANCE_GROUP=$with_cockpit_ws_instance_user
    else
        COCKPIT_WSINSTANCE_GROUP=$with_cockpit_ws_instance_group
    fi
fi

AC_SUBST(COCKPIT_WSINSTANCE_USER)
AC_SUBST(COCKPIT_WSINSTANCE_GROUP)

# admin users group
AC_ARG_WITH([admin-group],
            [AC_HELP_STRING([--with-admin-group=GROUP],
                            [system group to which admin users belong])],
            [admin_group=$withval],
            [
              AC_MSG_CHECKING([for system group to which admin users belong])
              CANDIDATE_GROUPS="wheel sudo root"
              admin_group="$(getent group ${CANDIDATE_GROUPS} | head -n1 | cut -f1 -d:)"
              if test -n "$admin_group"; then
                AC_MSG_RESULT([$admin_group])
              else
                AC_MSG_RESULT([unable to detect])
                AC_MSG_ERROR([none of '${CANDIDATE_GROUPS}' exist: please specify a group with --with-admin-group=])
              fi
            ])
AC_SUBST(admin_group)


AC_ARG_WITH(selinux_config_type,
	    AS_HELP_STRING([--with-selinux-config-type=<type>],
			   [SELinux context type for cockpit config files]
                          )
           )

COCKPIT_SELINUX_CONFIG_TYPE=$with_selinux_config_type
AC_SUBST(COCKPIT_SELINUX_CONFIG_TYPE)

# Documentation

AC_MSG_CHECKING([whether to build documentation])
AC_ARG_ENABLE(doc,
              AC_HELP_STRING([--disable-doc],
                             [Disable building documentation])
             )

if test "$enable_doc" = "no"; then
        AC_MSG_RESULT($enable_doc)

else
        if test "$enable_doc" = ""; then
                disable_msg="(perhaps --disable-doc)"
        fi

        enable_doc="yes"
        AC_MSG_RESULT($enable_doc)

        AC_PATH_PROG([XSLTPROC], [xsltproc], [no])
        if test "$XSLTPROC" = "no"; then
                AC_MSG_ERROR([the xsltproc command was not found $disable_msg])
        fi

        AC_PATH_PROG([XMLTO], [xmlto], [no])
        if test "$XMLTO" = "no"; then
                AC_MSG_ERROR([the xmlto command was not found $disable_msg])
        fi

        AC_SUBST(XSLTPROC)
        AC_SUBST(XMLTO)
fi

AM_CONDITIONAL([ENABLE_DOC], [test "$enable_doc" = "yes"])

# Debug

AC_MSG_CHECKING([for debug mode])
AC_ARG_ENABLE(debug,
              AC_HELP_STRING([--enable-debug=no/default/yes],
                             [Turn on or off debugging])
             )

if test "$enable_debug" != "no"; then
  AC_DEFINE_UNQUOTED(WITH_DEBUG, 1, [Print debug output])
  AC_DEFINE_UNQUOTED(_DEBUG, 1, [In debug mode])
  CFLAGS="$CFLAGS -g"
fi
debugdir='${prefix}/src/debug'
if test "$enable_debug" = "yes"; then
  debug_status="yes"
  debugdir=
  CFLAGS="$CFLAGS -O0"
  NODE_ENV="development"
elif test "$enable_debug" = "no"; then
  debug_status="no"
  CFLAGS="$CFLAGS -O2"
  NODE_ENV="production"
else
  debug_status="default"
  NODE_ENV="${NODE_ENV:-production}"
fi
AM_CONDITIONAL(WITH_DEBUG, test "$enable_debug" = "yes")
AC_MSG_RESULT($debug_status)
AC_SUBST(NODE_ENV)
AC_SUBST(debugdir)

# Coverage

AC_MSG_CHECKING([whether to build with coverage])
AC_ARG_ENABLE([coverage],
              [AS_HELP_STRING([--enable-coverage], [Whether to enable coverage testing])],
              [],
              [enable_coverage=no])

if test "$enable_coverage" = "yes"; then
  if test "$GCC" != "yes"; then
    AC_MSG_ERROR(Coverage testing requires GCC)
  fi

  CFLAGS="$CFLAGS -O0 -g --coverage"
  LDFLAGS="$LDFLAGS --coverage"
fi

AM_CONDITIONAL([WITH_COVERAGE], [test "$enable_coverage" = "yes"])
AC_MSG_RESULT([$enable_coverage])

# Strict

AC_ARG_ENABLE(strict, [
               AS_HELP_STRING([--enable-strict], [Strict code compilation])
             ])

AC_MSG_CHECKING([build strict])
if test "$enable_strict" = "yes"; then
  CFLAGS="$CFLAGS -Werror"
else
  enable_strict="no"
fi
AC_MSG_RESULT($enable_strict)

# Python version; prefer python3
m4_define_default([_AM_PYTHON_INTERPRETER_LIST], [python3 python2 python])
AM_PATH_PYTHON([2.7])

# Generate
#

AC_SUBST(PAM_LIBS)

AC_OUTPUT([
Makefile
doc/guide/version
src/tls/cockpit-certificate-helper
])


dnl ==========================================================================
echo "
               Cockpit $VERSION
             ================

        prefix:                     ${prefix}
        exec_prefix:                ${exec_prefix}
        libdir:                     ${libdir}
        libexecdir:                 ${libexecdir}
        bindir:                     ${bindir}
        sbindir:                    ${sbindir}
        datarootdir:                ${datarootdir}
        datadir:                    ${datadir}
        sysconfdir:                 ${sysconfdir}
        localstatedir:              ${localstatedir}
        pamdir:                     ${pamdir}
        systemd unit dir:           ${systemdunitdir}

        compiler:                   ${CC}
        cflags:                     ${CFLAGS}
        cppflags:                   ${CPPFLAGS}

        cockpit-ws user:            ${COCKPIT_USER}
        cockpit-ws group:           ${COCKPIT_GROUP}
        cockpit-ws instance user:   ${COCKPIT_WSINSTANCE_USER}
        cockpit-ws instance group:  ${COCKPIT_WSINSTANCE_GROUP}
        admin group:                ${admin_group}
        selinux config type:        ${COCKPIT_SELINUX_CONFIG_TYPE}

        Maintainer mode:            ${USE_MAINTAINER_MODE}
        Building docs:              ${enable_doc}
        Debug mode:                 ${debug_status}
        Node environment:           ${NODE_ENV}
        With coverage:              ${enable_coverage}
        With address sanitizer:     ${asan_status}
        With PCP:                   ${enable_pcp}
        With polkit:                ${enable_polkit}
        Branding:                   ${BRAND}

        cockpit-ssh:                ${enable_ssh}
        Supports key auth:          ${key_auth}

        ssh-add:                    ${SSH_ADD}
        ssh-agent:                  ${SSH_AGENT}
        usermod:                    ${USERMOD}
        chcon:                      ${CHCON}
"
