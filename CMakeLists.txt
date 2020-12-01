########################################################
#   SAMTOOLS-cmake
#

cmake_minimum_required(VERSION 3.16)

set(SAMTOOLS_VERSION_MAJOR 1)
set(SAMTOOLS_VERSION_MINOR 11)
set(SAMTOOLS_VERSION_MICRO 0)
set(VERSION "${SAMTOOLS_VERSION_MAJOR}.${SAMTOOLS_VERSION_MINOR}.${SAMTOOLS_VERSION_MICRO}")

project(samtools-cmake VERSION ${VERSION})

set(CMAKE_CXX_STANDARD 17)


include(CheckSymbolExists)
include(CheckCXXSymbolExists)
include(CheckLibraryExists)
include(CheckIncludeFile)
include(CheckIncludeFileCXX)
include(CheckFunctionExists)
include(CheckTypeSize)
include(GNUInstallDirs)


set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_VISIBILITY_INLINES_HIDDEN ON)
set(CMAKE_VERBOSE_MAKEFILE ON)

# put all runtime targets (exe and dll) into bin
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/bin)

# put libraries into lib
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/lib)

# archives
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/lib)

if (MSVC)
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS}" "/NODEFAULTLIB")
endif ()

########################################################
#   dependencies
#

add_subdirectory(third_party)

find_package(ZLIB REQUIRED)
find_package(BZip2)
find_package(LibLZMA)
find_package(CURL)
find_package(Threads)
find_package(OpenSSL)

set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH}" "${CMAKE_CURRENT_SOURCE_DIR}/cmake")


find_path(
        HTS_INSTALL_PREFIX
        NAMES
            lib/${CMAKE_SHARED_LIBRARY_PREFIX}hts${CMAKE_SHARED_LIBRARY_SUFFIX}
            lib/${CMAKE_SHARED_LIBRARY_PREFIX}htslib${CMAKE_SHARED_LIBRARY_SUFFIX}
            include/htslib/hts.h
        PATHS
            "${CMAKE_CURRENT_SOURCE_DIR}"
            "${OneDirectoryAboveWhereWeAre}"
            "${DIRECTORIES_IN_POTENTIAL_HTSLIB_LOCATION}"
            "${HTSLIB_INSTALL_PREFIX}" # alias
)
if (NOT HTS_INSTALL_PREFIX)
    message(WARNING "Could not automatically locate\
 path to htslib. Please provide the path to the install\
 tree for htslib to
-DHTS_INSTALL_PREFIX=/path/to/htslib/install/prefix
 If you add this directory to PATH or install\
 htslib in a system directory (/usr/local or C:/Program Files) it will\
be found automatically. ")
endif ()

message(STATUS "${HTS_INSTALL_PREFIX}/lib/cmake/hts")
set(CMAKE_PREFIX_PATH "${CMAKE_PREFIX_PATH}" "${HTS_INSTALL_PREFIX}/lib/cmake/hts")


find_package(HTS CONFIG REQUIRED)


#/* Define to 1 if a SysV or X/Open compatible Curses library is present */
#/* Define to 1 if <curses.h> is present */
check_include_file(curses.h HAVE_CURSES)

check_include_file(ncurses.h HAVE_NCURSES)

#/* Define to 1 if library supports color (enhanced functions) */
set(CMAKE_REQUIRED_INCLUDES "${CMAKE_REQUIRED_INCLUDES}" "curses.h")
check_function_exists(start_color HAVE_CURSES_COLOR)

#/* Define to 1 if library supports X/Open Enhanced functions */
set(HAVE_CURSES_ENHANCED)

#/* Define to 1 if library supports certain obsolete features */
set(HAVE_CURSES_OBSOLETE)

#/* Define to 1 if you have the `z' library (-lz). */
set(HAVE_LIBZ ${ZLIB_FOUND})

#/* Define to 1 if the Ncurses library is present */
find_library(
        NCURSES_LIBRARY
        NAMES
        ${CMAKE_SHARED_LIBRARY_PREFIX}ncurses${CMAKE_SHARED_LIBRARY_SUFFIX}
)
set(HAVE_NCURSES ${NCURSES_LIBRARY})

#/* Define to 1 if the NcursesW library is present */
find_library(
        NCURSESW_LIBRARY
        NAMES
        ${CMAKE_SHARED_LIBRARY_PREFIX}ncursesw${CMAKE_SHARED_LIBRARY_SUFFIX}
)
set(HAVE_NCURSESW ${NCURSESW_LIBRARY})


#/* Define to 1 if <ncursesw/curses.h> is present */
set(HAVE_NCURSESW_CURSES_H)
check_include_file(ncursesw/curses.h HAVE_NCURSESW_CURSES_H)

#/* Define to 1 if <ncursesw.h> is present */
set(HAVE_NCURSESW_H)
check_include_file(ncursew.h HAVE_NCURSESW_H)

#/* Define to 1 if <ncurses/curses.h> is present */
check_include_file(ncurses/curses.h HAVE_NCURSES_CURSES_H)

#/* Define to 1 if <ncurses.h> is present */
set(HAVE_NCURSES_H)
check_include_file(ncurses.h HAVE_CURSES_H)

#/* Define to 1 if <regex.h> is present */
check_include_file(regex.h HAVE_REGEX_H)

#/* Define to the address where bug reports for this package should be sent. */
set(PACKAGE_BUGREPORT)

#/* Define to the full name of this package. */
set(PACKAGE_NAME "${PROJECT_NAME}")

#/* Define to the full name and version of this package. */
set(PACKAGE_STRING "${PROJECT_NAME}-${PROJECT_VERSION}")

#/* Define to the one symbol short name of this package. */
set(PACKAGE_TARNAME)

#/* Define to the home page for this package. */
set(PACKAGE_URL)

#/* Define to the version of this package. */
set(PACKAGE_VERSION "${PROJECT_VERSION}")

#/* Number of bits in a file offset, on hosts where this is settable. */
set(_FILE_OFFSET_BITS)

#/* Define for large files, on AIX-style hosts. */
set(_LARGE_FILES)

configure_file(
        ${CMAKE_CURRENT_SOURCE_DIR}/config.h.in
        ${CMAKE_CURRENT_BINARY_DIR}/config.h
)

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/version.h"
        "#define SAMTOOLS_VERSION \"${PROJECT_VERSION}\""
        )

