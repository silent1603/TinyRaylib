cmake_minimum_required(VERSION 3.13)

set(CMAKE_TOOLCHAIN_FILE ${CMAKE_CURRENT_SOURCE_DIR}/vcpkg/scripts/buildsystems/vcpkg.cmake CACHE STRING "Vcpkg toolchain file")

project(TinyRaylib)

cmake_policy (SET CMP0111 NEW)

find_package(Git QUIET)
if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
# Update submodules as needed
    option(GIT_SUBMODULE "Check submodules during build" ON)
    if(GIT_SUBMODULE)
        message(STATUS "Submodule update")
    execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                    RESULT_VARIABLE GIT_SUBMOD_RESULT)
        if(NOT GIT_SUBMOD_RESULT EQUAL "0")
        message(FATAL_ERROR "git submodule update --init --recursive failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
        endif()
    endif()
endif()

if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/vcpkg")
    message(FATAL_ERROR "The submodules were not downloaded! GIT_SUBMODULE was turned off or failed. Please update submodules and try again.")
endif()


set(RELEASE_PATH "${PROJECT_SOURCE_DIR}/release")
set(DEBUG_PATH "${PROJECT_SOURCE_DIR}/debug")

find_package(Catch2 CONFIG REQUIRED)
find_package(EnTT CONFIG REQUIRED)
find_package(imgui CONFIG REQUIRED)
find_package(imguizmo CONFIG REQUIRED)
find_package(nlohmann_json CONFIG REQUIRED)
find_package(spdlog CONFIG REQUIRED)
find_package(raylib CONFIG REQUIRED)
if (MSVC)
    
elseif(UNIX AND NOT APPLE)
  find_package(glfw3 CONFIG REQUIRED)
endif()
find_package(box2d CONFIG REQUIRED)

set(PROJECT_PATH ${CMAKE_SOURCE_DIR}/tinyraylib)
set(SRC_PATH ${PROJECT_PATH}/src)
set(SHADER_PATH ${PROJECT_PATH}/shaders)
set(RESOURCE_PATH ${PROJECT_PATH}/resources)

file(GLOB_RECURSE PROJECT_SOURCES ${SRC_PATH}/*.cpp)
file(GLOB_RECURSE PROJECT_HEADERS ${SRC_PATH}/*.hpp)
file(GLOB_RECURSE PROJECT_SHADERS ${SHADER_PATH}/*.*)
file(GLOB_RECURSE PROJECT_RESOURCES ${RESOURCE_PATH}/*.*  )
file(GLOB PROJECT_CONFIGS CMakeLists.txt Readme.md .gitattributes .gitignore .gitmodules LICENSE)

#add_subdirectory(TinyRaylib)
source_group("shaders" FILES ${PROJECT_SHADERS})
source_group("sources" FILES ${PROJECT_SOURCES})
source_group("headers" FILES ${PROJECT_HEADERS})
source_group(TREE  ${PROJECT_PATH} FILES ${PROJECT_RESOURCES})

add_executable(TinyRaylib ${PROJECT_HEADERS} ${PROJECT_SOURCES} ${PROJECT_SHADERS} ${PROJECT_RESOURCES} ${PROJECT_CONFIGS})

set_property(TARGET TinyRaylib PROPERTY CXX_STANDARD 17)

target_link_libraries(TinyRaylib PRIVATE Catch2::Catch2 
                                        PRIVATE EnTT::EnTT 
                                        PRIVATE imgui::imgui 
                                        PRIVATE imguizmo::imguizmo 
                                        PRIVATE box2d::box2d   
                                        PRIVATE nlohmann_json nlohmann_json::nlohmann_json 
                                        PRIVATE spdlog::spdlog spdlog::spdlog_header_only                                        
)

target_include_directories(TinyRaylib PRIVATE ${raylib_INCLUDE_DIRS}) 
target_link_libraries(TinyRaylib PRIVATE ${raylib_LIBRARIES})

if (MSVC)
  set_target_properties(TinyRaylib PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY ${RELEASE_PATH})
  set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT TinyRaylib)
elseif(UNIX AND NOT APPLE)
  target_link_libraries(TinyRaylib PRIVATE Catch2::Catch2WithMain 
                                    PRIVATE glfw)
endif()

add_custom_command(TARGET TinyRaylib POST_BUILD 
                   COMMAND ${CMAKE_COMMAND} -E copy_directory 
                      ${CMAKE_SOURCE_DIR}/TinyRaylib/resources ${RELEASE_PATH}/resources 
                    COMMAND ${CMAKE_COMMAND} -E copy_directory 
                      ${CMAKE_SOURCE_DIR}/TinyRaylib/shaders ${RELEASE_PATH}/shaders
)

set_target_properties(TinyRaylib PROPERTIES 
           ARCHIVE_OUTPUT_DIRECTORY "${RELEASE_PATH}/lib" 
           LIBRARY_OUTPUT_DIRECTORY "${RELEASE_PATH}/lib" 
           RUNTIME_OUTPUT_DIRECTORY "${RELEASE_PATH}" 
           RUNTIME_OUTPUT_DIRECTORY_DEBUG "${RELEASE_PATH}" 
           RUNTIME_OUTPUT_DIRECTORY_RELEASE "${RELEASE_PATH}" 
)