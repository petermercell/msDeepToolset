cmake_minimum_required(VERSION 3.11)

project(msDeepTools)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ============================================================================
# RPATH Configuration - Make plugins portable
# ============================================================================
# Don't set RPATH - rely on Nuke's LD_LIBRARY_PATH environment
# This makes the plugins work regardless of where Nuke is installed
set(CMAKE_SKIP_BUILD_RPATH TRUE)
set(CMAKE_SKIP_INSTALL_RPATH TRUE)

# Set Nuke version and directory (allow override via -DNUKE_VERSION)
if(NOT DEFINED NUKE_VERSION)
    set(NUKE_VERSION "17.0v1")
endif()
set(NDKDIR /opt/Nuke${NUKE_VERSION})

message(STATUS "Nuke Version: ${NUKE_VERSION}")
message(STATUS "Nuke Directory: ${NDKDIR}")

# Verify Nuke directory exists
if(NOT EXISTS ${NDKDIR})
    message(FATAL_ERROR "Nuke directory not found: ${NDKDIR}")
endif()
if(NOT EXISTS ${NDKDIR}/libDDImage.so)
    message(FATAL_ERROR "libDDImage.so not found in: ${NDKDIR}")
endif()

# Uncomment the following if needed for compatibility
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_GLIBCXX_USE_CXX11_ABI=0")
add_compile_options(-fPIC)

# Strip the "lib" prefix so output is e.g. msDeepBlur.so not libmsDeepBlur.so
set(CMAKE_SHARED_LIBRARY_PREFIX "")

# ============================================================================
# Helper function to create a Nuke plugin target
# ============================================================================
function(add_nuke_plugin PLUGIN_NAME SOURCE_FILE)
    add_library(${PLUGIN_NAME} SHARED src/${SOURCE_FILE})

    target_include_directories(${PLUGIN_NAME} PRIVATE
        ${CMAKE_SOURCE_DIR}/include
        ${NDKDIR}/include
    )

    target_compile_definitions(${PLUGIN_NAME} PRIVATE
        _USE_MATH_DEFINES
    )

    target_link_libraries(${PLUGIN_NAME} PRIVATE
        ${NDKDIR}/libDDImage.so
        OpenGL
    )

    install(TARGETS ${PLUGIN_NAME} DESTINATION ${CMAKE_INSTALL_PREFIX})
endfunction()

# ============================================================================
# Build all plugins
# ============================================================================
add_nuke_plugin(msDeepBlur      msDeepBlur.cpp)
add_nuke_plugin(msDeepKeymix    msDeepKeymix.cpp)
add_nuke_plugin(msDeepReformat  msDeepReformat.cpp)

# ============================================================================
# Build Configuration Summary
# ============================================================================
message(STATUS "========================================")
message(STATUS "Build Configuration Summary:")
message(STATUS "  Plugins: msDeepBlur.so, msDeepKeymix.so, msDeepReformat.so")
message(STATUS "  Nuke Version: ${NUKE_VERSION}")
message(STATUS "  Nuke Directory: ${NDKDIR}")
message(STATUS "  RPATH: Not set (portable - uses Nuke's environment)")
message(STATUS "  Install Prefix: ${CMAKE_INSTALL_PREFIX}")
message(STATUS "========================================")
