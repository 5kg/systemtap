dnl configure.ac --- autoconf input file for systemtap
dnl Process this file with autoconf to produce a configure script.

AC_INIT([systemtap], 2.4, systemtap@sourceware.org, systemtap)
dnl                  ^^^ see also NEWS, systemtap.spec, testsuite/configure.ac
dnl                      doc/SystemTap_Beginners_Guide/en-US/Book_Info.xml

dnl Get the target arch for libHelperSDT.so
AC_CANONICAL_TARGET

AC_PREREQ(2.63)
dnl We don't maintain a ChangeLog, which makes us non-GNU -> foreign.
AM_INIT_AUTOMAKE([no-dist foreign])
AM_MAINTAINER_MODE

m4_ifdef([AM_SILENT_RULES],[AM_SILENT_RULES([yes])])

AC_USE_SYSTEM_EXTENSIONS
AM_PROG_MKDIR_P
AC_SUBST(MKDIR_P)
AC_PROG_LN_S
AC_PROG_CC
AC_PROG_CXX
AC_PROG_CPP
AM_PROG_CC_STDC
AM_PROG_CC_C_O
AC_PROG_RANLIB
AC_OBJEXT
AC_EXEEXT
AC_PROG_INSTALL
AC_PROG_MAKE_SET
AC_SUBST(CFLAGS)
AC_SUBST(CXXFLAGS)
AC_SYS_LARGEFILE
AM_GNU_GETTEXT(external)
AM_GNU_GETTEXT_VERSION(0.17)
AC_CHECK_FUNCS(ppoll)
AC_CHECK_FUNCS(openat)

dnl Handle the prologues option.
dnl
dnl If the user didn't specify --enable-prologues/--disable-prologues
dnl and the x86 system has a version of gcc less than version 4,
dnl automatically enable prologues.
if test "${enable_prologues+set}" != set; then
  AC_MSG_CHECKING([to see if prologue searching should be the default])
  if { echo '#if __i386__ == 1 && __GNUC__ < 4'
       echo ' yes '
       echo '#endif'; } | ${CC} -E - | grep yes > /dev/null; then
    enable_prologues=yes
    AC_MSG_RESULT([yes])
  else
    AC_MSG_RESULT([no])
  fi
fi
AC_ARG_ENABLE([prologues],
  AS_HELP_STRING([--enable-prologues], [make -P prologue-searching default]),
  [
if test "$enable_prologues" = yes; then
  AC_DEFINE([ENABLE_PROLOGUES],[],[make -P prologue-searching default])
fi])

dnl Handle the disable-sdt-probes option.
dnl
dnl Default to --disable-sdt-probes if --enable-sdt-probes/--disable-prologues
dnl was not specified and the gcc version is less than version 4,

if test "${enable_sdt_probes+set}" != set; then
  AC_MSG_CHECKING([to see if sdt probes should be the default])
  if { echo '#if __GNUC__ < 4'
       echo ' yes '
       echo '#endif'; } | ${CC} -E - | grep yes > /dev/null; then
    enable_sdt_probes=no
    AC_MSG_RESULT([no])
  else
    AC_MSG_RESULT([yes])
  fi
fi
AC_ARG_ENABLE([sdt-probes],
  [AS_HELP_STRING([--disable-sdt-probes],
                  [disable process.mark probes in stap, staprun, stapio])])
AS_IF([test "x$enable_sdt_probes" != xno], [
  AC_DEFINE([ENABLE_SDT_PROBES], [1],
            [Define to 1 to enable process.mark probes in stap, staprun, stapio.])
])

AC_ARG_ENABLE([ssp],
  [AS_HELP_STRING([--disable-ssp], [disable gcc stack-protector])])
AS_IF([test "x$enable_ssp" != xno],[
   save_CFLAGS="$CFLAGS"
   save_CXXFLAGS="$CXXFLAGS"
   CXXFLAGS="$CXXFLAGS -Werror -fstack-protector-all -D_FORTIFY_SOURCE=2"
   CFLAGS="$CFLAGS -Werror -fstack-protector-all -D_FORTIFY_SOURCE=2"
   AC_COMPILE_IFELSE([AC_LANG_SOURCE([int something ();])], [
       AC_MSG_NOTICE([Compiling with gcc -fstack-protector-all et al.])
	CFLAGS="$save_CFLAGS -fstack-protector-all -D_FORTIFY_SOURCE=2"
	CXXFLAGS="$save_CXXFLAGS -fstack-protector-all -D_FORTIFY_SOURCE=2"],[
       AC_MSG_NOTICE([Compiler does not support -fstack-protector-all et al.])
       CFLAGS="$save_CFLAGS"
       CXXFLAGS="$save_CXXFLAGS"])])


