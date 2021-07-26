# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.10.2)

# Declares and names the project.

project(macOS)

macro(copyExe EXE)
    if (APPLE)
        if(CMAKE_RUNTIME_OUTPUT_DIRECTORY)
            set(copyExe_IN_FILE ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${EXE}.app)
        else()
            set(copyExe_IN_FILE ${EXE}.app)
        endif()


        set(copyExe_OUT_FILE ${CMAKE_CURRENT_BINARY_DIR}/EXECUTABLES/${EXE}.app)
    endif()

    add_custom_command(
            TARGET ${EXE}
            POST_BUILD
            COMMAND cp -rv ${copyExe_IN_FILE} ${copyExe_OUT_FILE}
    )
endmacro()

add_executable(
    app
    AppDelegate.m
    ViewController.m
    main.m
)

set(PRINCIPAL_CLASS "NSApplication")

set_target_properties(
  app PROPERTIES
  MACOSX_BUNDLE TRUE
  PRINCIPAL_CLASS "NSApplication"
  MACOSX_BUNDLE_INFO_PLIST ${CMAKE_CURRENT_SOURCE_DIR}/Info.plist
)

# https://github.com/KhronosGroup/Vulkan-LoaderAndValidationLayers/blob/25d5884746a2de7b51a8ef3ec88e1cd8066460e8/demos/macOS/cube/cube.cmake#L32
# Compile the storyboard file with the ibtool.
add_custom_command(TARGET app POST_BUILD
    COMMAND ibtool --errors --warnings --notices --output-format human-readable-text
        --compile ${CMAKE_CURRENT_BINARY_DIR}/app.app/Contents/Resources/Main.storyboardc
        ${CMAKE_CURRENT_SOURCE_DIR}/Base.lproj/Main.storyboard
        COMMENT "Compiling storyboard"
)

target_link_libraries(app "-framework Cocoa")
copyExe(app)
