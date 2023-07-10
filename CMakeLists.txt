cmake_minimum_required(VERSION 3.16...3.26) #set the version of the target "VERSION" is used as a keyword to indicate that the following argument is specifying the version number.

include("${CMAKE_CURRENT_SOURCE_DIR}/cmake/common/bootstrap.cmake" NO_POLICY_SCOPE)  #include the other cmake file
#NO_POLICY_SCOPE:it means that the included script's policies will not affect the policies defined in the current CMakeLists.txt file. This is useful when you want to isolate the policy settings of the included script.

project(${_name} VERSION ${_version})  #specifying the project name using _name and _version variables from elsewhere

option(ENABLE_FRONTEND_API "Use obs-frontend-api for UI functionality" OFF) #setting custom var for enabling frontend api
option(ENABLE_QT "Use Qt functionality" OFF) # setting custom var for use Qt functionality

include(compilerconfig)  #these are the names of 3 files within ./cmake/linux/
include(defaults)
include(helpers)

add_library(${CMAKE_PROJECT_NAME} MODULE) #adds a library of type module: dynamically loadable module.  this allows the executable to use it at runtime

find_package(libobs REQUIRED) # locate and configure external libraries or packages. if not found, cmake stops config process 
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE OBS::libobs) #By linking the target with the library, the ${CMAKE_PROJECT_NAME} target will have access to the functionality and symbols provided by the "libobs" library during the build process.
#PRIVATE keyword in the target_link_libraries command restricts the linkage of specified libraries to only the target being linked

if(ENABLE_FRONTEND_API)#links the obs-frontend-api if its needed
  find_package(obs-frontend-api REQUIRED)
  target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE OBS::obs-frontend-api)
endif() 

if(ENABLE_QT)
  find_qt(COMPONENTS Widgets Core) #find_qt is a macro in another cmake file that finds the best Qt Version
  target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE Qt::Core Qt::Widgets) #link the current project to qt core and widgets
  target_compile_options( #add compile options for the project target
    ${CMAKE_PROJECT_NAME} PRIVATE $<$<C_COMPILER_ID:Clang,AppleClang>:-Wno-quoted-include-in-framework-header
                                  -Wno-comma>)
  set_target_properties(
    ${CMAKE_PROJECT_NAME}
    PROPERTIES AUTOMOC ON #When AUTOMOC is set to ON, CMake will automatically run meta object compiler on any C++ source
               AUTOUIC ON #When AUTOUIC is set to ON, CMake will automatically run UI compiler on any .ui files in the target's source files.
               AUTORCC ON)#When AUTORCC is set to ON, CMake will automatically run resource compiler on any .qrc files in the target's source files.
endif()

target_sources(${CMAKE_PROJECT_NAME} PRIVATE src/plugin-main.c) #Specifies sources to use when building a target and/or its dependents

set_target_properties_plugin(${CMAKE_PROJECT_NAME} PROPERTIES OUTPUT_NAME ${_name})

# set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES  ON) #adding the -fPIC compiler flags
# target_compile_options(${CMAKE_PROJECT_NAME} PRIVATE -fPIC)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC -g") #language wide flags passed to all invocations of the compiler
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -g")