dnl Link with gold if possible
dnl but: https://bugzilla.redhat.com/show_bug.cgi?id=636603
dnl
dnl AC_PATH_PROG(GOLD, [ld.gold], [no])
dnl if test "x$GOLD" != "xno"
dnl then
dnl   mkdir -p Bdir
dnl   ln -sf $GOLD Bdir/ld
dnl   LDFLAGS="$LDFLAGS -B`pwd`/Bdir/"
dnl   AC_MSG_NOTICE([using ld.gold to link])
dnl fi


# Compiling without fPIE by default (see PR 9922)
AC_ARG_ENABLE([pie],
  [AS_HELP_STRING([--enable-pie], [enable position-independent-executable])])
AS_IF([test "x$enable_pie" == xyes],[
   PIECFLAGS='-fPIE'
   PIECXXFLAGS='-fPIE'
   PIELDFLAGS='-pie -Wl,-z,relro -Wl,-z,now'
   save_CFLAGS="$CFLAGS"
   save_CXXFLAGS="$CXXFLAGS"
   save_LDFLAGS="$LDFLAGS"
   CFLAGS="$CFLAGS $PIECFLAGS"
   CXXFLAGS="$CXXFLAGS $PIECXXFLAGS"
   LDFLAGS="$LDFLAGS $PIELDFLAGS"
   AC_LINK_IFELSE([AC_LANG_SOURCE([void main () {}])], [
       AC_MSG_NOTICE([Compiling with gcc pie et al.])
   ], [
       AC_MSG_NOTICE([Compiler does not support -pie et al.])
       PIECFLAGS=""
       PIECXXFLAGS=""
       PIELDFLAGS=""
   ])
   CFLAGS="$save_CFLAGS"
   CXXFLAGS="$save_CXXFLAGS"
   LDFLAGS="$save_LDFLAGS"
])
AC_SUBST(PIELDFLAGS)
AC_SUBST(PIECFLAGS)
AC_SUBST(PIECXXFLAGS)

dnl Handle optional sqlite support.  If enabled/disabled by the user,
dnl do the right thing.  If not specified by the user, use it if
dnl present.
AC_ARG_ENABLE([sqlite],
  AS_HELP_STRING([--enable-sqlite], [build with sqlite support]),
  [],					dnl ACTION-IF-GIVEN
  [enable_sqlite=check])		dnl ACTION-IF-NOT-GIVEN
sqlite3_LIBS=
AS_IF([test "x$enable_sqlite" != xno],
  [AC_CHECK_LIB([sqlite3], [sqlite3_open],
    [AC_SUBST([sqlite3_LIBS], [-lsqlite3])
     AC_DEFINE([HAVE_LIBSQLITE3], [1], [Define to 1 if you have the 'sqlite3' library (-lsqlite3).])],
    [if test "x$enable_sqlite" != xcheck; then
      AC_MSG_FAILURE([--enable-sqlite was given, but test for sqlite failed])
     fi])])

dnl Handle the option to only build runtime
AC_ARG_ENABLE([translator],
  AS_HELP_STRING([--disable-translator], [build only runtime utilities]),
  [],
  [enable_translator="yes"])
AM_CONDITIONAL([BUILD_TRANSLATOR], [test "$enable_translator" == "yes"])

dnl Handle the option to build the crash extension
AC_ARG_ENABLE([crash],
  AS_HELP_STRING([--enable-crash@<:@=DIRECTORY@:>@],
    [enable crash extension (default is disabled).  Optional DIRECTORY
    is the path to the crash header file (needed if installed in a
    non-standard location).]),
  [if test "$enable_crash" != "no"; then
    dnl Handle custom install dir (if needed)
    save_CPPFLAGS="$CPPFLAGS"
    if test "$enable_crash" != "yes"; then
      staplog_CPPFLAGS=-I$enable_crash
      CPPFLAGS="${staplog_CPPFLAGS} $CPPFLAGS"
      AC_SUBST([staplog_CPPFLAGS])
    fi
    AC_CHECK_HEADERS([crash/defs.h], [],
     [AC_MSG_ERROR([cannot find required crash header (crash-devel may need to be installed)])],
     [
#define NR_CPUS 256
     ])
    CPPFLAGS="$save_CPPFLAGS"
  fi],
  [enable_crash="no"])
