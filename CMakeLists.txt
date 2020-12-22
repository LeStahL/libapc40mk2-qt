cmake_minimum_required(VERSION 3.14)

project(libapc40mk2-qt)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOMOC ON)

set(CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH} "C:\\Qt\\5.15.0\\msvc2019_64\\")

find_package(Qt5 COMPONENTS Core Widgets)
find_package(Git QUIET)

# Add QMidi submodule
if(${GIT_FOUND} AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
# Update submodules as needed
    option(GIT_SUBMODULE "Check submodules during build" ON)
    if(GIT_SUBMODULE)
        message(STATUS "Submodule update")
        execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
                        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                        RESULT_VARIABLE GIT_SUBMOD_RESULT)
        if(NOT GIT_SUBMOD_RESULT EQUAL "0")
            message(FATAL_ERROR "git submodule update --init failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
        endif()
    endif()
endif()

if(NOT EXISTS "${PROJECT_SOURCE_DIR}/thirdparty/QMidi/CMakeLists.txt")
    message(FATAL_ERROR "The submodules were not downloaded! GIT_SUBMODULE was turned off or failed. Please update submodules and try again.")
endif()

add_subdirectory(thirdparty/QMidi)

# # Make sure all source files are present to avoid obscure compiler errors.
set(${PROJECT_NAME}_SOURCES
    "${PROJECT_SOURCE_DIR}/src/MidiController.cpp"
    "${PROJECT_SOURCE_DIR}/src/ValueControl.cpp"
    "${PROJECT_SOURCE_DIR}/src/AkaiAPC40MkII.cpp"
)
foreach(FILE ${${PROJECT_NAME}_SOURCES})
    if(NOT EXISTS "${FILE}")
        message(FATAL_ERROR "File ${FILE} does not exist.")
    endif()
endforeach()

# Add library
include_directories(${PROJECT_NAME} "thirdparty/QMidi/src")
add_library(${PROJECT_NAME} ${${PROJECT_NAME}_SOURCES})

set(LIBRARIES
    Qt5::Core
    Qt5::Widgets
)

# Link static and shared version
target_link_libraries(${PROJECT_NAME} ${LIBRARIES})
