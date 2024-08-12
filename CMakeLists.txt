cmake_minimum_required(VERSION 3.14)
project(imgui)

# Set the version of imgui
set(IMGUI_VERSION "1.0")

# Define the target library
add_library(imgui STATIC
        imgui.cpp
        imgui_demo.cpp
        imgui_draw.cpp
        imgui_tables.cpp
        imgui_widgets.cpp
        backends/imgui_impl_glfw.cpp
        backends/imgui_impl_vulkan.cpp
)

# Set include directories
target_include_directories(imgui
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/backends>
        $<INSTALL_INTERFACE:include>
)

# Vulkan SDK setup
if (DEFINED VULKAN_SDK_PATH)
    set(Vulkan_INCLUDE_DIRS "${VULKAN_SDK_PATH}/Include")
    set(Vulkan_LIBRARIES "${VULKAN_SDK_PATH}/Lib/vulkan-1.lib")
    set(Vulkan_FOUND "True")
else ()
    find_package(Vulkan REQUIRED)
    message(STATUS "Found Vulkan: $ENV{VULKAN_SDK}")
endif ()

# GLFW setup
if (DEFINED GLFW_PATH)
    message(STATUS "Using GLFW path specified in .env at: ${GLFW_PATH}")
    set(GLFW_INCLUDE_DIRS "${GLFW_PATH}/include")
    if (MSVC)
        set(GLFW_LIB "${GLFW_PATH}/lib-vc2019/glfw3.lib")
    elseif (CMAKE_GENERATOR STREQUAL "MinGW Makefiles" OR CMAKE_GENERATOR STREQUAL "Ninja")
        set(GLFW_LIB "${GLFW_PATH}/lib-mingw-w64/libglfw3.a")
    endif ()
else ()
    find_package(glfw3 3.3 REQUIRED)
    set(GLFW_LIB glfw)
    message(STATUS "Found GLFW")
endif ()

# Add include directories
target_include_directories(imgui PUBLIC ${Vulkan_INCLUDE_DIRS} ${GLFW_INCLUDE_DIRS})

# Link libraries
target_link_libraries(imgui PUBLIC ${Vulkan_LIBRARIES} ${GLFW_LIB})

# Export imgui targets
install(TARGETS imgui
        EXPORT imgui-targets
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include
)

# Export the targets to a script
install(EXPORT imgui-targets
        FILE imgui-targets.cmake
        NAMESPACE imgui::
        DESTINATION lib/cmake/imgui
)

# Create a Config.cmake file
include(CMakePackageConfigHelpers)
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfig.cmake"
        INSTALL_DESTINATION lib/cmake/imgui
)

# Generate the version file for the config file
write_basic_package_version_file(
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfigVersion.cmake"
        VERSION "${IMGUI_VERSION}"
        COMPATIBILITY AnyNewerVersion
)

# Install the configuration files
install(FILES
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfigVersion.cmake"
        DESTINATION lib/cmake/imgui
)