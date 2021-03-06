AC_PREREQ[(2.64)]

m4_define(totem_version_major, 3)
m4_define(totem_version_minor, 20)
m4_define(totem_version_micro, 1)

# The version of the API exposed to plugins
m4_define(totem_api_version_major, 1)
m4_define(totem_api_version_minor, 0)

AC_INIT([totem],
        [totem_version_major.totem_version_minor.totem_version_micro],
        [http://bugzilla.gnome.org/enter_bug.cgi?product=totem],
        [totem],
        [https://wiki.gnome.org/Apps/Videos])

AM_MAINTAINER_MODE([enable])
AC_CONFIG_SRCDIR([src/totem.c])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_AUX_DIR([build-aux])

AM_INIT_AUTOMAKE([1.11 dist-xz tar-ustar no-dist-gzip check-news])

# Enable silent build when available (Automake 1.11)
m4_ifdef([AM_SILENT_RULES],[AM_SILENT_RULES([yes])])

dnl Add the languages which your application supports to po/LINGUAS
GETTEXT_PACKAGE=totem
AC_DEFINE_UNQUOTED(GETTEXT_PACKAGE, ["$GETTEXT_PACKAGE"], [Define to the Gettext package name])
AC_SUBST(GETTEXT_PACKAGE)
IT_PROG_INTLTOOL([0.50.1])

# User Documentation
YELP_HELP_INIT

GOBJECT_INTROSPECTION_CHECK([0.6.7])
m4_define([glib_required_version], [2.32.0])
AM_PATH_GLIB_2_0(glib_required_version)

AC_PROG_CXX
AM_PROG_CC_C_O

# Initialize libtool
LT_PREREQ([2.2])
LT_INIT
m4_ifdef([LT_OUTPUT], [LT_OUTPUT])

PKG_PROG_PKG_CONFIG

AC_PATH_PROG([GLIB_MKENUMS],[glib-mkenums])

# Requirements
GLIB_REQS=2.35.0
GIO_REQS=2.27.92
GTK_REQS=3.19.4
TOTEM_PLPARSER_REQS=3.10.1
VALA_REQS=0.14.1
PEAS_REQS=1.1.0
PYTHON_REQS=2.3
PYGOBJECT_REQS=2.90.3
GRILO_REQS=0.3.0
LIBXML_REQS=2.6.0
CAIRO_REQS=1.14.0

# Used in the pkg-config file for plugins
AC_SUBST(GLIB_REQS)
AC_SUBST(GTK_REQS)
AC_SUBST(TOTEM_PLPARSER_REQS)

TOTEM_VERSION_MAJOR=totem_version_major
TOTEM_VERSION_MINOR=totem_version_minor
TOTEM_VERSION_MICRO=totem_version_micro
AC_SUBST(TOTEM_VERSION_MAJOR)
AC_SUBST(TOTEM_VERSION_MINOR)
AC_SUBST(TOTEM_VERSION_MICRO)

TOTEM_API_VERSION=totem_api_version_major.totem_api_version_minor
AC_SUBST(TOTEM_API_VERSION)
AC_DEFINE_UNQUOTED(TOTEM_API_VERSION, ["$TOTEM_API_VERSION"], [Define to the Totem plugin API version])

# The full list of plugins
allowed_plugins="apple-trailers autoload-subtitles brasero-disc-recorder dbusservice im-status gromit lirc media-player-keys ontop opensubtitles properties pythonconsole recent rotation save-file samplepython sample-vala screensaver screenshot skipto zeitgeist-dp variable-rate vimeo"

PLUGINDIR='${libdir}/totem/plugins'
AC_SUBST(PLUGINDIR)

dnl the two versions here implicate the gstreamer core and gst-plugins
dnl release versions.
GST_MAJORMINOR=1.0
GST_REQS=1.6.0
GST_PLUG_BASE_REQS=1.6.0
GSTPLUG_REQS=0.11.93
CLUTTER_REQS=1.17.3
CLUTTER_GST_REQS=2.99.2
CLUTTER_GTK_REQS=1.7.1

dnl Check for the required GStreamer versions for missing plugins
dnl installation, unless this has been disabled.
MISSING_PLUGINS_MODULES=""
AC_MSG_CHECKING([whether to enable easy codec installation support])
AC_ARG_ENABLE([easy-codec-installation],
	[
		AS_HELP_STRING([--enable-easy-codec-installation],
			[Whether to enable easy codec installation support for GStreamer]
		)
	], [
		case "${enableval}" in
			yes) enable_easy_codec_installation=yes ;;
			no)  enable_easy_codec_installation=no ;;
			*) enable_easy_codec_installation=auto ;;
		esac
	], [
		dnl default value
		enable_easy_codec_installation=auto
	]
)
AC_MSG_RESULT([$enable_easy_codec_installation])