AM_CONDITIONAL([BUILD_CRASHMOD], [test "$enable_crash" != "no"])

dnl Handle the option to build the documentation
building_docs="no"
AC_ARG_ENABLE([docs],
  AS_HELP_STRING([--enable-docs],
    [enable building documentation (default on if latex etc. found).]),
  [enable_docs=$enableval],
  [enable_docs="check"])
AC_CHECK_PROG(have_xvfb, xvfb-run, yes, no)
AM_CONDITIONAL([HAVE_XVFB], [test "$have_xvfb" == "yes"])
AC_CHECK_PROG(have_latex, latex, yes, no)
AC_CHECK_PROG(have_dvips, dvips, yes, no)
AC_CHECK_PROG(have_ps2pdf, ps2pdf, yes, no)
AC_CHECK_PROG(have_latex2html, latex2html, yes, no)
if test "x${have_latex}${have_dvips}${have_ps2pdf}${have_latex2html}" != "xyesyesyesyes"; then
   if test "$enable_docs" == "yes"; then
      AC_MSG_ERROR([cannot find all tools for building documentation])
   fi
  if test "$enable_docs" == "check"; then
    AC_MSG_WARN([will not build documentation, cannot find all tools])
  fi
fi
if test "x${have_latex}${have_dvips}${have_ps2pdf}${have_latex2html}" == "xyesyesyesyes" -a "$enable_docs" != "no"; then
   building_docs="yes"
fi
AM_CONDITIONAL([BUILD_DOCS], [test "$building_docs" == "yes"])

dnl Handle the options to build the publican options
building_publican="no"
AC_ARG_ENABLE([publican],
  AS_HELP_STRING([--enable-publican],
    [enable building publican documentation guides (default on if publican found and other documentation built).]),
  [enable_publican=$enableval],
  [enable_publican="check"])
if test "$building_docs" == "no" -a "$enable_publican" == "yes" ; then
   AC_MSG_ERROR([must use --enable-docs with --enable-publican])
fi
AC_CHECK_PROG(have_publican, publican, yes, no)
if test "$enable_publican" == "yes"; then
   if test "x${have_publican}" != "xyes"; then
      AC_MSG_ERROR([cannot find publican for building publican guides])
   fi
fi
if test "x${have_publican}" == "xyes" -a "$enable_publican" != "no" -a "${building_docs}" == "yes"; then
   building_publican="yes"
fi
AM_CONDITIONAL([BUILD_PUBLICAN], [test "$building_publican" == "yes"])
publican_brand="common"
AC_ARG_WITH([publican-brand],
  AS_HELP_STRING([--with-publican-brand=BRAND],
    [building publican documentation guides using given brand (defaults to "common").]),
  [publican_brand=$withval],
  [publican_brand="common"])
PUBLICAN_BRAND=$publican_brand
AC_SUBST(PUBLICAN_BRAND)

dnl Handle the option to build the reference documentation
building_refdocs="no"
AC_ARG_ENABLE([refdocs],
  AS_HELP_STRING([--enable-refdocs],
    [enable building reference documentation (default on if xmlto etc. found and other documentation built).]),
  [enable_refdocs=$enableval],
  [enable_refdocs="check"])
if test "$building_docs" == "no" -a "$enable_refdocs" == "yes" ; then
   AC_MSG_ERROR([must use --enable-docs with --enable-refdocs])
fi
AC_CHECK_PROG(have_xmlto, xmlto, yes, no)
if test "$enable_refdocs" == "yes"; then
   if test "x${have_xmlto}" != "xyes"; then
      AC_MSG_ERROR([cannot find xmlto for building reference documentation])
   fi
fi
if test "x${have_xmlto}" == "xyes" -a "$enable_refdocs" != "no" -a "${building_docs}" == "yes"; then
   building_refdocs="yes"
fi
AM_CONDITIONAL([BUILD_REFDOCS], [test "$building_refdocs" == "yes"])

