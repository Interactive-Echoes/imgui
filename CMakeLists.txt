cmake_minimum_required(VERSION 3.20)
project(imgui)

# Set backends directory
set(BACKENDS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/backends)

# Initialize all source files including backends
# ImGui
file(GLOB IMGUI_FILES *.h *.cpp)
# Vulkan
file(GLOB_RECURSE IMGUI_VULKAN_FILES ${BACKENDS_DIR}/*vulkan*.h ${BACKENDS_DIR}/*vulkan*.cpp)
# glfw
file(GLOB_RECURSE IMGUI_GLFW_FILES ${BACKENDS_DIR}/*glfw*.h ${BACKENDS_DIR}/*glfw*.cpp)

# Adding the imgui library with the source files
add_library(imgui STATIC ${IMGUI_FILES} ${IMGUI_VULKAN_FILES} ${IMGUI_GLFW_FILES})

# Find required packages and link them to the imgui project
# Vulkan
find_package(Vulkan REQUIRED)
target_link_libraries(imgui ${Vulkan_LIBRARIES})
target_include_directories (imgui PUBLIC ${Vulkan_INCLUDE_DIRS})
# glfw
find_package(glfw3 REQUIRED)
target_link_libraries(imgui glfw)

# Set include director for the rest of the project modules
set(INCDIRS ${CMAKE_CURRENT_SOURCE_DIR} ${BACKENDS_DIR})
target_include_directories(imgui PUBLIC ${INCDIRS})