cmake_minimum_required(VERSION 3.14) # For get_filename_component

project(orbis-lib-gen)
include(FetchContent)

# Show progress of dependency cloning.
Set(FETCHCONTENT_QUIET FALSE)

# Fetch ps4libdoc definitions
FetchContent_Declare(
  ps4libdoc
  GIT_REPOSITORY https://github.com/idc/ps4libdoc.git
  GIT_TAG 62d172c83a819234a8bf61a89c10ae781669a67b # branch 5.05
  GIT_SHALLOW TRUE
  GIT_PROGRESS TRUE
  EXCLUDE_FROM_ALL
)
FetchContent_MakeAvailable(ps4libdoc)

# Run python script to generate library definitions
set(LIB_PATH ${ps4libdoc_SOURCE_DIR}/system/common/lib)
execute_process(COMMAND python generate.py ${LIB_PATH}
                WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR})

# Generate stub targets for each c file
set(LIB_TARGETS "")
file(GLOB SOURCE_FILES ${CMAKE_CURRENT_SOURCE_DIR}/build/*.c)
foreach(SOURCE_FILE ${SOURCE_FILES})
    get_filename_component(FILENAME ${SOURCE_FILE} NAME_WE)
    add_library(${FILENAME} SHARED ${SOURCE_FILE})
    target_compile_options(${FILENAME} PRIVATE -ffreestanding -nostdlib -fno-builtin -fPIC)
    string(APPEND LIB_TARGETS " ${FILENAME}")
endforeach()