AC_CHECK_PROG(have_fop, fop, yes, no)
if test "x${have_fop}" == "xyes"; then
   # Due to rhbz505364 / 830266, we must actually test-run fop, not just
   # hope that it works.
   AC_MSG_CHECKING([to see if xmlto --with-fop actually works])
   if xmlto --with-fop pdf ${srcdir}/doc/SystemTap_Tapset_Reference/dummy-tapsets.xml >/dev/null 2>&1; then
      AC_MSG_RESULT([yes])
   else
      AC_MSG_RESULT([it's dead, Jim])
      have_fop="broken"
   fi
   rm -f dummy-tapsets.pdf
fi
AM_CONDITIONAL([HAVE_FOP], [test "$have_fop" == "yes"])

dnl There is a strange bug in older versions of xmlto when generating pdf.
dnl https://bugzilla.redhat.com/show_bug.cgi?id=526273
dnl So make sure to have a chapter title starting with L plus an refentry.
dnl This will make sure the xmlto pdf support test fails on buggy versions.
cat > conftest.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE book PUBLIC "-//OASIS//DTD DocBook XML V4.1.2//EN"
"http://www.oasis-open.org/docbook/xml/4.1.2/docbookx.dtd" []>
   <book lang="en">
   <bookinfo><title>foo</title></bookinfo>
   <chapter id="logging.stp"><title>Logging Tapset</title>
   <refentry id="API-log"><refnamediv><refname>log</refname>
   <refpurpose>logging</refpurpose></refnamediv>
   <refsect1><title>Description</title>
   <para>baz</para></refsect1></refentry></chapter>
   </book>
EOF

if test "x${have_xmlto}" == "xyes"; then
AC_MSG_CHECKING([for xmlto --stringparam support])
   xmlto --stringparam man.authors.section.enabled=0 html-nochunks conftest.xml >/dev/null 2>&1
   if test $? == 0; then
      have_xmlto_stringparam="yes"
      AC_MSG_RESULT([yes])
   else
      AC_MSG_RESULT([no])
   fi
   rm -f conftest.html
fi
AM_CONDITIONAL([XMLTO_STRINGPARAM], [test "$have_xmlto_stringparam" == "yes"])

if test "x${building_refdocs}" == "xyes"; then
AC_MSG_CHECKING([for xmlto pdf support])
   # xmlto pdf may work *only* with --with-fop
   # XXX: To an extent this test is redundant in that the --with-fop test
   # already ran xmlto --with-fop pdf ....
   if test "x$have_fop" == "xyes"; then
      XMLTOPDF_FOP=--with-fop
   else
      XMLTOPDF_FOP=
   fi
   xmlto $XMLTOPDF_FOP pdf conftest.xml >& /dev/null
   if test $? == 0; then
      have_xmlto_pdf="yes"
      AC_MSG_RESULT([yes])
   else
      AC_MSG_RESULT([no])
      AC_MSG_WARN([Not building reference documentation in PDF format])
   fi
   rm -f conftest.pdf
fi
AM_CONDITIONAL([BUILD_PDFREFDOCS], [test "$have_xmlto_pdf" == "yes"])

rm -f conftest.xml

dnl See if we have the nss/nspr headers and libraries
AC_ARG_WITH([nss],
  AS_HELP_STRING([--without-nss],
    [Do not use NSS even if present]))

AS_IF([test "x$with_nss" != "xno"], [
  PKG_CHECK_MODULES([nss], [nss >= 3],
    [have_nss=yes
     AC_DEFINE([HAVE_NSS], [1], [Define to 1 if you have the nss libraries.])
    ], [have_nss=no])
], [have_nss=no])

AM_CONDITIONAL([HAVE_NSS], [test "${have_nss}" = "yes"])

dnl Handle the option to build the compile server.
AC_ARG_ENABLE([server],
  AS_HELP_STRING([--enable-server],
    [enable building of stap-server (default on if nss etc. found).]),
  [enable_server=$enableval],
  [enable_server="check"])

if test "$enable_server" != "no"; then
dnl See if we have enough libraries and tools to build the compile server
  if test "x${have_nss}" != "xyes"; then
    AC_MSG_WARN([will not build systemtap compile server, cannot find nss headers])
  fi		       
fi
AM_CONDITIONAL([BUILD_SERVER], [test "${have_nss}" == "yes" -a "$enable_server" != "no"])

if test "${have_nss}" != "yes"; then
  AC_MSG_WARN([compile-server client functionality will be disabled, cannot find nss development files])
fi		       

dnl See if we have the avahi libraries and headers
AC_ARG_WITH([avahi],
  AS_HELP_STRING([--without-avahi],
    [Do not use Avahi even if present]))

AS_IF([test "x$with_avahi" != "xno"], [
  PKG_CHECK_MODULES([avahi], [avahi-client],
    [have_avahi=yes
     AC_DEFINE([HAVE_AVAHI], [1], [Define to 1 if you have the avahi libraries.])
    ], [have_avahi=no])
], [have_avahi=no])

AM_CONDITIONAL([HAVE_AVAHI], [test "${have_avahi}" = "yes"])

if test "${have_avahi}" != "yes"; then
  AC_MSG_WARN([some compile-server functionality will be restricted, cannot find avahi development files])
fi


dnl Look for librpm.
AC_ARG_WITH([rpm],
  [AS_HELP_STRING([--with-rpm],
                  [query rpm database for missing debuginfos])], [], [with_rpm="auto"])
if test "$with_rpm" != "no"; then
  AC_CHECK_LIB(rpm, rpmtsInitIterator, [
                    AC_DEFINE([HAVE_LIBRPM],[1],[have librpm])
                    stap_LIBS="$stap_LIBS -lc -lrpm"
                    have_librpm="yes"], [have_librpm="no"])
  AC_CHECK_LIB(rpmio, rpmFreeCrypto, [
                    AC_DEFINE([HAVE_LIBRPMIO],[1],[have librpmio])
                    stap_LIBS="$stap_LIBS -lc -lrpmio"
                    have_librpmio="yes"], [have_librpmio="no"])
  if test "x$have_librpm" != "xyes" -a "$with_rpm" == "yes"; then
     AC_MSG_ERROR([cannot find librpm])
  fi
  if test "x$have_librpmio" != "xyes" -a "$with_rpm" == "yes"; then
     AC_MSG_WARN([cannot find librpmio])
  fi
fi


dnl Handle elfutils.  If '--with-elfutils=DIR' wasn't specified, used
dnl the system's elfutils.
build_elfutils=no
AC_ARG_WITH([elfutils],
  AS_HELP_STRING([--with-elfutils=DIRECTORY],
    [find elfutils source code in DIRECTORY]),
  [
case "$with_elfutils" in
yes) AC_MSG_ERROR([--with-elfutils requires an argument]) ;;
''|no) ;;
*) build_elfutils=yes ;;
esac])
AM_CONDITIONAL(BUILD_ELFUTILS, test $build_elfutils = yes)
AC_SUBST(elfutils_abs_srcdir, `AS_IF([test $build_elfutils = yes],
				     [cd $with_elfutils && pwd])`)

