# CMakeLists.txt

PROJECT(PCRE C CXX)

CMAKE_MINIMUM_REQUIRED(VERSION 2.8.0)
#CMAKE_POLICY(SET CMP0026 OLD)

set(PCRE_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/..)

set(PCRE_INCLUDE_DIR ${PCRE_SOURCE_DIR} CACHE INTERNAL "pcre library" FORCE)
set(PCRE_LIBRARY ${PROJECT_BINARY_DIR} CACHE INTERNAL "pcre library" FORCE)

SET(CMAKE_MODULE_PATH ${PCRE_SOURCE_DIR}/cmake) # for FindReadline.cmake

# Configuration checks

INCLUDE(CheckIncludeFile)
INCLUDE(CheckIncludeFileCXX)
INCLUDE(CheckFunctionExists)
INCLUDE(CheckTypeSize)

CHECK_INCLUDE_FILE(dirent.h     HAVE_DIRENT_H)
CHECK_INCLUDE_FILE(stdint.h     HAVE_STDINT_H)
CHECK_INCLUDE_FILE(inttypes.h   HAVE_INTTYPES_H)
CHECK_INCLUDE_FILE(sys/stat.h   HAVE_SYS_STAT_H)
CHECK_INCLUDE_FILE(sys/types.h  HAVE_SYS_TYPES_H)
CHECK_INCLUDE_FILE(unistd.h     HAVE_UNISTD_H)
CHECK_INCLUDE_FILE(windows.h    HAVE_WINDOWS_H)

CHECK_INCLUDE_FILE_CXX(type_traits.h            HAVE_TYPE_TRAITS_H)
CHECK_INCLUDE_FILE_CXX(bits/type_traits.h       HAVE_BITS_TYPE_TRAITS_H)

CHECK_FUNCTION_EXISTS(bcopy     HAVE_BCOPY)
CHECK_FUNCTION_EXISTS(memmove   HAVE_MEMMOVE)
CHECK_FUNCTION_EXISTS(strerror  HAVE_STRERROR)
CHECK_FUNCTION_EXISTS(strtoll   HAVE_STRTOLL)
CHECK_FUNCTION_EXISTS(strtoq    HAVE_STRTOQ)
CHECK_FUNCTION_EXISTS(_strtoi64 HAVE__STRTOI64)

CHECK_TYPE_SIZE("long long"             LONG_LONG)
CHECK_TYPE_SIZE("unsigned long long"    UNSIGNED_LONG_LONG)

# User-configurable options
#
# (Note: CMakeSetup displays these in alphabetical order, regardless of
# the order we use here)

SET(BUILD_SHARED_LIBS OFF CACHE BOOL
    "Build shared libraries instead of static ones.")

OPTION(PCRE_BUILD_PCRE8 "Build 8 bit PCRE library" ON)

OPTION(PCRE_BUILD_PCRE16 "Build 16 bit PCRE library" OFF)

OPTION(PCRE_BUILD_PCRE32 "Build 32 bit PCRE library" OFF)

OPTION(PCRE_BUILD_PCRECPP "Build the PCRE C++ library (pcrecpp)." ON)

SET(PCRE_EBCDIC OFF CACHE BOOL
    "Use EBCDIC coding instead of ASCII. (This is rarely used outside of mainframe systems.)")

SET(PCRE_EBCDIC_NL25 OFF CACHE BOOL
    "Use 0x25 as EBCDIC NL character instead of 0x15; implies EBCDIC.")

SET(PCRE_LINK_SIZE "2" CACHE STRING
    "Internal link size (2, 3 or 4 allowed). See LINK_SIZE in config.h.in for details.")

SET(PCRE_PARENS_NEST_LIMIT "250" CACHE STRING
    "Default nested parentheses limit. See PARENS_NEST_LIMIT in config.h.in for details.")

SET(PCRE_MATCH_LIMIT "10000000" CACHE STRING
    "Default limit on internal looping. See MATCH_LIMIT in config.h.in for details.")

SET(PCRE_MATCH_LIMIT_RECURSION "MATCH_LIMIT" CACHE STRING
    "Default limit on internal recursion. See MATCH_LIMIT_RECURSION in config.h.in for details.")

