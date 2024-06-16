cmake_minimum_required(VERSION 3.20)
project(imgui)

# Set backends directory
set(BACKENDSDIR ${CMAKE_CURRENT_SOURCE_DIR}/backends)

# Initialize all source files including backends
file(GLOB IMGUI_FILES *.h *.cpp)
file(GLOB_RECURSE OPENGL_FILES ${BACKENDSDIR}/*opengl*.h ${BACKENDSDIR}/*opengl*.cpp)
file(GLOB_RECURSE GLFW_FILES ${BACKENDSDIR}/*glfw*.h ${BACKENDSDIR}/*glfw*.cpp)

# Adding the imgui library with the source files
add_library(imgui STATIC ${IMGUI_FILES} ${OPENGL_FILES} ${GLFW_FILES})

# Find required glfw3 package and link it to the imgui library
find_package(glfw3 REQUIRED)
target_link_libraries(imgui glfw)

# Conditional linking goes here
if (APPLE)
target_link_libraries(imgui "-framework OpenGL")
endif()

# Set include director for the rest of the project modules
set(INCDIRS ${CMAKE_CURRENT_SOURCE_DIR} ${BACKENDSDIR})
target_include_directories(imgui PUBLIC ${INCDIRS})