if test $enable_translator = yes; then
  if test $build_elfutils = no; then
    # Need libdwfl-capable recent elfutils http://elfutils.fedorahosted.org/

    # On modern debian/ubuntu, libebl has been merged into libdw
    # http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=457543
    save_LIBS="$LIBS"
    AC_CHECK_LIB(ebl, ebl_get_elfmachine,[ebl_LIBS=-lebl],[ebl_LIBS=])
    LIBS="$save_LIBS"

    save_LIBS="$LIBS"
    AC_CHECK_LIB(dw, dwfl_module_getsym,[],[
      AC_MSG_ERROR([missing elfutils development headers/libraries (install elfutils-devel, libebl-dev, libdw-dev and/or libebl-devel)])],
      [-Wl,--start-group -ldw $ebl_LIBS -Wl,--end-group -lelf])
    AC_CHECK_LIB(dw, dwarf_next_unit,[],[
      AC_MSG_ERROR([elfutils, libdw too old, need 0.148+])],
      [-Wl,--start-group -ldw $ebl_LIBS -Wl,--end-group -lelf])
    stap_LIBS="$stap_LIBS -Wl,--start-group -ldw $ebl_LIBS -Wl,--end-group -lelf"
    LIBS="$save_LIBS"
  else
    # We built our own and stap_LDFLAGS points at the install.
    stap_LIBS="$stap_LIBS -Wl,--start-group -ldw -lebl -Wl,--end-group -lelf"
  fi
fi

