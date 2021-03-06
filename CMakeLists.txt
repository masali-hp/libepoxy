cmake_minimum_required(VERSION 3.10)
project("libepoxy")

find_package(PythonInterp 2.7.0 REQUIRED)
if (PYTHON_VERSION_MAJOR GREATER 2)
    message(FATAL_ERROR "Python 2 is required, but Python ${PYTHON_VERSION_MAJOR} was found.")
endif ()

if (NOT DEFINED EPOXY_LIB_TYPE)
    set(EPOXY_LIB_TYPE "STATIC")
endif ()

if (EPOXY_LIB_TYPE MATCHES "SHARED")
    if (MSVC)
        set(EPOXY_PUBLIC "__declspec(dllexport) extern")
    else ()
        set(EPOXY_PUBLIC "__attribute__((visibility(\"default\"))) extern")
    endif ()
elseif (EPOXY_LIB_TYPE MATCHES "STATIC")
    set(EPOXY_PUBLIC)
else ()
    message(FATAL "EPOXY_LIB_TYPE must be STATIC or SHARED")
endif ()

if (MSVC)
    set(inline "__inline")
else ()
    set(inline "inline")
endif ()

if (NOT DEFINED ENABLE_EGL)
    if (WIN32 OR APPLE)
        set(ENABLE_EGL 0)
    else ()
        set(ENABLE_EGL 1)
    endif ()
endif ()

if (NOT DEFINED ENABLE_GLX)
    set(ENABLE_GLX 0)
endif ()

if (NOT DEFINED ENABLE_WGL)
    if (WIN32)
        set(ENABLE_WGL 1)
    else ()
        set(ENABLE_WGL 0)
    endif ()
endif ()

foreach (var "ENABLE_EGL" "ENABLE_GLX" "ENABLE_WGL")
    set (_value ${${var}})
    message(STATUS "${var} is set to ${_value}")
    if (NOT(${_value} EQUAL 0) AND NOT(${_value} EQUAL 1))
        message(FATAL_ERROR "${var} must be set to 0 or 1 (${${var}})")
    endif ()
endforeach ()

if (ENABLE_EGL)
    find_library(EGL_LIBARY NAMES libEGL)
    message(STATUS "Found EGL_LIBARY=${EGL_LIBARY}")
    find_path(EGL_INCLUDE_DIR NAMES EGL/egl.h)
    message(STATUS "Found EGL_INCLUDE_DIR=${EGL_INCLUDE_DIR}")
endif ()

configure_file("cmake-config.h.in" "${CMAKE_CURRENT_BINARY_DIR}/config.h")

set(DISPATCH_GENERATOR ${CMAKE_CURRENT_SOURCE_DIR}/src/gen_dispatch.py)
set(GL_REGISTRY "${CMAKE_CURRENT_SOURCE_DIR}/registry/gl.xml")
set(EGL_REGISTRY "${CMAKE_CURRENT_SOURCE_DIR}/registry/egl.xml")
set(WGL_REGISTRY "${CMAKE_CURRENT_SOURCE_DIR}/registry/wgl.xml")
set(GLX_REGISTRY "${CMAKE_CURRENT_SOURCE_DIR}/registry/glx.xml")

# Append the given dependencies to the source file
macro(ADD_SOURCE_DEPENDENCIES _source _deps)
    set(_tmp)
    get_source_file_property(_tmp ${_source} OBJECT_DEPENDS)
    if (NOT _tmp)
        set(_tmp "")
    endif ()

    foreach (f ${_deps})
        list(APPEND _tmp "${f}")
    endforeach ()

    set_source_files_properties(${_source} PROPERTIES OBJECT_DEPENDS "${_tmp}")
    unset(_tmp)
endmacro()

add_subdirectory("src")

message(STATUS "Install target is ${CMAKE_INSTALL_PREFIX}")

install(DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/include/epoxy" DESTINATION "include")
install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include/epoxy" DESTINATION "include" FILES_MATCHING PATTERN "*.h")