project(imgui
    DESCRIPTION "Dear ImGui"
    HOMEPAGE_URL "https://github.com/mozahzah/imgui"
    LANGUAGES CXX)

message("\n------------------------------------------------------------")
message("ImGui\n")

# Initialize all source files including backends
set(BACKENDS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/backends)
file(GLOB IMGUI_FILES *.h *.cpp)
file(GLOB_RECURSE IMGUI_VULKAN_FILES ${BACKENDS_DIR}/*vulkan*.h ${BACKENDS_DIR}/*vulkan*.cpp)
file(GLOB_RECURSE IMGUI_GLFW_FILES ${BACKENDS_DIR}/*glfw*.h ${BACKENDS_DIR}/*glfw*.cpp)

if(WIN32)
  if(DEFINED ENV{VCPKG_ROOT})
    message("Using VCPKG Root as $ENV{VCPKG_ROOT}")
    set(VCPKG_CMAKE_PATH "$ENV{VCPKG_ROOT}\\scripts\\buildsystems\\vcpkg.cmake")
    include(${VCPKG_CMAKE_PATH})
    message("Found VCPKG CMake as ${VCPKG_CMAKE_PATH}")
  else()
    message(WARNING "VCPKG_ROOT is not found. Make sure required libraries are added to either PATH or defined by a parent CMakeLists file. Required libraries are Vulkan and Glfw3")
  endif()
endif()

add_library(imgui STATIC ${IMGUI_FILES} ${IMGUI_VULKAN_FILES} ${IMGUI_GLFW_FILES})
find_package(Vulkan REQUIRED)
find_package(glfw3 REQUIRED)
target_link_libraries(imgui PUBLIC Vulkan::Vulkan glfw)
target_include_directories(imgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} ${BACKENDS_DIR})

add_custom_command(TARGET imgui POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:Vulkan::Vulkan> ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/$<CONFIG>)
add_custom_command(TARGET imgui POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:glfw> ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/$<CONFIG>)

message("------------------------------------------------------------\n")