SET(PCREGREP_BUFSIZE "20480" CACHE STRING
    "Buffer size parameter for pcregrep. See PCREGREP_BUFSIZE in config.h.in for details.")

SET(PCRE_NEWLINE "LF" CACHE STRING
    "What to recognize as a newline (one of CR, LF, CRLF, ANY, ANYCRLF).")

SET(PCRE_NO_RECURSE OFF CACHE BOOL
    "If ON, then don't use stack recursion when matching. See NO_RECURSE in config.h.in for details.")

SET(PCRE_POSIX_MALLOC_THRESHOLD "10" CACHE STRING
    "Threshold for malloc() usage. See POSIX_MALLOC_THRESHOLD in config.h.in for details.")

SET(PCRE_SUPPORT_JIT OFF CACHE BOOL
    "Enable support for Just-in-time compiling.")

SET(PCRE_SUPPORT_PCREGREP_JIT ON CACHE BOOL
    "Enable use of Just-in-time compiling in pcregrep.")

SET(PCRE_SUPPORT_UTF OFF CACHE BOOL
    "Enable support for Unicode Transformation Format (UTF-8/UTF-16/UTF-32) encoding.")

SET(PCRE_SUPPORT_UNICODE_PROPERTIES OFF CACHE BOOL
    "Enable support for Unicode properties (if set, UTF support will be enabled as well).")

SET(PCRE_SUPPORT_BSR_ANYCRLF OFF CACHE BOOL
    "ON=Backslash-R matches only LF CR and CRLF, OFF=Backslash-R matches all Unicode Linebreaks")

SET(PCRE_SUPPORT_VALGRIND OFF CACHE BOOL
    "Enable Valgrind support.")

OPTION(PCRE_SHOW_REPORT    "Show the final configuration report" ON)
OPTION(PCRE_BUILD_PCREGREP "Build pcregrep" ON)
OPTION(PCRE_BUILD_TESTS    "Build the tests" ON)

IF (MINGW)
  OPTION(NON_STANDARD_LIB_PREFIX
         "ON=Shared libraries built in mingw will be named pcre.dll, etc., instead of libpcre.dll, etc."
         OFF)

  OPTION(NON_STANDARD_LIB_SUFFIX
         "ON=Shared libraries built in mingw will be named libpcre-0.dll, etc., instead of libpcre.dll, etc."
         OFF)
ENDIF(MINGW)

IF(MSVC)
  OPTION(PCRE_STATIC_RUNTIME OFF CACHE BOOL
	"ON=Compile against the static runtime (/MT)."
	OFF)
  OPTION(INSTALL_MSVC_PDB
         "ON=Install .pdb files built by MSVC, if generated"
         OFF)
ENDIF(MSVC)

# Prepare build configuration

SET(pcre_have_type_traits 0)
SET(pcre_have_bits_type_traits 0)

IF(HAVE_TYPE_TRAITS_H)
        SET(pcre_have_type_traits 1)
ENDIF(HAVE_TYPE_TRAITS_H)

IF(HAVE_BITS_TYPE_TRAITS_H)
        SET(pcre_have_bits_type_traits 1)
ENDIF(HAVE_BITS_TYPE_TRAITS_H)

SET(pcre_have_long_long 0)
SET(pcre_have_ulong_long 0)

IF(HAVE_LONG_LONG)
        SET(pcre_have_long_long 1)
ENDIF(HAVE_LONG_LONG)

IF(HAVE_UNSIGNED_LONG_LONG)
        SET(pcre_have_ulong_long 1)
ENDIF(HAVE_UNSIGNED_LONG_LONG)

IF(NOT BUILD_SHARED_LIBS)
        SET(PCRE_STATIC 1)
ENDIF(NOT BUILD_SHARED_LIBS)

IF(NOT PCRE_BUILD_PCRE8 AND NOT PCRE_BUILD_PCRE16 AND NOT PCRE_BUILD_PCRE32)
        MESSAGE(FATAL_ERROR "At least one of PCRE_BUILD_PCRE8, PCRE_BUILD_PCRE16 or PCRE_BUILD_PCRE32 must be enabled")
ENDIF(NOT PCRE_BUILD_PCRE8 AND NOT PCRE_BUILD_PCRE16 AND NOT PCRE_BUILD_PCRE32)