AC_SUBST(stap_LIBS)
AC_MSG_NOTICE([stap will link $stap_LIBS])

# staprun has more modest libelf needs
if test $build_elfutils = no; then
  AC_CHECK_HEADERS([libelf.h])

  save_LIBS="$LIBS"
  dnl this will only succeed with elfutils 0.142+
  AC_CHECK_LIB(elf,elf_getshdrstrndx,[
    AC_DEFINE([HAVE_ELF_GETSHDRSTRNDX],[1],[Define to 1 if libelf has elf_getshdrstrndx])
    staprun_LIBS="$staprun_LIBS -lelf"
  ])
  LIBS="$save_LIBS"
else
  # We built our own and staprun_LDFLAGS points at the install.
  staprun_LIBS="$staprun_LIBS -lelf"
fi

AC_SUBST(staprun_LIBS)
AC_MSG_NOTICE([staprun will link $staprun_LIBS])


dnl Plop in the build (configure) date
date=`date +%Y-%m-%d`
AC_DEFINE_UNQUOTED(DATE, "$date", [Configuration/build date])
AC_SUBST(DATE, "$date")

# Before PR4037, we used to arrange to pass CFLAGS+=-m64 for a staprun
# being compiled on 32-bit userspace but running against 64-bit kernels.
# This is no longer necessary.

# Use tr1/unordered_map if available
AC_LANG_PUSH(C++)
AC_CHECK_HEADERS([tr1/unordered_map])
AC_CHECK_HEADERS([tr1/memory])
AC_CHECK_HEADERS([boost/shared_ptr.hpp])
AC_LANG_POP(C++)


# Check for Dyninst headers and libraries
AC_ARG_WITH([dyninst],
  AS_HELP_STRING([--with-dyninst=DIRECTORY],
    [find dyninst headers/libraries in DIRECTORY]))

case "$with_dyninst" in
no) ;;
''|yes) # Try a simple-minded distro search
     DYNINST_CXXFLAGS="-I/usr/include/dyninst"
     DYNINST_LDFLAGS="-L/usr/lib64/dyninst -L/usr/lib/dyninst" # XXX both lib paths!?!
     ;;
*)  # Use paths in the user-specified prefix
    DYNINST_CXXFLAGS="-I$with_dyninst/include"
    DYNINST_LDFLAGS="-L$with_dyninst/lib/" # XXX need lib64 too?
    ;;
esac
if test "$with_dyninst" != "no"; then
  AC_LANG_PUSH(C++)
  old_CPPFLAGS="$CPPFLAGS"
  CPPFLAGS="$CPPFLAGS $DYNINST_CXXFLAGS"
  AC_MSG_NOTICE([checking dyninst support])
  AC_CHECK_HEADERS([BPatch_object.h], [
      AC_SUBST(DYNINST_CXXFLAGS)
      AC_SUBST(DYNINST_LDFLAGS)
      AC_DEFINE([HAVE_DYNINST],[1],[Define to 1 if Dyninst is enabled])
      have_dyninst=yes
      AC_MSG_NOTICE([dyninst support available])])
  if test -n "$with_dyninst" -a "$have_dyninst" != "yes"; then
    AC_MSG_ERROR([Dyninst does not appear to be usable])
  fi
  CPPFLAGS="$old_CPPFLAGS"
  AC_LANG_POP(C++)
fi
AM_CONDITIONAL([HAVE_DYNINST], [test "${have_dyninst}" = "yes"])


AC_CACHE_CHECK([for assembler .section "?" flags support], stap_cv_sectionq, [
old_CFLAGS="$CFLAGS"
CFLAGS="$CFLAGS -Wa,--fatal-warnings"
AC_COMPILE_IFELSE([AC_LANG_SOURCE([asm(".section .note.foo,\"?\",\"note\"\n"
		       ".byte 1, 2, 3\n"
		       ".previous\n"
		       ".section .text,\"axG\",\"progbits\",foogroup,comdat\n"
		       ".byte 1\n"
		       ".pushsection .note.foo,\"?\",\"note\"\n"
		       ".byte 4, 5, 6\n"
		       ".popsection\n"
		       ".byte 2\n");])],
		  stap_cv_sectionq=yes, stap_cv_sectionq=no)
CFLAGS="$old_CFLAGS"])
AC_SUBST(support_section_question)
support_section_question=0
if test $stap_cv_sectionq = yes; then
  support_section_question=1
