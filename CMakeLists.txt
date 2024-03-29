cmake_minimum_required(VERSION 3.5)

project(QInput VERSION 0.1 LANGUAGES CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_CXX_COMPILER "g++")

set(CMAKE_VERBOSE_MAKEFILE ON)

enable_language(C)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets)

if (WIN32 OR WIN64)
    set(UIOHOOK_SOURCE_DIR "windows")
elseif (APPLE)
    set(UIOHOOK_SOURCE_DIR "darwin")
else()
    set(UIOHOOK_SOURCE_DIR "x11")
endif()

add_library(libs
    "src/logger.c"
    "src/${UIOHOOK_SOURCE_DIR}/input_helper.c"
    "src/${UIOHOOK_SOURCE_DIR}/input_hook.c"
    "src/${UIOHOOK_SOURCE_DIR}/post_event.c"
    "src/${UIOHOOK_SOURCE_DIR}/system_properties.c"
)

set(PROJECT_SOURCES
        src/hook.cpp
        src/main.cpp
        src/mainwindow.cpp
        src/mainwindow.ui
        include/hook.h
        include/mainwindow.h
)

if (${CMAKE_SYSTEM_NAME} MATCHES "Windows")
    enable_language("RC")
    set (WIN32_RESOURCES ${CMAKE_CURRENT_SOURCE_DIR}/resources/QInput.rc)
endif()

if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(QInput
        MANUAL_FINALIZATION
        ${PROJECT_SOURCES}
        ${WIN32_RESOURCES}
    )
# Define target properties for Android with Qt 6 as:
#    set_property(TARGET QInput APPEND PROPERTY QT_ANDROID_PACKAGE_SOURCE_DIR
#                 ${CMAKE_CURRENT_SOURCE_DIR}/android)
# For more information, see https://doc.qt.io/qt-6/qt-add-executable.html#target-creation
else()
    if(ANDROID)
        add_library(QInput SHARED
            ${PROJECT_SOURCES}
        )
# Define properties for Android with Qt 5 after find_package() calls as:
#    set(ANDROID_PACKAGE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/android")
    else()
        add_executable(QInput
            ${PROJECT_SOURCES}
        )
    endif()
endif()

target_link_libraries(QInput PRIVATE Qt${QT_VERSION_MAJOR}::Widgets)

#Include directories
set(INCLUDE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_include_directories(QInput PUBLIC ${INCLUDE_DIRECTORY})

set(SOURCES src/hook.cpp ${INCLUDE_DIRECTORY}/uiohook.h ${INCLUDE_DIRECTORY}/hook.h)

if (UNIX)
    #Wl rpath
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wl,-rpath, ${CMAKE_CURRENT_SOURCE_DIR}/lib/libuiohook.so.1.2.0")
    #add_executable(test ${SOURCES})
endif()

#Copy library files
file (COPY ${CMAKE_CURRENT_SOURCE_DIR}/lib/libuiohook.so.1.2.0 DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file (COPY ${CMAKE_CURRENT_SOURCE_DIR}/lib/libuiohook.so.1 DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file (COPY ${CMAKE_CURRENT_SOURCE_DIR}/lib/libuiohook.so DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

#target_include_directories(test PRIVATE ${INCLUDE_DIRECTORY})
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,-rpath='$ORIGIN'")

#Directories
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/resources DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

if(WIN32)
    target_link_libraries(QInput PRIVATE Advapi32)
endif()

target_link_libraries(QInput PRIVATE libs)


set_target_properties(QInput PROPERTIES
    MACOSX_BUNDLE_GUI_IDENTIFIER my.example.com
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    MACOSX_BUNDLE TRUE
    WIN32_EXECUTABLE TRUE
)


if(QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(QInput)
endif()
