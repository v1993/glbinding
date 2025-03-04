
# 
# External dependencies
# 

find_package(Qt5Core    5.1 QUIET)
find_package(Qt5Gui     5.1 QUIET)
find_package(Qt5Widgets 5.1 QUIET)
find_package(cpplocate      QUIET)


# Enable automoc
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(AUTOMOC_MOC_OPTIONS PROPERTIES FOLDER CMakeAutomocTargets)
set_property(GLOBAL PROPERTY AUTOMOC_FOLDER CMakeAutomocTargets)

# ENABLE CMP0020: Automatically link Qt executables to qtmain target on Windows.
cmake_policy(SET CMP0020 NEW)


# 
# Library name and options
# 

# Target name
set(target cubescape-qt)

# Exit here if required dependencies are not met
if (NOT Qt5Core_FOUND)
    message(STATUS "Example ${target} skipped: Qt5 not found")
    return()
endif()

# On macOS qt5 fails to link against opengl
if(APPLE)
    find_package(OpenGL     REQUIRED)
endif()


if (NOT cpplocate_FOUND)
    message(STATUS "Example ${target}: using static data path (cpplocate not found)")
else()
    message(STATUS "Example ${target}")
endif()


# 
# Sources
# 

set(cubscape_path ../cubescape)

set(sources
    main.cpp

    Canvas.cpp
    Canvas.h
    Painter.cpp
    Painter.h
    Viewer.cpp
    Viewer.h
    Viewer.ui

    ${cubscape_path}/CubeScape.cpp
    ${cubscape_path}/CubeScape.h
    ${cubscape_path}/glutils.cpp
    ${cubscape_path}/glutils.h
    ${cubscape_path}/RawFile.cpp
    ${cubscape_path}/RawFile.h
)


# 
# Create executable
# 

# Build executable
add_executable(${target}
    MACOSX_BUNDLE
    ${sources}
)

# Create namespaced alias
add_executable(${META_PROJECT_NAME}::${target} ALIAS ${target})


# 
# Project options
# 

set_target_properties(${target}
    PROPERTIES
    ${DEFAULT_PROJECT_OPTIONS}
    INSTALL_RPATH "${EXECUTABLE_INSTALL_RPATH}"
    FOLDER "${IDE_FOLDER}"
)


# 
# Include directories
# 

target_include_directories(${target}
    PRIVATE
    ${DEFAULT_INCLUDE_DIRECTORIES}
    ${PROJECT_BINARY_DIR}/source/include
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR} # for UIC generated headers
)


# 
# Libraries
# 

target_link_libraries(${target}
    PRIVATE
    ${DEFAULT_LIBRARIES}
    ${META_PROJECT_NAME}::glbinding
    ${META_PROJECT_NAME}::glbinding-aux
    Qt5::Core
    Qt5::Gui
    Qt5::Widgets
    "$<$<BOOL:${cpplocate_FOUND}>:cpplocate::cpplocate>"
    "$<$<PLATFORM_ID:Darwin>:${OPENGL_gl_LIBRARY}>"
)


# 
# Compile definitions
# 

target_compile_definitions(${target}
    PRIVATE
    ${DEFAULT_COMPILE_DEFINITIONS}
    $<$<BOOL:${cpplocate_FOUND}>:cpplocate_FOUND>
)


# 
# Compile options
# 

target_compile_options(${target}
    PRIVATE
    ${DEFAULT_COMPILE_OPTIONS_PRIVATE}
    PUBLIC
    ${DEFAULT_COMPILE_OPTIONS_PUBLIC}
)


# 
# Linker options
# 

target_link_libraries(${target}
    PRIVATE
    ${DEFAULT_LINKER_OPTIONS}
)


#
# Target Health
#

perform_health_checks(
    ${target}
    ${sources}
)


# 
# Deployment
# 

# Executable
install(TARGETS ${target}
    RUNTIME DESTINATION ${INSTALL_EXAMPLES} COMPONENT examples_qt
    BUNDLE  DESTINATION ${INSTALL_EXAMPLES} COMPONENT examples_qt
)
