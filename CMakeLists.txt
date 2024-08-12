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
target_include_directories(imgui PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${CMAKE_CURRENT_SOURCE_DIR}/backends
)

# Link GLFW and Vulkan libraries
find_package(glfw3 REQUIRED)
find_package(Vulkan REQUIRED)

target_link_libraries(imgui PUBLIC glfw Vulkan::Vulkan)