IF(PCRE_BUILD_PCRE8)
        SET(SUPPORT_PCRE8 1)
ENDIF(PCRE_BUILD_PCRE8)

IF(PCRE_BUILD_PCRE16)
        SET(SUPPORT_PCRE16 1)
ENDIF(PCRE_BUILD_PCRE16)

IF(PCRE_BUILD_PCRE32)
        SET(SUPPORT_PCRE32 1)
ENDIF(PCRE_BUILD_PCRE32)

IF(PCRE_BUILD_PCRECPP AND NOT PCRE_BUILD_PCRE8)
        MESSAGE(STATUS "** PCRE_BUILD_PCRE8 must be enabled for the C++ library support")
        SET(PCRE_BUILD_PCRECPP OFF)
ENDIF(PCRE_BUILD_PCRECPP AND NOT PCRE_BUILD_PCRE8)

IF(PCRE_BUILD_PCREGREP AND NOT PCRE_BUILD_PCRE8)
        MESSAGE(STATUS "** PCRE_BUILD_PCRE8 must be enabled for the pcregrep program")
        SET(PCRE_BUILD_PCREGREP OFF)
ENDIF(PCRE_BUILD_PCREGREP AND NOT PCRE_BUILD_PCRE8)

IF(PCRE_SUPPORT_LIBREADLINE AND PCRE_SUPPORT_LIBEDIT)
        MESSAGE(FATAL_ERROR "Only one of libreadline or libeditline can be specified")
ENDIF(PCRE_SUPPORT_LIBREADLINE AND PCRE_SUPPORT_LIBEDIT)

IF(PCRE_SUPPORT_BSR_ANYCRLF)
        SET(BSR_ANYCRLF 1)
ENDIF(PCRE_SUPPORT_BSR_ANYCRLF)

IF(PCRE_SUPPORT_UTF OR PCRE_SUPPORT_UNICODE_PROPERTIES)
        SET(SUPPORT_UTF 1)
        SET(PCRE_SUPPORT_UTF ON)
ENDIF(PCRE_SUPPORT_UTF OR PCRE_SUPPORT_UNICODE_PROPERTIES)

IF(PCRE_SUPPORT_UNICODE_PROPERTIES)
        SET(SUPPORT_UCP 1)
ENDIF(PCRE_SUPPORT_UNICODE_PROPERTIES)

IF(PCRE_SUPPORT_JIT)
        SET(SUPPORT_JIT 1)
ENDIF(PCRE_SUPPORT_JIT)

IF(PCRE_SUPPORT_PCREGREP_JIT)
        SET(SUPPORT_PCREGREP_JIT 1)
ENDIF(PCRE_SUPPORT_PCREGREP_JIT)

IF(PCRE_SUPPORT_VALGRIND)
        SET(SUPPORT_VALGRIND 1)
ENDIF(PCRE_SUPPORT_VALGRIND)

# This next one used to contain
#       SET(PCRETEST_LIBS ${READLINE_LIBRARY})
# but I was advised to add the NCURSES test as well, along with
# some modifications to cmake/FindReadline.cmake which should
# make it possible to override the default if necessary. PH

IF(PCRE_SUPPORT_LIBREADLINE)
        SET(SUPPORT_LIBREADLINE 1)
        SET(PCRETEST_LIBS ${READLINE_LIBRARY} ${NCURSES_LIBRARY})
ENDIF(PCRE_SUPPORT_LIBREADLINE)

# libedit is a plug-compatible alternative to libreadline

IF(PCRE_SUPPORT_LIBEDIT)
        SET(SUPPORT_LIBEDIT 1)
        SET(PCRETEST_LIBS ${EDITLINE_LIBRARY} ${NCURSES_LIBRARY})
ENDIF(PCRE_SUPPORT_LIBEDIT)

IF(PCRE_SUPPORT_LIBZ)
        SET(SUPPORT_LIBZ 1)
        SET(PCREGREP_LIBS ${PCREGREP_LIBS} ${ZLIB_LIBRARIES})
ENDIF(PCRE_SUPPORT_LIBZ)

