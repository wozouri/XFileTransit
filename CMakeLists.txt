cmake_minimum_required(VERSION 3.16)
project(XFileTransit LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_compile_definitions(WIN32_LEAN_AND_MEAN)
if(MSVC)
    add_compile_options("/utf-8")
endif()

find_package(Qt5 REQUIRED COMPONENTS Core Quick Widgets Gui Network QuickControls2)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

add_subdirectory(src)