if test "x$enable_easy_codec_installation" != "xno"; then
	MISSING_PLUGINS_MODULES="gstreamer-pbutils-1.0"
	PKG_CHECK_MODULES(MISSING_PLUGINS, $MISSING_PLUGINS_MODULES,
	[
		AC_DEFINE([ENABLE_MISSING_PLUGIN_INSTALLATION], 1,
		          [Whether we can and want to do installation of missing plugins])
	])
fi

# ================================================================
# Backend requirements
# ================================================================

BACKEND_MODULES="gstreamer-1.0 >= $GST_REQS gstreamer-base-1.0 >= $GST_REQS gstreamer-plugins-base-1.0 >= $GST_PLUG_BASE_REQS $MISSING_PLUGINS_MODULES gstreamer-tag-1.0 >= $GSTPLUG_REQS gstreamer-audio-1.0 gstreamer-video-1.0 clutter-1.0 >= $CLUTTER_REQS clutter-gst-3.0 >= $CLUTTER_GST_REQS clutter-gtk-1.0 cairo >= $CAIRO_REQS gsettings-desktop-schemas"

PKG_CHECK_MODULES(BACKEND, $BACKEND_MODULES)
PKG_CHECK_MODULES(BACKEND_TEST, $BACKEND_MODULES gtk+-3.0 >= $GTK_REQS)

dnl Check for required plugins
gst10_toolsdir=`$PKG_CONFIG --variable=toolsdir gstreamer-1.0`
gst10_inspect="$gst10_toolsdir/gst-inspect-1.0"

dnl Give error and exit if we don't have the gst_inspect tool
AC_MSG_CHECKING([GStreamer 1.0 inspection tool])
if test -r "$gst10_inspect"; then
	AC_MSG_RESULT([yes])
else
	AC_MSG_RESULT([no])
	AC_MSG_ERROR([
		Cannot find required GStreamer-1.0 tool 'gst-inspect-1.0'.
		It should be part of gstreamer-1_0-utils. Please install it.
	])
fi

dnl Check for elements from gst-plugins-base
dnl Set plugins which contain below elements
set -- playback videoscale
for base_element in playbin videoscale
do
	AC_MSG_CHECKING([GStreamer 1.0 $base_element element])
	if $gst10_inspect $base_element >/dev/null 2>/dev/null; then
		AC_MSG_RESULT([yes])
	else
		AC_MSG_RESULT([no])
		AC_MSG_ERROR([
			Cannot find required GStreamer-1.0 plugin '$1'.
			It should be part of gst-plugins-base. Please install it.
		])

	fi
	shift;
done

dnl Check for elements from gst-plugins-good
dnl Set plugins which contain below elements
set -- autodetect goom scaletempo
for good_element in autoaudiosink goom scaletempo
do
	AC_MSG_CHECKING([GStreamer 1.0 $good_element element])
	if $gst10_inspect $good_element >/dev/null 2>/dev/null; then
		AC_MSG_RESULT([yes])
	else
		AC_MSG_RESULT([no])
		AC_MSG_ERROR([
			Cannot find required GStreamer-1.0 plugin '$1'.
			It should be part of gst-plugins-good. Please install it.
		])
	fi
	shift;
