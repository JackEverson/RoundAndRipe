cmake_minimum_required(VERSION 3.15...4.0)

project(RoundAndRipe VERSION 1.0
      DESCRIPTION "Round and Ripe — a first-person farm-automation game about
      quotas, machines, and exactly one liver"
      LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if(MSVC)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -WX-")
else()
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++23")
endif()

# The engine (git submodule). Its CMakeLists is guarded: consumed like this it
# only defines the libraries (garden_engine + glad/glfw/glm/imgui/OpenAL),
# no game targets, no resource copying.
add_subdirectory(garden_engine)

# Resources: this repo's res/ into the build dir next to the exe
add_custom_target(copy_resources
COMMENT "Copying Resources"
)

add_custom_command(TARGET copy_resources PRE_BUILD
COMMAND ${CMAKE_COMMAND} -E copy_directory_if_different
"${CMAKE_CURRENT_SOURCE_DIR}/res/"
"${CMAKE_BINARY_DIR}/res/"
)

set(APP_ICON_RESOURCE "${CMAKE_CURRENT_SOURCE_DIR}/resources.rc")

# The game
set(RoundAndRipe_files
  src/GardenScene.cpp
  src/SaveSystem.cpp

  # include files listed for IDE sake
  src/Event.hpp
  src/Field.hpp
  src/GardenRoom.hpp
  src/GardenScene.hpp
  src/MenuScene.hpp
  src/Plants.hpp
  src/RoundAndRipeEvents.hpp
  src/SaveSystem.hpp
  src/Structures.hpp
  src/Tile.hpp
)

add_library(RoundAndRipe_game STATIC ${RoundAndRipe_files})
target_include_directories(RoundAndRipe_game PUBLIC src)
target_compile_features(RoundAndRipe_game PUBLIC cxx_std_23)
target_link_libraries(RoundAndRipe_game PUBLIC garden_engine)

add_executable(RoundAndRipe src/RoundAndRipe.cpp ${APP_ICON_RESOURCE})
target_link_libraries(RoundAndRipe PUBLIC RoundAndRipe_game)
add_dependencies(RoundAndRipe copy_resources)