IF(PCRE_SUPPORT_LIBBZ2)
        SET(SUPPORT_LIBBZ2 1)
        SET(PCREGREP_LIBS ${PCREGREP_LIBS} ${BZIP2_LIBRARIES})
ENDIF(PCRE_SUPPORT_LIBBZ2)

SET(NEWLINE "")

IF(PCRE_NEWLINE STREQUAL "LF")
        SET(NEWLINE "10")
ENDIF(PCRE_NEWLINE STREQUAL "LF")
IF(PCRE_NEWLINE STREQUAL "CR")
        SET(NEWLINE "13")
ENDIF(PCRE_NEWLINE STREQUAL "CR")
IF(PCRE_NEWLINE STREQUAL "CRLF")
        SET(NEWLINE "3338")
ENDIF(PCRE_NEWLINE STREQUAL "CRLF")
IF(PCRE_NEWLINE STREQUAL "ANY")
        SET(NEWLINE "-1")
ENDIF(PCRE_NEWLINE STREQUAL "ANY")
IF(PCRE_NEWLINE STREQUAL "ANYCRLF")
        SET(NEWLINE "-2")
ENDIF(PCRE_NEWLINE STREQUAL "ANYCRLF")

IF(NEWLINE STREQUAL "")
        MESSAGE(FATAL_ERROR "The PCRE_NEWLINE variable must be set to one of the following values: \"LF\", \"CR\", \"CRLF\", \"ANY\", \"ANYCRLF\".")
ENDIF(NEWLINE STREQUAL "")

IF(PCRE_EBCDIC)
        SET(EBCDIC 1)
IF(PCRE_NEWLINE STREQUAL "LF")
        SET(NEWLINE "21")
ENDIF(PCRE_NEWLINE STREQUAL "LF")
IF(PCRE_NEWLINE STREQUAL "CRLF")
        SET(NEWLINE "3349")
ENDIF(PCRE_NEWLINE STREQUAL "CRLF")
ENDIF(PCRE_EBCDIC)

IF(PCRE_EBCDIC_NL25)
        SET(EBCDIC 1)
        SET(EBCDIC_NL25 1)
IF(PCRE_NEWLINE STREQUAL "LF")
        SET(NEWLINE "37")
ENDIF(PCRE_NEWLINE STREQUAL "LF")
IF(PCRE_NEWLINE STREQUAL "CRLF")
        SET(NEWLINE "3365")
ENDIF(PCRE_NEWLINE STREQUAL "CRLF")
ENDIF(PCRE_EBCDIC_NL25)

IF(PCRE_NO_RECURSE)
        SET(NO_RECURSE 1)
ENDIF(PCRE_NO_RECURSE)

# Output files
CONFIGURE_FILE(${PCRE_SOURCE_DIR}/config-cmake.h.in
               ${PROJECT_BINARY_DIR}/config.h
               @ONLY)

# Parse version numbers and date out of configure.ac

file(STRINGS ${PCRE_SOURCE_DIR}/configure.ac
  configure_lines
  LIMIT_COUNT 50 # Read only the first 50 lines of the file
)

set(SEARCHED_VARIABLES "pcre_major" "pcre_minor" "pcre_prerelease" "pcre_date")
foreach(configure_line ${configure_lines})
    foreach(_substitution_variable ${SEARCHED_VARIABLES})
        string(TOUPPER ${_substitution_variable} _substitution_variable_upper)
        if (NOT ${_substitution_variable_upper})
            string(REGEX MATCH "m4_define\\(${_substitution_variable}, \\[(.*)\\]" MACTHED_STRING ${configure_line})
            if (CMAKE_MATCH_1)
                set(${_substitution_variable_upper} ${CMAKE_MATCH_1})
            endif()
        endif()
    endforeach()
endforeach()

CONFIGURE_FILE(${PCRE_SOURCE_DIR}/pcre.h.in
               ${PROJECT_BINARY_DIR}/pcre.h
               @ONLY)

# What about pcre-config and libpcre.pc?

IF(PCRE_BUILD_PCRECPP)
        CONFIGURE_FILE(${PCRE_SOURCE_DIR}/pcre_stringpiece.h.in
                       ${PROJECT_BINARY_DIR}/pcre_stringpiece.h
                       @ONLY)

        CONFIGURE_FILE(${PCRE_SOURCE_DIR}/pcrecpparg.h.in
                       ${PROJECT_BINARY_DIR}/pcrecpparg.h
                       @ONLY)
