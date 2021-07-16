### Sets the minimal required CMake version to run this script.
cmake_minimum_required(VERSION 3.13)

### Sets project name and version.
project(cmake_discord_game_sdk VERSION 0.1)

### Module used to download Discord Game SDK.
include(FetchContent)

### OS Requirements
### - OS must contain an identifier corresponding to your current operating system: Windows, Macos, Linux.
if (NOT OS) # If OS is not defined, it tries to find it.
    if (WIN32)
        set(OS Windows)
    elseif (APPLE)
        set(OS Macos)
    elseif (UNIX)
        set(OS Linux)
    else ()
        message(FATAL_ERROR "Cannot detect operating system")
    endif ()
    message(STATUS "OS was not defined, automatically set to ${OS}")
elseif(NOT OS STREQUAL "Windows" AND NOT OS STREQUAL "Macos" AND NOT OS STREQUAL "Linux") # OS can only have following values: Windows, Macos, Linux.
    message(FATAL_ERROR "OS has wrong value. Was: ${OS}, must be: Windows|Macos|Linux")
endif ()

### ARCH Requirements
### - ARCH must contain the processor architecture you want to use to build the SDK, possible values: x86_64, x86 (Window only).
if (NOT ARCH)
    message(WARNING "ARCH not defined, automatically set to x86_64")
    set(ARCH x86_64)
endif ()
if (ARCH STREQUAL "x86" AND NOT OS STREQUAL "Windows")              # ARCH can be x86 only on Windows
    message(FATAL_ERROR "x86 is only possible on Windows")
endif ()
if (ARCH STREQUAL "x86" OR ARCH STREQUAL "x86_64")
    message(STATUS "ARCH is ${ARCH}")
else ()
    message(FATAL_ERROR "ARCH has wrong value. Was: ${ARCH}, must be: x86_64|x86")
endif ()

### Current Discord Game SDK Version
set(DISCORD_GAME_SDK_V_MAJOR 2)
set(DISCORD_GAME_SDK_V_MINOR 5)
set(DISCORD_GAME_SDK_V_BUILD 6)

### Downloads Discord Game SDK
FetchContent_Declare(
    discord_game_sdk
    URL "https://dl-game-sdk.discordapp.net/${DISCORD_GAME_SDK_V_MAJOR}.${DISCORD_GAME_SDK_V_MINOR}.${DISCORD_GAME_SDK_V_BUILD}/discord_game_sdk.zip"
)
FetchContent_GetProperties(discord_game_sdk)
if (NOT discord_game_sdk_POPULATED)
    FetchContent_Populate(discord_game_sdk)
endif ()
set(DISCORD_GAME_SDK_PATH ${discord_game_sdk_SOURCE_DIR})

### Installs Discord Game SDK headers
install(DIRECTORY ${DISCORD_GAME_SDK_PATH}/c/ DESTINATION include/ FILES_MATCHING PATTERN "*.h")
install(DIRECTORY ${DISCORD_GAME_SDK_PATH}/cpp/ DESTINATION include/ FILES_MATCHING PATTERN "*.h")

### Target To Download Discord Game SDK
set(CMAKE_DISCORD_GAME_SDK_CONFIG_TARGET_NAME ${PROJECT_NAME}_config)
set(CMAKE_DISCORD_GAME_SDK_INCLUDE_DIR ${DISCORD_GAME_SDK_PATH}/c;${DISCORD_GAME_SDK_PATH}/cpp)
add_library(${CMAKE_DISCORD_GAME_SDK_CONFIG_TARGET_NAME} INTERFACE)
target_include_directories(${CMAKE_DISCORD_GAME_SDK_CONFIG_TARGET_NAME}
    INTERFACE ${CMAKE_DISCORD_GAME_SDK_INCLUDE_DIR})
set_target_properties(${CMAKE_DISCORD_GAME_SDK_CONFIG_TARGET_NAME} PROPERTIES INTERFACE_COMPILE_DEFINITIONS
    CMAKE_DISCORD_GAME_SDK_VERSION="${DISCORD_GAME_SDK_V_MAJOR}.${DISCORD_GAME_SDK_V_MINOR}.${DISCORD_GAME_SDK_V_BUILD}")

### Installs Discord Game SDK binaries
if (OS STREQUAL "Windows")
    if (ARCH STREQUAL "x86_64")
        set(CMAKE_DISCORD_GAME_SDK_LIB_PATH ${DISCORD_GAME_SDK_PATH}/lib/x86_64/discord_game_sdk.dll.lib)
    else ()
        set(CMAKE_DISCORD_GAME_SDK_LIB_PATH ${DISCORD_GAME_SDK_PATH}/lib/x86/discord_game_sdk.dll.lib)
    endif ()
elseif (OS STREQUAL "Linux")
    set(CMAKE_DISCORD_GAME_SDK_LIB_PATH ${DISCORD_GAME_SDK_PATH}/lib/x86_64/discord_game_sdk.so)
elseif (OS STREQUAL "Macos")
    set(CMAKE_DISCORD_GAME_SDK_LIB_PATH ${DISCORD_GAME_SDK_PATH}/lib/x86_64/discord_game_sdk.dylib)
endif ()
install(FILES ${CMAKE_DISCORD_GAME_SDK_LIB_PATH} DESTINATION lib/)

### Final Target
set(CMAKE_DISCORD_GAME_SDK_TARGET_NAME ${PROJECT_NAME})
set(CMAKE_DISCORD_GAME_SDK_SOURCES
    ${DISCORD_GAME_SDK_PATH}/cpp/achievement_manager.cpp
    ${DISCORD_GAME_SDK_PATH}/cpp/activity_manager.cpp
    ${DISCORD_GAME_SDK_PATH}/cpp/application_manager.cpp
    ${DISCORD_GAME_SDK_PATH}/cpp/core.cpp
    ${DISCORD_GAME_SDK_PATH}/cpp/image_manager.cpp
    ${DISCORD_GAME_SDK_PATH}/cpp/lobby_manager.cpp
    ${DISCORD_GAME_SDK_PATH}/cpp/network_manager.cpp
    ${DISCORD_GAME_SDK_PATH}/cpp/overlay_manager.cpp
    ${DISCORD_GAME_SDK_PATH}/cpp/relationship_manager.cpp
    ${DISCORD_GAME_SDK_PATH}/cpp/storage_manager.cpp
    ${DISCORD_GAME_SDK_PATH}/cpp/store_manager.cpp
    ${DISCORD_GAME_SDK_PATH}/cpp/types.cpp
    ${DISCORD_GAME_SDK_PATH}/cpp/user_manager.cpp
    ${DISCORD_GAME_SDK_PATH}/cpp/voice_manager.cpp)
add_library(${CMAKE_DISCORD_GAME_SDK_TARGET_NAME} STATIC ${CMAKE_DISCORD_GAME_SDK_SOURCES})
target_link_libraries(${CMAKE_DISCORD_GAME_SDK_TARGET_NAME}
    PUBLIC ${CMAKE_DISCORD_GAME_SDK_CONFIG_TARGET_NAME} ${CMAKE_DISCORD_GAME_SDK_LIB_PATH})

### Install Final Target
install(TARGETS ${CMAKE_DISCORD_GAME_SDK_TARGET_NAME})