
macro(GENERATE_DISPATCH_SOURCE _gen_source _registry _source_files)
  set(_gen_output ${CMAKE_CURRENT_BINARY_DIR}/${_gen_source})
  add_custom_command(
    OUTPUT ${_gen_output}
    MAIN_DEPENDENCY ${_registry}
    COMMAND ${PYTHON_EXECUTABLE} ${DISPATCH_GENERATOR} --source --no-header --outputdir=${CMAKE_CURRENT_BINARY_DIR} ${_registry}
    VERBATIM)
    list(APPEND EPOXY_GENERATED_SOURCES ${_gen_output})
    list(APPEND EPOXY_SOURCES ${_source_files})
    set_source_files_properties(${_gen_output} PROPERTIES GENERATED TRUE)
    ADD_SOURCE_DEPENDENCIES(${_registry} ${_gen_output})
endmacro()

set(EPOXY_HEADERS "${CMAKE_CURRENT_SOURCE_DIR}/../include/epoxy/common.h")
set(EPOXY_GENERATED_HEADERS "${CMAKE_CURRENT_BINARY_DIR}/../config.h")

macro(GENERATE_DISPATCH_HEADER _gen_header _registry _header_files)
  set(_output_dir ${CMAKE_CURRENT_BINARY_DIR}/../include/epoxy)
  set(_gen_output ${_output_dir}/${_gen_header})
  add_custom_command(
    OUTPUT ${_gen_output}
    MAIN_DEPENDENCY ${_registry}
    COMMAND ${PYTHON_EXECUTABLE} ${DISPATCH_GENERATOR} --header --no-source --outputdir=${_output_dir} ${_registry}
    VERBATIM)
    list(APPEND EPOXY_GENERATED_HEADERS ${_gen_output})
    list(APPEND EPOXY_HEADERS ${_header_files})
    set_source_files_properties(${_gen_output} PROPERTIES GENERATED TRUE)
    ADD_SOURCE_DEPENDENCIES(${_registry} ${_gen_output})
endmacro()

GENERATE_DISPATCH_SOURCE("gl_generated_dispatch.c" ${GL_REGISTRY} "dispatch_common.c;dispatch_common.h")
GENERATE_DISPATCH_HEADER("gl_generated.h" ${GL_REGISTRY} "${CMAKE_CURRENT_SOURCE_DIR}/../include/epoxy/gl.h")

if (ENABLE_EGL)
  GENERATE_DISPATCH_SOURCE("egl_generated_dispatch.c" ${EGL_REGISTRY} "dispatch_egl.c")
  GENERATE_DISPATCH_HEADER("egl_generated.h" ${EGL_REGISTRY} "${CMAKE_CURRENT_SOURCE_DIR}/../include/epoxy/egl.h")
endif ()
if (ENABLE_WGL)
  GENERATE_DISPATCH_SOURCE("wgl_generated_dispatch.c" ${WGL_REGISTRY} "dispatch_wgl.c")
  GENERATE_DISPATCH_HEADER("wgl_generated.h" ${WGL_REGISTRY} "${CMAKE_CURRENT_SOURCE_DIR}/../include/epoxy/wgl.h")
endif ()
if (ENABLE_GLX)
  GENERATE_DISPATCH_SOURCE("glx_generated_dispatch.c" ${GLX_REGISTRY} "dispatch_glx.c")
  GENERATE_DISPATCH_HEADER("glx_generated.h" ${GLX_REGISTRY} "${CMAKE_CURRENT_SOURCE_DIR}/../include/epoxy/glx.h")
endif ()

include_directories(
  ${CMAKE_CURRENT_BINARY_DIR}/..
  ${CMAKE_CURRENT_BINARY_DIR}/../include
  ${CMAKE_CURRENT_SOURCE_DIR}/../include
  ${CMAKE_CURRENT_SOURCE_DIR}
)

add_library(epoxy ${EPOXY_LIB_TYPE} ${EPOXY_SOURCES} ${EPOXY_GENERATED_SOURCES} ${EPOXY_HEADERS} ${EPOXY_GENERATED_HEADERS})

target_link_libraries(epoxy
    opengl32 gdi32 
)

set_property(DIRECTORY . PROPERTY FOLDER "libepoxy")
