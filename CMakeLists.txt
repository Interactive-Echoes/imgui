project(imgui
    DESCRIPTION "Dear ImGui"
    HOMEPAGE_URL "https://github.com/mozahzah/imgui"
    LANGUAGES CXX)

message("\n------------------------------------------------------------")
message("${PROJECT_NAME}\n")

# Initialize all source files including backends
set(BACKENDS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/backends)
file(GLOB IMGUI_FILES *.h *.cpp)
file(GLOB IMGUI_FREETYPE_FILES ${CMAKE_CURRENT_SOURCE_DIR}/misc/freetype/*.h ${CMAKE_CURRENT_SOURCE_DIR}/misc/freetype/*.cpp)
file(GLOB_RECURSE IMGUI_VULKAN_FILES ${BACKENDS_DIR}/*vulkan*.h ${BACKENDS_DIR}/*vulkan*.cpp)
file(GLOB_RECURSE IMGUI_GLFW_FILES ${BACKENDS_DIR}/*glfw*.h ${BACKENDS_DIR}/*glfw*.cpp)

# First, try to find libraries
find_package(Vulkan)
find_package(glfw3)
find_package(Freetype)

if(WIN32)
  if(DEFINED ENV{VCPKG_ROOT})
    set(VCPKG_CMAKE_PATH "$ENV{VCPKG_ROOT}\\scripts\\buildsystems\\vcpkg.cmake")
    include(${VCPKG_CMAKE_PATH})
    message("Found VCPKG CMake as ${VCPKG_CMAKE_PATH}")
    
    if(NOT Vulkan_FOUND)
      message("Using VCPKG to find Vulkan")
      find_package(Vulkan)
      if(Vulkan_FOUND)
        message("Using VCPKG provided Vulkan")
      endif()
    endif()
    
    if(NOT TARGET glfw)
      message("Using VCPKG to find glfw3")
      find_package(glfw3)
      if(TARGET glfw)
        message("Using VCPKG provided Glfw")
      endif()
    endif()

    if(NOT FREETYPE_FOUND)
      message("Using VCPKG to find freetype")
      find_package(Freetype)
      if(FREETYPE_FOUND)
        message("Using VCPKG provided freetype")
      endif()
    endif()

  else()
    message(WARNING "VCPKG_ROOT is not found. Make sure required libraries are added to either PATH or defined by a parent CMakeLists file. Required libraries are Vulkan and Glfw3")
  endif()
endif()

# Final check to ensure libraries are found
if(NOT Vulkan_FOUND)
  message(FATAL_ERROR "Vulkan not found")
endif()

if(NOT TARGET glfw)
  message(FATAL_ERROR "GLFW3 not found")
endif()

if(NOT FREETYPE_FOUND)
  message(FATAL_ERROR "Freetype not found")
endif()

set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
add_library(${PROJECT_NAME} SHARED ${IMGUI_FILES} ${IMGUI_FREETYPE_FILES} ${IMGUI_VULKAN_FILES} ${IMGUI_GLFW_FILES})
target_link_libraries(${PROJECT_NAME} PUBLIC Vulkan::Vulkan glfw Freetype::Freetype)
target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} ${BACKENDS_DIR})
file(GLOB HEADER_FILES "./*.h")
set_property(TARGET ${PROJECT_NAME} PROPERTY PUBLIC_HEADER ${HEADER_FILES})

set_target_properties(${PROJECT_NAME} PROPERTIES 
  RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
  LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
  ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

install(TARGETS ${PROJECT_NAME}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

get_target_property(DEPENDENCIES ${PROJECT_NAME} LINK_LIBRARIES)
foreach(DEPENDENCY ${DEPENDENCIES})
  if(TARGET ${DEPENDENCY})
    message("Setting up dependency: ${DEPENDENCY}")
    
    set_target_properties(${DEPENDENCY} PROPERTIES 
      RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
      LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
      ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

    get_target_property(DEPENDENCY_TYPE ${DEPENDENCY} TYPE)
    if(NOT DEPENDENCY_TYPE)
      set(DEPENDENCY_TYPE STATIC_LIBRARY)
    endif()

    if(DEPENDENCY_TYPE STREQUAL "STATIC_LIBRARY")
      set(DESTINATION_DIR ${CMAKE_INSTALL_LIBDIR})
    elseif(DEPENDENCY_TYPE STREQUAL "SHARED_LIBRARY")
      if(WIN32)
        set(DESTINATION_DIR ${CMAKE_INSTALL_BINDIR})
      else()
        set(DESTINATION_DIR ${CMAKE_INSTALL_LIBDIR})
      endif()
    else()
      set(DESTINATION_DIR ${CMAKE_INSTALL_LIBDIR})
    endif()
    if (${DESTINATION_DIR})
      install(FILES "$<TARGET_FILE:${DEPENDENCY}>" DESTINATION ${DESTINATION_DIR})
    endif()
  endif()
endforeach()

message("------------------------------------------------------------\n")