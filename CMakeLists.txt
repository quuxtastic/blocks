cmake_minimum_required(VERSION 2.8)

project(blocks)

if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to Debug")
    set(CMAKE_BUILD_TYPE Debug CACHE STRING "Set build type" FORCE)
endif(NOT CMAKE_BUILD_TYPE)
if(NOT CMAKE_CONFIGURATION_TYPES)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
        "Debug"
        "Release"
        "RelWithDebInfo"
    )
endif(NOT CMAKE_CONFIGURATION_TYPES)

## build settings

add_definitions(-std=c++0x)

set(BLOCKS_BOOST_COMPONENTS regex thread date_time system python)

if(WIN32 OR APPLE)
    set(Boost_USE_STATIC_LIBS TRUE)
else(WIN32 OR APPLE)
    set(Boost_USE_STATIC_LIBS ${OGRE_STATIC})
endif(WIN32 OR APPLE)
if(MINGW)
    set(CMAKE_FIND_LIBRARY_PREFIXES ${CMAKE_FIND_LIBRARY_PREFIXES} "")
endif(MINGW)

find_package(Boost COMPONENTS ${BLOCKS_BOOST_COMPONENTS} REQUIRED)
if(NOT Boost_FOUND)
    set(Boost_USE_STATIC_LIBS NOT ${Boost_USE_STATIC_LIBS})
    find_package(Boost COMPONENTS ${BLOCKS_BOOST_COMPONENTS} REQUIRED)
endif(NOT Boost_FOUND)
find_package(Boost REQUIRED)

add_definitions(-DBOOST_ALL_NO_LIB)

if(WIN32)
    set(CMAKE_MODULE_PATH "$ENV{OGRE_HOME}/CMake/;${CMAKE_MODULE_PATH}")
endif(WIN32)
if(UNIX)
    if(EXISTS "/usr/local/lib/OGRE/cmake")
        set(CMAKE_MODULE_PATH "/usr/local/lib/OGRE/cmake/;${CMAKE_MODULE_PATH}")
    elseif(EXISTS "/usr/lib/OGRE/cmake")
        set(CMAKE_MODULE_PATH "/usr/lib/OGRE/cmake/;${CMAKE_MODULE_PATH}")
    endif(EXISTS "/usr/local/lib/OGRE/cmake")
endif(UNIX)
find_package(OGRE REQUIRED)

# OIS' cmake file is included with OGRE. It must be set up /after/ the OGRE
# cmake package has been found
find_package(OIS REQUIRED)

if(WIN32)
    set(CMAKE_MODULE_PATH "$ENV{PolyVox_HOME}/CMake/;${CMAKE_MODULE_PATH}")
endif(WIN32)
if(UNIX)
    set(CMAKE_MODULE_PATH "/usr/local/share/PolyVox/cmake/;${CMAKE_MODULE_PATH}")
endif(UNIX)
find_package(PolyVox)

## source files

set(HEADERS
    ./BaseApplication.h
    ./TutorialApplication.h
)

set(SOURCES
    ./BaseApplication.cpp
    ./TutorialApplication.cpp
)

## build targets

include_directories(
    ${Boost_INCLUDE_DIR}
    ${OGRE_INCLUDE_DIRS}
    ${OGRE_Overlay_INCLUDE_DIRS}
    ${OIS_INCLUDE_DIRS}
    ${PolyVox_INCLUDE_DIRS}
)

if(WIN32)
    add_executable(blocks WIN32 ${HEADERS} ${SOURCES})
endif(WIN32)
if(UNIX)
    add_executable(blocks ${HEADERS} ${SOURCES})
endif(UNIX)

set_target_properties(blocks PROPERTIES DEBUG_POSTFIX _d)
target_link_libraries(blocks
    ${Boost_LIBRARIES}
    ${OGRE_LIBRARIES}
    ${OIS_LIBRARIES}
    ${OGRE_Overlay_LIBRARIES}
    ${PolyVox_LIBRARIES}
)

file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/dist)
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/dist/media)

if(WIN32 AND NOT MINGW)
    add_custom_command(TARGET blocks POST_BUILD
        COMMANd copy \"$(TargetPath)\" .\\dist
    )
endif(WIN32 AND NOT MINGW)
if(MINGW OR UNIX)
    set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/dist)
endif(MINGW OR UNIX)

if(WIN32)
    install(TARGETS blocks
        RUNTIME DESTINATION ./
        CONFIGURATIONS All
    )

    install(DIRECTORY ${CMAKE_SOURCE_DIR}/dist/media
        DESTINATION ${PROJECT_BINARY_DIR}/dist
        CONFIGURATIONS Release RelWithDebInfo Debug
    )

    install(FILES
        ${CMAKE_SOURCE_DIR}/dist/bin/plugins.cfg
        ${CMAKE_SOURCE_DIR}/dist/bin/resources.cfg
        DESTINATION ${PROJECT_BINARY_DIR}/dist
        CONFIGURATIONS Release RelWithDebInfo
    )
    install(FILES
        ${CMAKE_SOURCE_DIR}/dist/bin/plugins_d.cfg
        ${CMAKE_SOURCE_DIR}/dist/bin/resources_d.cfg
        DESTINATION ${PROJECT_BINARY_DIR}/dist
        CONFIGURATIONS Debug
    )

    install(FILES
        ${OGRE_PLUGIN_DIR_REL}/RenderSystem_Direct3D9.dll
        ${OGRE_PLUGIN_DIR_REL}/RenderSystem_GL.dll
        ${Boost_DATE_TIME_LIBRARY_RELEASE}
        ${Boost_THREAD_LIBRARY_RELEASE}
        ${Boost_SYSTEM_LIBRARY_RELEASE}
        ${Boost_ASIO_LIBRARY_RELEASE}
        ${OGRE_PLUGIN_DIR_REL}/OgreMain.dll
        ${OGRE_PLUGIN_DIR_REL}/libOIS.dll
        DESTINATION ${PROJECT_BINARY_DIR}/dist
        CONFIGURATIONS Release RelWithDebInfo
    )
    install(FILES
        ${OGRE_PLUGIN_DIR_DBG}/RenderSystem_Direct3D9_d.dll
        ${OGRE_PLUGIN_DIR_DBG}/RenderSystem_GL_d.dll
        ${Boost_DATE_TIME_LIBRARY_DEBUG}
        ${Boost_THREAD_LIBRARY_DEBUG}
        ${Boost_SYSTEM_LIBRARY_DEBUG}
        ${Boost_ASIO_LIBRARY_DEBUG}
        ${OGRE_PLUGIN_DIR_DBG}/OgreMain_d.dll
        ${OGRE_PLUGIN_DIR_DBG}/libOIS_d.dll
        DESTINATION ${PROJECT_BINARY_DIR}/dist
        CONFIGURATIONS Debug
    )
endif(WIN32)
if(UNIX)
    install(TARGETS blocks
        RUNTIME DESTINATION ./
        CONFIGURATIONS All
    )

    install(DIRECTORY ${CMAKE_SOURCE_DIR}/dist/media
        DESTINATION ${PROJECT_BINARY_DIR}/dist
        CONFIGURATIONS Release RelWithDebInfo Debug
    )

    install(FILES
        ${CMAKE_SOURCE_DIR}/dist/plugins.cfg
        ${CMAKE_SOURCE_DIR}/dist/resources.cfg
        DESTINATION ${PROJECT_BINARY_DIR}/dist
        CONFIGURATIONS Release RelWithDebInfo Debug
    )

endif(UNIX)
