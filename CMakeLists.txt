cmake_minimum_required(VERSION 3.10)
project("libepoxy")

find_package(PythonInterp 2.7.0 REQUIRED)
if (PYTHON_VERSION_MAJOR GREATER 2)
    message(FATAL_ERROR "Python 2 is required, but Python ${PYTHON_VERSION_MAJOR} was found.")
endif ()

set(EPOXY_LIB_TYPE "STATIC")

if (UNIX)
  if (EPOXY_LIB_TYPE MATCHES "SHARED")
    set(EPOXY_PUBLIC "__attribute__((visibility(\"default\"))) extern")
  else ()
    set(EPOXY_PUBLIC)
  endif ()
  set(inline "inline")
elseif (MSVC)
  if (EPOXY_LIB_TYPE MATCHES "SHARED")
    set(EPOXY_PUBLIC "__declspec(dllexport) extern")
  else ()
    set(EPOXY_PUBLIC)
  endif ()
  set(inline "__inline")
endif ()

set(ENABLE_EGL 1)
set(ENABLE_GLX 0)
set(ENABLE_WGL 1)

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