done

# ================================================================
# Python plugins
# ================================================================
# We don't Python or PyGObject at compile time, but they are needed for the Python plugins at runtime, so we check anyway.
# We need pylint for `make check`.

AC_MSG_CHECKING([whether Python plugin support is requested])
AC_ARG_ENABLE([python],
	AS_HELP_STRING([--enable-python],[Enable python support]),
	[enable_python=$enableval],
	[enable_python=autodetect])
AC_MSG_RESULT([$enable_python])

have_python=no
have_pygobject=no
PYTHON_MODULES=""

# Test for the Python run time and PyGObject. Test for pylint as well, for `make check`.
if test "x$enable_python" != "xno"; then
	AM_PATH_PYTHON([$PYTHON_REQS],[have_python=yes],[have_python=no])
	if test "x$PYTHON" = "x:"; then
		have_python=no
	fi

	PYTHON_MODULES="pygobject-3.0 >= $PYGOBJECT_REQS"
	PKG_CHECK_MODULES([PYGOBJECT],[$PYTHON_MODULES],[have_pygobject=yes],[have_pygobject=no])
	AC_PATH_PROG([PYLINT], [pylint], [no])
fi

# Output the results of the Python checks
if test "x$have_python" != "xyes" -o "x$have_pygobject" != "xyes" -o "x$PYLINT" = "xno"; then
	if test "x$enable_python" = "xyes"; then
		AC_MSG_ERROR([python >= $PYTHON_REQS, $PYTHON_MODULES or pylint not found])
	elif test "x$enable_python" = "xautodetect"; then
		enable_python=no
		AC_MSG_WARN([python >= $PYTHON_REQS, $PYTHON_MODULES or pylint not found; disabling Python support])
	fi
elif test "x$enable_python" != "xno"; then
	enable_python=yes
fi

AM_CONDITIONAL([ENABLE_PYTHON],[test "x$enable_python" = "xyes"])

# ================================================================
# Player requirements
#================================================================

PKG_CHECK_MODULES(PLAYER, $BACKEND_MODULES glib-2.0 >= $GLIB_REQS gio-2.0 >= $GIO_REQS gtk+-3.0 >= $GTK_REQS gdk-x11-3.0 >= $GTK_REQS gmodule-2.0 totem-plparser >= $TOTEM_PLPARSER_REQS libpeas-1.0 >= $PEAS_REQS libpeas-gtk-1.0 >= $PEAS_REQS $PYTHON_MODULES grilo-0.3 >= $GRILO_REQS grilo-pls-0.3 >= $GRILO_REQS gnome-desktop-3.0)
PKG_CHECK_MODULES(LIBPLAYER, glib-2.0 >= $GLIB_REQS gio-2.0 >= $GIO_REQS gtk+-3.0 >= $GTK_REQS gdk-x11-3.0 >= $GTK_REQS clutter-gtk-1.0)
PKG_CHECK_MODULES(HELPER, gstreamer-1.0 gstreamer-tag-1.0)
PKG_CHECK_MODULES(TIME_HELPER, glib-2.0)
PKG_CHECK_MODULES(RTL_HELPER, glib-2.0 gtk+-3.0 >= $GTK_REQS)
PKG_CHECK_MODULES(PIXBUF_HELPER, gdk-pixbuf-2.0 gstreamer-tag-1.0 >= $GSTPLUG_REQS gstreamer-video-1.0)
PKG_CHECK_MODULES(THUMBNAILER, gdk-3.0 totem-plparser >= $TOTEM_PLPARSER_REQS gstreamer-tag-1.0 >= $GSTPLUG_REQS gstreamer-video-1.0)
PKG_CHECK_MODULES(PREVIEW, gstreamer-1.0 >= $GST_REQS gio-2.0 >= $GIO_REQS gdk-pixbuf-2.0)
PKG_CHECK_MODULES(PROPERTIES, gtk+-3.0 >= $GTK_REQS)

