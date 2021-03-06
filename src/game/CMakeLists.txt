cmake_minimum_required(VERSION 3.16)
# set(game_target "${CMAKE_PROJECT_NAME}")

# game src files
set(game_src_includes
    "main.cpp"
)
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${game_src_includes})

# TODO: Make platform specific entry point files
# # Add platform specific entry point
# # --------------------------------------------------------------------------------
# if (BUILD_FOR_DESKTOP)
#     set(game_src_includes
#         "main.cpp"
#     )
# endif()

add_executable(game_target
    "${game_src_includes}"
)

# Precompile Headers
# --------------------------------------------------------------------------------
target_precompile_headers(game_target PRIVATE
                          <cinttypes>
                          <string>
                          <vector>
                          <glm/vec4.hpp>
                          <glm/mat4x4.hpp>
                          )

if (BUILD_FOR_DESKTOP)
    find_package(Vulkan REQUIRED)

    target_link_libraries(game_target PUBLIC
        glm::glm
        shared_target
    )

endif ()

if (BUILD_RENDERER_VULKAN)
    find_package(Vulkan REQUIRED)

    copy_vulkan_libs_to_target_bundle(game_target)
endif()

copy_folder_to_target_resources(game_target
                                ${CMAKE_SOURCE_DIR}/resources/shaders
                                shaders)

# Platform includes
# --------------------------------------------------------------------------------
if (APPLE)
	find_library(ObjC objc)
	find_library(CoreFoundation CoreFoundation)
	find_library(CoreVideo CoreVideo)
	find_library(CoreGraphics CoreGraphics)
	find_library(IOKit IOKit)
    find_library(AppKit AppKit)

    target_link_libraries(game_target PUBLIC
        ${CoreGraphics}
        ${CoreVideo}
        ${CoreFoundation}
        ${IOKit}
        ${AppKit}
        ${ObjC}
    )

    set_target_properties(game_target PROPERTIES
                          MACOSX_BUNDLE TRUE
                          XCODE_ATTRIBUTE_LD_RUNPATH_SEARCH_PATHS "@executable_path @executable_path/../Frameworks")

elseif (BUILD_FOR_WINDOWS)
    set_property(TARGET game_target PROPERTY VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/resources")
endif()

if ( CMAKE_CXX_COMPILER_ID MATCHES "Clang|AppleClang|GNU" )
    target_compile_options( game_target PRIVATE -Werror -Wall -Wextra -Wunreachable-code -Wpedantic)
endif()
if ( CMAKE_CXX_COMPILER_ID MATCHES "Clang" )
    target_compile_options( game_target PRIVATE -Wweak-vtables -Wexit-time-destructors -Wglobal-constructors -Wmissing-noreturn )
endif()
if ( CMAKE_CXX_COMPILER_ID MATCHES "MSVC" )
    target_compile_options( game_target PRIVATE /WX /W4 /w44265 /w44061 /w44062 )
endif()