ENDIF(PCRE_BUILD_PCRECPP)

# Character table generation

OPTION(PCRE_REBUILD_CHARTABLES "Rebuild char tables" OFF)
IF(PCRE_REBUILD_CHARTABLES)
  ADD_EXECUTABLE(dftables dftables.c)

  GET_TARGET_PROPERTY(DFTABLES_EXE dftables LOCATION)

  ADD_CUSTOM_COMMAND(
    COMMENT "Generating character tables (pcre_chartables.c) for current locale"
    DEPENDS dftables
    COMMAND ${DFTABLES_EXE}
    ARGS        ${PROJECT_BINARY_DIR}/pcre_chartables.c
    OUTPUT      ${PROJECT_BINARY_DIR}/pcre_chartables.c
  )
ELSE(PCRE_REBUILD_CHARTABLES)
  CONFIGURE_FILE(${PCRE_SOURCE_DIR}/pcre_chartables.c.dist
                    ${PROJECT_BINARY_DIR}/pcre_chartables.c
                    COPYONLY)
ENDIF(PCRE_REBUILD_CHARTABLES)

# Source code

SET(PCRE_HEADERS ${PROJECT_BINARY_DIR}/pcre.h)

IF(PCRE_BUILD_PCRE8)
SET(PCRE_SOURCES
  ${PCRE_SOURCE_DIR}/pcre_byte_order.c
  ${PCRE_SOURCE_DIR}/pcre_chartables.c
  ${PCRE_SOURCE_DIR}/pcre_compile.c
  ${PCRE_SOURCE_DIR}/pcre_config.c
  ${PCRE_SOURCE_DIR}/pcre_dfa_exec.c
  ${PCRE_SOURCE_DIR}/pcre_exec.c
  ${PCRE_SOURCE_DIR}/pcre_fullinfo.c
  ${PCRE_SOURCE_DIR}/pcre_get.c
  ${PCRE_SOURCE_DIR}/pcre_globals.c
  ${PCRE_SOURCE_DIR}/pcre_jit_compile.c
  ${PCRE_SOURCE_DIR}/pcre_maketables.c
  ${PCRE_SOURCE_DIR}/pcre_newline.c
  ${PCRE_SOURCE_DIR}/pcre_ord2utf8.c
  ${PCRE_SOURCE_DIR}/pcre_refcount.c
  ${PCRE_SOURCE_DIR}/pcre_string_utils.c
  ${PCRE_SOURCE_DIR}/pcre_study.c
  ${PCRE_SOURCE_DIR}/pcre_tables.c
  ${PCRE_SOURCE_DIR}/pcre_ucd.c
  ${PCRE_SOURCE_DIR}/pcre_valid_utf8.c
  ${PCRE_SOURCE_DIR}/pcre_version.c
  ${PCRE_SOURCE_DIR}/pcre_xclass.c
)

SET(PCREPOSIX_HEADERS ${PCRE_SOURCE_DIR}/pcreposix.h)

SET(PCREPOSIX_SOURCES ${PCRE_SOURCE_DIR}/pcreposix.c)

ENDIF(PCRE_BUILD_PCRE8)

