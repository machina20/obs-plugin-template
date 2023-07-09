cmake_minimum_required(VERSION 3.16...3.26)

include("${CMAKE_CURRENT_SOURCE_DIR}/cmake/common/bootstrap.cmake" NO_POLICY_SCOPE)  #include the other cmake file

project(${_name} VERSION ${_version})  #specifying the project name using _name and _version variables from elsewhere

option(ENABLE_FRONTEND_API "Use obs-frontend-api for UI functionality" OFF) #setting custom var for enabling frontend api
option(ENABLE_QT "Use Qt functionality" OFF) # setting custom var for use Qt functionality

include(compilerconfig)
include(defaults)
include(helpers)

add_library(${CMAKE_PROJECT_NAME} MODULE) #adds a library of type module: dynamically loadable module.

find_package(libobs REQUIRED)
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE OBS::libobs)

if(ENABLE_FRONTEND_API)
  find_package(obs-frontend-api REQUIRED)
  target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE OBS::obs-frontend-api)
endif()

if(ENABLE_QT)
  find_qt(COMPONENTS Widgets Core)
  target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE Qt::Core Qt::Widgets)
  target_compile_options(
    ${CMAKE_PROJECT_NAME} PRIVATE $<$<C_COMPILER_ID:Clang,AppleClang>:-Wno-quoted-include-in-framework-header
                                  -Wno-comma>)
  set_target_properties(
    ${CMAKE_PROJECT_NAME}
    PROPERTIES AUTOMOC ON
               AUTOUIC ON
               AUTORCC ON)
endif()

target_sources(${CMAKE_PROJECT_NAME} PRIVATE src/plugin-main.c)

set_target_properties_plugin(${CMAKE_PROJECT_NAME} PROPERTIES OUTPUT_NAME ${_name})

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC -g")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -g")