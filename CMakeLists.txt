cmake_minimum_required(VERSION 3.14)

project(dos2unix LANGUAGES C
VERSION 7.4.3)

set(DOS2UNIX_DATE "2022-06-05")
option(UCS "Unicode capable" true)
option(LFS "Large file support" true)
option(ENABLE_NLS "native language support" true)

include(GNUInstallDirs)
include(FetchContent)

set(CMAKE_TLS_VERIFY true)

set(url https://git.code.sf.net/p/dos2unix/dos2unix.git)
set(tag dos2unix-7.4.3)

FetchContent_Declare(dos2unix
GIT_REPOSITORY ${url}
GIT_TAG ${tag}
GIT_SHALLOW true
INACTIVITY_TIMEOUT 60
)

FetchContent_Populate(dos2unix)

set(src ${dos2unix_SOURCE_DIR}/dos2unix)

add_compile_options(
"$<$<C_COMPILER_ID:GNU>:-Wall;-Wextra;-Wconversion>"
)

if(ENABLE_NLS)
  find_library(INTL NAMES intl REQUIRED)
endif()

# if(MINGW)
#   find_file(CRTglob
#   NAMES CRT_glob.o
#   PATH_SUFFIXES lib
#   REQUIRED
#   )
# endif()

add_library(common OBJECT ${src}/common.c)
target_compile_definitions(common PUBLIC
VER_REVISION=\"${PROJECT_VERSION}\"
VER_DATE=\"${DOS2UNIX_DATE}\"
$<$<BOOL:${MINGW}>:D2U_UNIFILE>
$<$<BOOL:${UCS}>:D2U_UNICODE>
"$<$<BOOL:${LFS}>:_LARGEFILE_SOURCE;_FILE_OFFSET_BITS=64>"
"$<$<BOOL:${ENABLE_NLS}>:ENABLE_NLS;LOCALEDIR=\"${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LOCALEDIR}\">"
)
target_link_libraries(common PRIVATE
$<$<BOOL:${ENABLE_NLS}>:${INTL}>
)

add_library(querycp OBJECT ${src}/querycp.c)

add_executable(dos2unix ${src}/dos2unix.c)
target_link_libraries(dos2unix PRIVATE common querycp)
target_include_directories(dos2unix PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
target_compile_definitions(dos2unix PRIVATE
$<$<BOOL:${ENABLE_NLS}>:PACKAGE=\"dos2unix\">
)

add_executable(unix2dos ${src}/unix2dos.c)
target_link_libraries(unix2dos PRIVATE common querycp

)
target_include_directories(unix2dos PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
target_compile_definitions(unix2dos PRIVATE
$<$<BOOL:${ENABLE_NLS}>:PACKAGE=\"unix2dos\">
)

install(TARGETS dos2unix unix2dos)