###################################################################
#   Sources
#

set(
        SOURCES
        "${CMAKE_CURRENT_SOURCE_DIR}/bam.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam2bcf.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_ampliconclip.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_endian.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_lpileup.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_plbuf.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_tview.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/bedidx.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/sam.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/sam_opts.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/sample.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/samtools.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/stats_isize.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/tmp_file.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/amplicon_stats.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam2bcf.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam2bcf_indel.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam2depth.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_addrprg.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_ampliconclip.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_aux.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_cat.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_color.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_fastq.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_flags.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_index.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_lpileup.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_markdup.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_mate.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_md.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_plbuf.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_plcmd.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_quickcheck.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_reheader.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_rmdup.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_rmdupse.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_sort.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_split.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_stat.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_tview.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_tview_curses.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bam_tview_html.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bamshuf.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bamtk.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bedcov.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/bedidx.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/coverage.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/cut_target.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/dict.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/faidx.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/padding.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/phase.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/sam.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/sam_opts.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/sam_utils.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/sam_view.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/sample.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/stats.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/stats_isize.c"
        "${CMAKE_CURRENT_SOURCE_DIR}/tmp_file.c"

)


set(target samtools)

set(
        INCLUDE_DIRECTORIES
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>"
)

set(LINK_LIBRARIES HTS::hts linux2win)

# build static library
add_library(${target}-static STATIC ${SOURCES})
target_link_libraries(${target}-static PUBLIC ${LINK_LIBRARIES})
target_include_directories(${target}-static PUBLIC ${INCLUDE_DIRECTORIES})
target_compile_definitions(${target}-static PUBLIC SAMTOOLS_STATIC_DEFINE)

#set_target_properties(${target}-static PROPERTIES CXX_STANDARD 17)

# build shared library
add_library(${target} SHARED ${SOURCES})
target_link_libraries(${target} PUBLIC ${LINK_LIBRARIES})
target_include_directories(${target} PUBLIC ${INCLUDE_DIRECTORIES})
target_compile_definitions(${target} PRIVATE SAMTOOLS_EXPORT)
#set_target_properties(${target} PROPERTIES CXX_STANDARD 17)

include(GenerateExportHeader)
generate_export_header(${target})

















