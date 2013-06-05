# Settings passed on the command line:
#
# PROJECT_LIBS_DIR = the directory which all external libraries may be referenced from.
# PROJECT_ARCH = the build architecture
# PROJECT_LINK = whether to build a static or dynamic library.

##############################
# General Settings
##############################

cmake_minimum_required (VERSION 2.8.9)
project (boss)

set (BOSS_SRC "${CMAKE_SOURCE_DIR}/src/backend/metadata.cpp" "${CMAKE_SOURCE_DIR}/src/backend/game.cpp" "${CMAKE_SOURCE_DIR}/src/backend/helpers.cpp" "${PROJECT_LIBS_DIR}/pugixml/src/pugixml.cpp")

# Include source and library directories.
include_directories ("${PROJECT_LIBS_DIR}/alphanum" "${PROJECT_LIBS_DIR}/boost" "${PROJECT_LIBS_DIR}/yaml-cpp/include" "${CMAKE_SOURCE_DIR}/src" "${PROJECT_LIBS_DIR}/libloadorder/src" "${PROJECT_LIBS_DIR}/libespm" "${PROJECT_LIBS_DIR}/zlib" "${PROJECT_LIBS_DIR}/pugixml/src" "${PROJECT_LIBS_DIR}/wxWidgets/include")


##############################
# Platform-Specific Settings
##############################

# Settings when compiling for Windows.
IF (CMAKE_SYSTEM_NAME MATCHES "Windows")
    add_definitions (-DUNICODE -D_UNICODE)
    set (BOSS_SRC ${BOSS_SRC} "${CMAKE_SOURCE_DIR}/src/resource.rc")
    IF (${PROJECT_LINK} MATCHES "STATIC")
        add_definitions (-DLIBLO_STATIC)
        add_definitions (-DBOSS_STATIC)
    ELSE ()
        add_definitions (-DLIBLO_STATIC)
        add_definitions (-DBOSS_EXPORT)
    ENDIF ()
ENDIF ()

# Settings when not cross-compiling.
IF (CMAKE_SYSTEM_NAME MATCHES CMAKE_HOST_SYSTEM_NAME)
    link_directories ("${PROJECT_LIBS_DIR}/boost/stage-${PROJECT_ARCH}/lib")
ENDIF ()

link_directories ("${PROJECT_LIBS_DIR}/yaml-cpp/build")
link_directories ("${PROJECT_LIBS_DIR}/libloadorder/build")
link_directories ("${PROJECT_LIBS_DIR}/zlib")
link_directories ("${PROJECT_LIBS_DIR}/wxWidgets/lib")

# Settings when compiling and cross-compiling on Linux.
IF (CMAKE_HOST_SYSTEM_NAME MATCHES "Linux")
    set (CMAKE_C_FLAGS  "-m${PROJECT_ARCH}")
    set (CMAKE_CXX_FLAGS "-m${PROJECT_ARCH}")
    set (CMAKE_EXE_LINKER_FLAGS "-static-libstdc++ -static-libgcc")
    set (CMAKE_SHARED_LINKER_FLAGS "-static-libstdc++ -static-libgcc")
    set (CMAKE_MODULE_LINKER_FLAGS "-static-libstdc++ -static-libgcc")

    IF (CMAKE_SYSTEM_NAME MATCHES "Windows")
        link_directories ("${PROJECT_LIBS_DIR}/boost/stage-mingw-${PROJECT_ARCH}/lib")
        include_directories ("${PROJECT_LIBS_DIR}/wxWidgets/lib/wx/include/i686-w64-mingw32-msw-unicode-static-2.9")
        set (BOSS_LIBS ${BOSS_LIBS} version wx_mswu_webview-2.9-i686-w64-mingw32 wx_mswu_core-2.9-i686-w64-mingw32 wx_baseu-2.9-i686-w64-mingw32 wx_mswu_adv-2.9-i686-w64-mingw32 wxpng-2.9-i686-w64-mingw32 wxzlib-2.9-i686-w64-mingw32 comctl32)
        set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--subsystem,windows")
    ENDIF ()

    set (BOSS_LIBS ${BOSS_LIBS} zlibstatic loadorder yaml-cpp boost_locale boost_filesystem boost_system boost_regex)
ENDIF ()

# Settings when compiling on Windows.
IF (CMAKE_HOST_SYSTEM_NAME MATCHES "Windows")
    set (BOSS_LIBS zlibstatic loadorder yaml-cpp boost_locale-vc110-mt-1_52 libboost_filesystem-vc110-mt-1_52 libboost_system-vc110-mt-1_52)
    set (CMAKE_CXX_FLAGS "/EHsc")
ENDIF ()

##############################
# More General Settings
##############################

set (BOSS_GUI_SRC ${BOSS_SRC} "${CMAKE_SOURCE_DIR}/src/gui/ids.cpp" "${CMAKE_SOURCE_DIR}/src/gui/main.cpp" "${CMAKE_SOURCE_DIR}/src/gui/settings.cpp" "${CMAKE_SOURCE_DIR}/src/gui/editor.cpp" "${CMAKE_SOURCE_DIR}/src/gui/viewer.cpp")

set (BOSS_API_SRC ${BOSS_SRC} "${CMAKE_SOURCE_DIR}/src/api/api.cpp")


##############################
# Actual Building
##############################

# Build API.
add_library           (boss ${PROJECT_LINK} ${BOSS_API_SRC})
target_link_libraries (boss ${BOSS_LIBS})

# Build application.
add_executable        (BOSS ${BOSS_GUI_SRC})
target_link_libraries (BOSS ${BOSS_LIBS})