LIBGD_INIT([
	tagged-entry
	main-view
	static
 ])


dnl ================================================================
dnl Vala plugins
dnl ================================================================
AC_MSG_CHECKING([whether Vala plugin support is requested])
AC_ARG_ENABLE([vala],
	AS_HELP_STRING([--enable-vala],[Enable Vala plugin support]),
	[enable_vala=$enableval],
	[enable_vala=auto])
AC_MSG_RESULT([$enable_vala])
if test "x$enable_vala" != "xno"; then
	AM_PROG_VALAC([$VALA_REQS])
	if test "$VALAC" = "valac"; then
		if test "x$enable_vala" = "xyes"; then
			AC_MSG_ERROR([Vala plugin support explicitly requested, but not found])
		fi
		with_vala=no
	else
		with_vala=yes
	fi
fi

AM_CONDITIONAL(ENABLE_VALA, test "x$with_vala" = "xyes")

# ********************
# Movie player plugins
# ********************

plugin_error_or_ignore()
{
	if test "${error_on_bad_plugin}" = "1" ; then
		AC_MSG_ERROR([$1])
	else
		AC_MSG_WARN([$1 (disabling plugin)])
	fi
}

plugin_error()
{
	AC_MSG_ERROR([$1])
}

AC_ARG_WITH([plugins],
	[AS_HELP_STRING([--with-plugins],[Which Totem plugins to compile (default: auto; "all", "none" and "autodetect" are valid)])],
	[], [with_plugins=autodetect])

used_plugins=""
used_plugins2=""

if test "x${with_plugins}" = "xautodetect" ; then
	# Default to all plugins and just ignore them if they won't compile
	with_plugins="all"
	error_on_bad_plugin="0"
else
	# Clean up the plugin list and error if a plugin won't compile
	with_plugins=`echo ,"${with_plugins}", | sed -e 's/[[	,]][[	,]]*/ /g' -e 's/,$//'`
	error_on_bad_plugin="1"
fi

# Iterate through the wanted plugins and weed out bad ones (or error)
for wanted_plugin in ${with_plugins}; do
	for allowed_plugin in ${allowed_plugins}; do
		case ${wanted_plugin} in
			all)
				# List all plugins as being used
				used_plugins=${allowed_plugins}
			;;
			autodetect)
				# List all plugins as being used
				used_plugins=${allowed_plugins}
			;;
			none) used_plugins="";; # List no plugins
			${allowed_plugin}) used_plugins="${used_plugins} ${wanted_plugin}";; # List the plugin
			*) ;;
		esac
	done
done