IF(PCRE_BUILD_PCRE16)
SET(PCRE16_SOURCES
  ${PCRE_SOURCE_DIR}/pcre16_byte_order.c
  ${PCRE_SOURCE_DIR}/pcre16_chartables.c
  ${PCRE_SOURCE_DIR}/pcre16_compile.c
  ${PCRE_SOURCE_DIR}/pcre16_config.c
  ${PCRE_SOURCE_DIR}/pcre16_dfa_exec.c
  ${PCRE_SOURCE_DIR}/pcre16_exec.c
  ${PCRE_SOURCE_DIR}/pcre16_fullinfo.c
  ${PCRE_SOURCE_DIR}/pcre16_get.c
  ${PCRE_SOURCE_DIR}/pcre16_globals.c
  ${PCRE_SOURCE_DIR}/pcre16_jit_compile.c
  ${PCRE_SOURCE_DIR}/pcre16_maketables.c
  ${PCRE_SOURCE_DIR}/pcre16_newline.c
  ${PCRE_SOURCE_DIR}/pcre16_ord2utf16.c
  ${PCRE_SOURCE_DIR}/pcre16_refcount.c
  ${PCRE_SOURCE_DIR}/pcre16_string_utils.c
  ${PCRE_SOURCE_DIR}/pcre16_study.c
  ${PCRE_SOURCE_DIR}/pcre16_tables.c
  ${PCRE_SOURCE_DIR}/pcre16_ucd.c
  ${PCRE_SOURCE_DIR}/pcre16_utf16_utils.c
  ${PCRE_SOURCE_DIR}/pcre16_valid_utf16.c
  ${PCRE_SOURCE_DIR}/pcre16_version.c
  ${PCRE_SOURCE_DIR}/pcre16_xclass.c
)
ENDIF(PCRE_BUILD_PCRE16)

IF(PCRE_BUILD_PCRE32)
SET(PCRE32_SOURCES
  ${PCRE_SOURCE_DIR}/pcre32_byte_order.c
  ${PCRE_SOURCE_DIR}/pcre32_chartables.c
  ${PCRE_SOURCE_DIR}/pcre32_compile.c
  ${PCRE_SOURCE_DIR}/pcre32_config.c
  ${PCRE_SOURCE_DIR}/pcre32_dfa_exec.c
  ${PCRE_SOURCE_DIR}/pcre32_exec.c
  ${PCRE_SOURCE_DIR}/pcre32_fullinfo.c
  ${PCRE_SOURCE_DIR}/pcre32_get.c
  ${PCRE_SOURCE_DIR}/pcre32_globals.c
  ${PCRE_SOURCE_DIR}/pcre32_jit_compile.c
  ${PCRE_SOURCE_DIR}/pcre32_maketables.c
  ${PCRE_SOURCE_DIR}/pcre32_newline.c
  ${PCRE_SOURCE_DIR}/pcre32_ord2utf32.c
  ${PCRE_SOURCE_DIR}/pcre32_refcount.c
  ${PCRE_SOURCE_DIR}/pcre32_string_utils.c
  ${PCRE_SOURCE_DIR}/pcre32_study.c
  ${PCRE_SOURCE_DIR}/pcre32_tables.c
  ${PCRE_SOURCE_DIR}/pcre32_ucd.c
  ${PCRE_SOURCE_DIR}/pcre32_utf32_utils.c
  ${PCRE_SOURCE_DIR}/pcre32_valid_utf32.c
  ${PCRE_SOURCE_DIR}/pcre32_version.c
  ${PCRE_SOURCE_DIR}/pcre32_xclass.c
)
ENDIF(PCRE_BUILD_PCRE32)

SET(PCRECPP_HEADERS
  ${PCRE_SOURCE_DIR}/pcrecpp.h
  ${PCRE_SOURCE_DIR}/pcre_scanner.h
  ${PROJECT_BINARY_DIR}/pcrecpparg.h
  ${PROJECT_BINARY_DIR}/pcre_stringpiece.h
)

SET(PCRECPP_SOURCES
        ${PCRE_SOURCE_DIR}/pcrecpp.cc
        ${PCRE_SOURCE_DIR}/pcre_scanner.cc
        ${PCRE_SOURCE_DIR}/pcre_stringpiece.cc
)

# Build setup

ADD_DEFINITIONS(-DHAVE_CONFIG_H)

IF(MSVC)
        ADD_DEFINITIONS(-D_CRT_SECURE_NO_DEPRECATE -D_CRT_SECURE_NO_WARNINGS)
ENDIF(MSVC)

SET(CMAKE_INCLUDE_CURRENT_DIR 1)
# needed to make sure to not link debug libs
# against release libs and vice versa
IF(WIN32)
  SET(CMAKE_DEBUG_POSTFIX "d")
ENDIF(WIN32)

SET(targets)

# Libraries
# pcre

ADD_LIBRARY(pcre STATIC ${PCRE_HEADERS} ${PCRE_SOURCES} ${PROJECT_BINARY_DIR}/config.h)

# end CMakeLists.txt