fi
AC_CONFIG_FILES([includes/sys/sdt-config.h po/Makefile.in])

if test $build_elfutils = yes -a $enable_translator = yes; then
  case "$with_elfutils" in
  /*) elfutils_srcdir="$with_elfutils" ;;
  *) elfutils_srcdir="../$with_elfutils" ;;
  esac
  AC_MSG_NOTICE([running ${elfutils_srcdir}/configure])

  save_CFLAGS="$CFLAGS"
  save_CXXFLAGS="$CXXFLAGS"
  CXXFLAGS="$CXXFLAGS -fgnu89-inline"
  CFLAGS="$CFLAGS -fgnu89-inline"
  gnu89_inline_flag=""
  AC_COMPILE_IFELSE([AC_LANG_SOURCE([int something ();])], [
    AC_MSG_NOTICE([Compiling elfutils with gcc -fgnu89-inline])
    gnu89_inline_flag="-fgnu89-inline"],[
    AC_MSG_NOTICE([Compiler does not support -fgnu89-inline])])
  CFLAGS="$save_CFLAGS"
  CXXFLAGS="$save_CXXFLAGS"

  # Our libdw.so's libebl will look in $ORIGIN/../lib/... but that
  # $ORIGIN is where libdw.so resides, which is not where there is a ../lib.
  # Note that $libdir might be using a quoted use of $exec_prefix or $prefix.
  # So we must make sure to pass those settings to elfutils configure.
  elfutils_rpath="-Wl,--enable-new-dtags,-rpath,${libdir}/${PACKAGE_NAME}"
  here=`pwd`
  # Check whether this is a source release, or a source checkout.
  # We need --enable-maintainer-mode if it isn't a source release.
  # elfutils.spec is only distributed in source releases.
  if test -f ${elfutils_srcdir}/elfutils.spec; then
	need_maintainer_option=""
  else
	need_maintainer_option="--enable-maintainer-mode"
  fi
  (mkdir -p build-elfutils && cd build-elfutils &&
   bash ${elfutils_srcdir}/configure --enable-libebl-subdir=${PACKAGE_NAME} \
				--includedir="${here}/include-elfutils" \
				--libdir="${here}/lib-elfutils" \
				--exec-prefix="$exec_prefix" \
				--prefix="$prefix" \
				--enable-dwz \
				${need_maintainer_option} \
				CFLAGS="${CFLAGS/-Wall/} $gnu89_inline_flag -fexceptions" \
				LDFLAGS="$LDFLAGS $elfutils_rpath" &&
   if test -f ${elfutils_srcdir}/config/version.h.in; then
        echo Found elfutils/version.h header...
   else
	echo elfutils/version.h header not found, need elfutils 0.142+...
	exit -1
   fi
  )
  if test $? != 0; then exit -1; fi
fi

dnl Optional libselinux support allows stapdyn to check
dnl for booleans that would prevent Dyninst from working.
AC_ARG_WITH([selinux],
  AS_HELP_STRING([--without-selinux],
    [Do not use libselinux even if present]))

AS_IF([test "x$with_selinux" != "xno"], [
  PKG_CHECK_MODULES([selinux], [libselinux],
    [have_selinux=yes
     AC_DEFINE([HAVE_SELINUX], [1], [Define to 1 if you have the SELinux libraries.])
    ], [have_selinux=no])
], [have_selinux=no])

AM_CONDITIONAL([HAVE_SELINUX], [test "${have_selinux}" = "yes"])

dnl Handle java+byteman support
AC_CHECK_PROG(have_javac, javac, yes, no)
AC_CHECK_PROG(have_jar, jar, yes, no)
if test "$have_javac" != no -a "$have_jar" != no; then
echo java found, will try to configure Byteman support

AC_ARG_WITH([java],
  [AS_HELP_STRING([--with-java=DIRECTORY],
     [Specify JDK directory to compile libHelperSDT.so against (default is /usr/lib/jvm/java)])],
  [],
  [with_java=/usr/lib/jvm/java])
AC_CHECK_FILE([$with_java],[],[with_java=no])
AC_SUBST(JAVADIR, "${with_java}") # always needed to compile

if test "$with_java" != "no"; then
AC_DEFINE_UNQUOTED(HAVE_JAVA, "1", [Flag indicating that libHelperSDT.so is available (can be found in PKGLIBDIR)])
fi

AM_CONDITIONAL([HAVE_JAVA], [test "$with_java" != "no"])

else
AC_MSG_WARN([will not run per-method java probing, missing byteman or java requirements])
AM_CONDITIONAL([HAVE_JAVA],false)
fi # java+byteman support

AC_SUBST(ENABLE_NLS, "$USE_NLS")
AC_SUBST(localedir, "$localedir")
AC_SUBST(LOCALEDIR, "$localedir")

AC_ARG_WITH([extra-version],
  AS_HELP_STRING([--with-extra-version=STRING],
    [Add STRING to stap -V version]))
AS_IF([test "x$with_extra_version" != "xno"], [
    stap_extra_version="$with_extra_version"
    AC_MSG_NOTICE([Adding extra version $stap_extra_version])], [
    stap_extra_version=""])
AC_DEFINE_UNQUOTED(STAP_EXTRA_VERSION, "$stap_extra_version", [extra stap version code])
AC_SUBST(STAP_EXTRA_VERSION, "$stap_extra_version")

dnl This is here mainly to make sure that configure --prefix=... changes
dnl the config.h files so files depending on it are recompiled
dnl prefix is passed through indirectly in the Makefile.am AM_CPPFLAGS.
dnl Formerly: Don't use this directly (when not given it is set to NONE).
dnl Currently: inline autoconf's later defaulting

stap_prefix=$prefix
test "$stap_prefix" = NONE && stap_prefix=$ac_default_prefix
AC_DEFINE_UNQUOTED(STAP_PREFIX, "$stap_prefix", [configure prefix location])
AC_SUBST(STAP_PREFIX, "$stap_prefix")

AC_CONFIG_HEADERS([config.h:config.in])
AC_CONFIG_FILES([Makefile doc/Makefile man/Makefile \
doc/beginners/Makefile doc/SystemTap_Tapset_Reference/Makefile \
man/stappaths.7 \
initscript/config.systemtap initscript/config.stap-server \
initscript/systemtap initscript/stap-server ])
AC_CONFIG_SUBDIRS(testsuite)
if test $enable_translator == "yes"; then
	AC_CONFIG_FILES([run-stap], [chmod +x run-stap])
fi
AC_CONFIG_FILES([dtrace], [chmod +x dtrace])
AC_CONFIG_FILES(stapdyn/Makefile)
AC_CONFIG_FILES(java/Makefile)
AC_CONFIG_FILES([java/stapbm], [chmod +x java/stapbm])
AC_CONFIG_FILES(staprun/Makefile)
AC_CONFIG_FILES([staprun/run-staprun], [chmod +x staprun/run-staprun])
AC_CONFIG_FILES(re2c-migrate/Makefile)

# Setup "shadow" directory doc/beginners that has everything setup for
# publican in one directory (through directory links if necessary).
# It would be nice to use AC_CONFIG_LINKS, but automake complains when
# the src is a directory and not a file.
AC_CONFIG_FILES([doc/beginners/publican.cfg:doc/SystemTap_Beginners_Guide/publican.cfg.in])
AC_CONFIG_COMMANDS([doc/beginners],
                   [rm -f $ac_abs_top_builddir/doc/beginners/en-US $ac_abs_top_builddir/doc/beginners/build/en-US/testsuite && mkdir -p $ac_abs_top_builddir/doc/beginners/build/en-US && ln -s $ac_abs_top_srcdir/doc/SystemTap_Beginners_Guide/en-US $ac_abs_top_builddir/doc/beginners/en-US && ln -s $ac_abs_top_srcdir/testsuite $ac_abs_top_builddir/doc/beginners/build/en-US/testsuite])

AC_OUTPUT

if test "${prefix}" = "/usr/local"; then
   AC_MSG_NOTICE([])
   AC_MSG_NOTICE([For a private or temporary build of systemtap, we recommend])
   AC_MSG_NOTICE([configuring with a prefix.  For example, try])
   AC_MSG_NOTICE([$0 $ac_configure_args --prefix=$HOME/systemtap-${PACKAGE_VERSION}-$$])
   AC_MSG_NOTICE([Running systemtap uninstalled, entirely out of the build tree,])
   AC_MSG_NOTICE([is not supported.])
fi
