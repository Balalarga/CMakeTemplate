cmake_minimum_required(VERSION 3.20)

project( LANGUAGES CXX)

set(IS_LIBRARY OFF)
set(IS_INTERFACE OFF)

set(USE_MODULE_SYSTEM ON)
set(MODULES_DIR ${CMAKE_CURRENT_SOURCE_DIR}/Source)

set(HAVE_TESTS OFF)


set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)


set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH};${CMAKE_CURRENT_SOURCE_DIR}/CMakeUtils")
include(Utils\FileSystem)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

if(WIN32)
    add_compile_options("$<$<C_COMPILER_ID:MSVC>:/utf-8>")
    add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")
endif()

if(HAVE_TESTS)
    add_subdirectory(Tests)
endif()

if(IS_LIBRARY)
    if (IS_INTERFACE)
        add_library(${PROJECT_NAME})
    else()
        add_library(${PROJECT_NAME} INTERFACE)
    endif()
else()
    add_executable(${PROJECT_NAME})
endif()

if(USE_MODULE_SYSTEM)
    ListOfDirectChildDirectories MODULES_DIR ${MODULES_DIR})
    foreach(MODULE ${MODULES_LIST})
        if(EXISTS ${MODULES_DIR}/${MODULE}/CMakeLists.txt)
            add_subdirectory(${MODULES_DIR}/${MODULE} ${MODULE})
            target_include_directories(${PROJECT_NAME} INTERFACE ${MODULE})
            target_link_libraries(${PROJECT_NAME} INTERFACE ${MODULE})
            get_property(MODULE_TARGET DIRECTORY ${MODULES_DIR}/${MODULE} PROPERTY BUILDSYSTEM_TARGETS)
            set_target_properties(${MODULE_TARGET} PROPERTIES FOLDER ${PROJECT_NAME})
            message("Added submodule ${MODULE_TARGET}(${MODULE})")
        endif()
    endforeach()
else()
    add_subdirectory(Source)
    get_property(MODULE_TARGET DIRECTORY Source PROPERTY BUILDSYSTEM_TARGETS)
    set_target_properties(${MODULE_TARGET} PROPERTIES FOLDER ${PROJECT_NAME})
endif()

set_target_properties(${PROJECT_NAME}
    PROPERTIES
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/Bin"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/Bin"
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/Bin"
)