# Check for plugin-specific requirements and error if necessary
for plugin in ${used_plugins}; do
	add_plugin="1"

	case ${plugin} in
		gromit)
			have_x11=no
			gdk_targets=$(pkg-config --variable=targets gtk+-3.0)
			for gdk_target in $gdk_targets; do
				if test x$gdk_target = xx11; then
					have_x11=yes
					break
				fi
			done

			if test "${have_x11}" != "yes" ; then
				plugin_error_or_ignore "the gromit plugin is not supported on non-X11 targets"
				add_plugin="0"
			fi
		;;
		media-player-keys)
			PKG_CHECK_MODULES(MEDIA_PLAYER_KEYS, gio-2.0,
				[HAVE_MEDIA_PLAYER_KEYS=yes], [HAVE_MEDIA_PLAYER_KEYS=no])
			if test "${HAVE_MEDIA_PLAYER_KEYS}" != "yes" ; then
				plugin_error "you need gio installed for the media-player-keys plugin"
				add_plugin="0"
			fi
		;;
		lirc)
			AC_CHECK_LIB(lirc_client, lirc_init,
				[AC_CHECK_HEADER(lirc/lirc_client.h,
				 	[LIRC_LIBS="-llirc_client"
					 HAVE_LIRC=yes],
					[
					 LIRC_LIBS=""
					 HAVE_LIRC=no])])
			AC_SUBST(LIRC_LIBS)
			if test "${HAVE_LIRC}" != "yes" ; then
				plugin_error_or_ignore "you need lirc_client installed for the lirc plugin"
				add_plugin="0"
			fi
		;;
		rotation)
			if test "${with_vala}" != "yes" ; then
				plugin_error_or_ignore "you need vala installed to use the rotation plugin"
				add_plugin="0"
                        elif test "x$found_introspection" != "xyes"; then
				plugin_error_or_ignore "you need introspection support for the rotation plugin"
				add_plugin="0"
			fi
		;;
		sample-vala)
			if test "${with_vala}" != "yes" ; then
				plugin_error_or_ignore "you need vala installed to use the sample-vala plugin"
				add_plugin="0"
                        elif test "x$found_introspection" != "xyes"; then
				plugin_error_or_ignore "you need introspection support for the sample-vala plugin"
				add_plugin="0"
			fi
		;;
		brasero-disc-recorder)
			dnl this is only needed to use brasero icons
			PKG_CHECK_MODULES(DISC_RECORDER, libxml-2.0 >= 2.6.0 gtk+-x11-3.0,
					[BUILD_RECORDER=yes], [BUILD_RECORDER=no])
			if test "${BUILD_RECORDER}" != "yes" ; then
				plugin_error_or_ignore "you need libxml-2.0 >= 2.6.0 and gtk+-x11-3.0 to use the brasero-disc-recorder plugin"
				add_plugin="0"
			fi
		;;
		save-file)
			dnl the nautilus check should be a run-time check
			dnl but it might not always be running
			PKG_CHECK_MODULES(SAVE_FILE_CHECK, gio-2.0 >= 2.26 libnautilus-extension >= 2.91.3,
					  [BUILD_SAVE_FILE=yes], [BUILD_SAVE_FILE=no])
			if test "${BUILD_SAVE_FILE}" != "yes" ; then
				plugin_error_or_ignore "you need gio-2.0 >=2.26 to use the save-file plugin"
				add_plugin="0"
			else
				PKG_CHECK_MODULES(SAVE_FILE, gio-2.0 >= 2.26)
			fi
		;;
		zeitgeist-dp)
			if test "${with_vala}" != "yes" ; then
				plugin_error_or_ignore "you need vala installed to use the zeitgeist-dp plugin"
				add_plugin="0"
                        elif test "x$found_introspection" != "xyes"; then
				plugin_error_or_ignore "you need introspection support for the zeitgeist-dp plugin"
				add_plugin="0"
			else
				PKG_CHECK_MODULES(LIBZEITGEIST, zeitgeist-2.0 >= 0.9.12,
				[HAS_LIBZEITGEIST=yes], [HAS_LIBZEITGEIST=no])
				if test "${HAS_LIBZEITGEIST}" != "yes" ; then
					plugin_error_or_ignore "you need zeitgeist-2.0 >= 0.9.12 to use the zeitgeist-dp plugin"
					add_plugin="0"
				fi
			fi
		;;
	esac

	# Add the specified plugin
	if test "${add_plugin}" = "1" ; then
		used_plugins2="${used_plugins2} ${plugin}"
	fi
done

ALL_PLUGINS=$allowed_plugins
PLUGINS=$used_plugins2

AC_MSG_CHECKING([which plugins to compile])
AC_MSG_RESULT([$PLUGINS])

AC_SUBST([ALL_PLUGINS])
AC_SUBST([PLUGINS])

dnl ********
dnl Nautilus
dnl ********

dnl Check for Nautilus property page build
AC_ARG_ENABLE(nautilus,
	AS_HELP_STRING([--enable-nautilus],[compile the nautilus plugin]),
	[case "${enableval}" in
	yes) ENABLE_NAUTILUS=yes ;;
	no) ENABLE_NAUTILUS=no ;;
	*) AC_MSG_ERROR(bad value ${enableval} for --enable-nautilus) ;;
	esac],
	[ENABLE_NAUTILUS=yes]) dnl Default value

