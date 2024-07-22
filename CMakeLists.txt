project(imgui
    DESCRIPTION "Dear ImGui"
    HOMEPAGE_URL "https://github.com/mozahzah/imgui"
    LANGUAGES CXX)

message("\n------------------------------------------------------------")
message("${PROJECT_NAME}\n")

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

set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
add_library(${PROJECT_NAME} SHARED ${IMGUI_FILES} ${IMGUI_VULKAN_FILES} ${IMGUI_GLFW_FILES})
find_package(Vulkan REQUIRED)
find_package(glfw3 REQUIRED)
target_link_libraries(${PROJECT_NAME} PUBLIC Vulkan::Vulkan glfw)
target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} ${BACKENDS_DIR})

install(TARGETS ${PROJECT_NAME}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME})

get_target_property(DEPENDENCIES ${PROJECT_NAME} LINK_LIBRARIES)
foreach(DEPENDENCY ${DEPENDENCIES})
  if(TARGET ${DEPENDENCY})
    message("Installing dependency: ${DEPENDENCY}")
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
    install(FILES "$<TARGET_FILE:${DEPENDENCY}>" DESTINATION ${DESTINATION_DIR})
  endif()
endforeach()

message("------------------------------------------------------------\n")