if test x$ENABLE_NAUTILUS = "xyes" ; then
	PKG_CHECK_MODULES(NAUTILUS, gtk+-3.0 gthread-2.0 libnautilus-extension gstreamer-tag-1.0 >= $GSTPLUG_REQS $MISSING_PLUGINS_MODULES,
			[HAVE_NAUTILUS=yes], [HAVE_NAUTILUS=no])
fi

AC_ARG_WITH(nautilusdir,
	    AS_HELP_STRING([--with-nautilusdir=DIR],[Installation path for Nautilus extension @<:@auto@:>@]),
	    [ac_with_nautilusdir=$withval],
	    [ac_with_nautilusdir=""])

if test x$HAVE_NAUTILUS = "xyes"; then
	AC_DEFINE(HAVE_NAUTILUS, 1, [defined if you build the nautilus plugin])
	if test "${ac_with_nautilusdir}" = ""; then
		ac_with_nautilusdir=`pkg-config --variable=extensiondir libnautilus-extension`
	fi
	AC_SUBST(NAUTILUSDIR)
	AC_MSG_NOTICE([installing nautilus plugin in ${ac_with_nautilusdir}])
	AC_SUBST([NAUTILUSDIR],[${ac_with_nautilusdir}])

fi
AM_CONDITIONAL(HAVE_NAUTILUS, test x$HAVE_NAUTILUS = "xyes")

GLIB_GSETTINGS

GNOME_DEBUG_CHECK
GNOME_COMPILE_WARNINGS([maximum])
GNOME_CXX_WARNINGS
GNOME_MAINTAINER_MODE_DEFINES
GTK_DOC_CHECK([1.14],[--flavour no-tmpl])

APPSTREAM_XML

# This macro expands DIR and assigns it to RET.
# If DIR is NONE, then it's replaced by DEFAULT.
# Based on AC_DEFINE_DIR
AC_DEFUN([TOTEM_FULLPATH], [
  test "x$prefix" = xNONE && prefix="$ac_default_prefix"
  test "x$exec_prefix" = xNONE && exec_prefix='${prefix}'
  ac_expand=[$]$1
  test "x$ac_expand" = xNONE && ac_expand="[$]$2"
  ac_expand=`eval echo [$]ac_expand`
  $3=`eval echo [$]ac_expand`
])

dnl FULL_LIBEXECDIR is used for X-GNOME-Bugzilla-ExtraInfoScript expansion
dnl in data/org.gnome.Totem.desktop.in.in.in
TOTEM_FULLPATH(libexecdir, NONE, FULL_LIBEXECDIR)
AC_SUBST(FULL_LIBEXECDIR)

dnl run in source tree
AC_ARG_ENABLE(run-in-source-tree,
	      AS_HELP_STRING([--disable-run-in-source-tree],[disable search for UI files and plugins in local directory ]),
	[case "${enableval}" in
	yes) ENABLE_RUN_IN_SOURCE_TREE=yes ;;
	no)  ENABLE_RUN_IN_SOURCE_TREE=no ;;
	*) AC_MSG_ERROR(bad value ${enableval} for --disable-run-in-source-tree) ;;
	esac],
[ENABLE_RUN_IN_SOURCE_TREE=yes]) dnl Default value
if test x$ENABLE_RUN_IN_SOURCE_TREE = xyes; then
	AC_DEFINE(TOTEM_RUN_IN_SOURCE_TREE, 1, [enable search for UI files and plugins in local directory])
fi

if test "$GCC" = "yes" ; then
	CFLAGS="\
	-Wall \
	-Wchar-subscripts -Wmissing-declarations -Wmissing-prototypes \
	-Wnested-externs -Wpointer-arith \
	-Wcast-align -Wsign-compare \
	-fno-strict-aliasing \
	$CFLAGS"
fi

AC_SUBST([AM_CPPFLAGS])
AC_SUBST([AM_CFLAGS])
AC_SUBST([AM_CXXFLAGS])
AC_SUBST([AM_LDFLAGS])

AC_CONFIG_FILES([
Makefile
totem.spec
libgd/Makefile
src/Makefile
src/gst/Makefile
src/properties/Makefile
src/plugins/Makefile
src/plugins/apple-trailers/Makefile
src/plugins/autoload-subtitles/Makefile
src/plugins/dbusservice/Makefile
src/plugins/screensaver/Makefile
src/plugins/screenshot/Makefile
src/plugins/ontop/Makefile
src/plugins/im-status/Makefile
src/plugins/gromit/Makefile
src/plugins/lirc/Makefile
src/plugins/media-player-keys/Makefile
src/plugins/opensubtitles/Makefile
src/plugins/opensubtitles/org.gnome.totem.plugins.opensubtitles.gschema.xml
src/plugins/properties/Makefile
src/plugins/recent/Makefile
src/plugins/rotation/Makefile
src/plugins/save-file/Makefile
src/plugins/skipto/Makefile
src/plugins/samplepython/Makefile
src/plugins/sample-vala/Makefile
src/plugins/pythonconsole/Makefile
src/plugins/pythonconsole/org.gnome.totem.plugins.pythonconsole.gschema.xml
src/plugins/brasero-disc-recorder/Makefile
src/plugins/zeitgeist-dp/Makefile
src/plugins/variable-rate/Makefile
src/plugins/vimeo/Makefile
src/backend/Makefile
data/Makefile
data/org.gnome.totem.gschema.xml
data/org.gnome.Totem.desktop.in.in
data/totem.pc
data/appdata/Makefile
data/icons/Makefile
data/icons/16x16/Makefile
data/icons/22x22/Makefile
data/icons/24x24/Makefile
data/icons/32x32/Makefile
data/icons/48x48/Makefile
data/icons/256x256/Makefile
data/icons/symbolic/Makefile
po/Makefile.in
help/Makefile
docs/Makefile
docs/reference/Makefile
docs/reference/version.xml
])
AC_OUTPUT

AC_MSG_NOTICE([Totem was configured with the following options:])
AC_MSG_NOTICE([** Using the GStreamer-$GST_MAJORMINOR backend])
if test "x$MISSING_PLUGINS_LIBS" != "x"; then
	AC_MSG_NOTICE([** Easy codec installation support enabled])
else
	AC_MSG_NOTICE([   Easy codec installation support disabled])
fi
if test x$enable_python = xyes ; then
	AC_MSG_NOTICE([** Python plugin support enabled])
else
	AC_MSG_NOTICE([   Python plugin support disabled])
fi
if test x$with_vala = xyes ; then
	AC_MSG_NOTICE([** Vala plugin support enabled])
else
	AC_MSG_NOTICE([   Vala plugin support disabled])
fi
if test "x${PLUGINS}" != "x" ; then
	for allowed_plugin in ${ALL_PLUGINS}; do
		for plugin in ${PLUGINS}; do
			case ${allowed_plugin} in
				${plugin})
					AC_MSG_NOTICE([** ${allowed_plugin} plugin enabled])
					continue 2
				;;
				*);;
			esac
		done
		AC_MSG_NOTICE([   ${allowed_plugin} plugin disabled])
	done
else
	AC_MSG_NOTICE([   No Totem plugins enabled])
fi
if test x$ENABLE_NAUTILUS = xyes ; then
	AC_MSG_NOTICE([** Nautilus properties page enabled])
else
	AC_MSG_NOTICE([   Nautilus properties page disabled])
fi
if test "$with_smclient" != "no" ; then
	AC_MSG_NOTICE([** SM client support enabled])
else
	AC_MSG_NOTICE([   SM client support disabled])
fi

AC_MSG_NOTICE([End options])

