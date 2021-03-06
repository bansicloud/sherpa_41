cmake_minimum_required(VERSION 3.1)
project(sherpa_41)

# Set compiler flags
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})

option(EXECUTABLE "Generate ${PROJECT_NAME} executable" ON)
option(SANITIZER "Test with clang sanitizer" OFF)
option(COVERAGE "Test with coverage" OFF)

include(gtest.cmake)
include_directories(./include)

# Define the source files and dependencies for the executable
set(SOURCE_FILES
        src/css.cpp
        src/display.cpp
        src/dom.cpp
        src/layout.cpp
        src/style.cpp
        src/parser/parser.cpp
        src/parser/css.cpp
        src/parser/html.cpp
        src/renderer/canvas.cpp
        src/visitor/printer.cpp
        )
set(TEST_FILES
        tests/css.cpp
        tests/display.cpp
        tests/dom.cpp
        tests/layout.cpp
        tests/main.cpp
        tests/style.cpp
        tests/parser/css.cpp
        tests/parser/html.cpp
        tests/renderer/canvas.cpp
        tests/visitor/printer.cpp
        )
set(APP_FILES
        src/main.cpp
        )

# set compile flags
if (MSVC)
    set(PROJ_COMPILE_OPTS /std:c++17 /W4 /WX /EHsc)

    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /Ox")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /Od")

    # VS 2017 removed tr1
    add_definitions(-DGTEST_HAS_TR1_TUPLE=0)
else()
    set(PROJ_COMPILE_OPTS -std=c++17 -Wall -Werror -Wextra -pedantic -pedantic-errors -g)
    set(EXEC_COMPILE_OPTS -Wno-keyword-macro) # Magick++ error
    set(EXEC_COMPILE_OPTS ${EXEC_COMPILE_OPTS} -march=native -O3 -pipe)
endif()

# sherpa_41 executable
if (EXECUTABLE)
    add_executable(${PROJECT_NAME} ${SOURCE_FILES} ${APP_FILES})
    add_definitions( -DMAGICKCORE_QUANTUM_DEPTH=16 )
    add_definitions( -DMAGICKCORE_HDRI_ENABLE=0 )
    find_package(ImageMagick COMPONENTS Magick++)
    include_directories(${ImageMagick_INCLUDE_DIRS})
    target_compile_options(${PROJECT_NAME} PRIVATE ${CMAKE_CXX_FLAGS} ${PROJ_COMPILE_OPTS} ${EXEC_COMPILE_OPTS})
    target_link_libraries(${PROJECT_NAME} ${ImageMagick_LIBRARIES})
endif()

# tests
find_package(Threads)
add_executable(${PROJECT_NAME}-test ${SOURCE_FILES} ${TEST_FILES})
target_compile_options(${PROJECT_NAME}-test PRIVATE ${CMAKE_CXX_FLAGS} ${PROJ_COMPILE_OPTS})
target_link_libraries(${PROJECT_NAME}-test PRIVATE gtest ${CMAKE_THREAD_LIBS_INIT})

# coverage
if (COVERAGE)
    target_compile_options(${PROJECT_NAME}-test PRIVATE --coverage)
    target_link_libraries(${PROJECT_NAME}-test PRIVATE --coverage)
endif()

# sanitizer
if (SANITIZER)
    target_compile_options(${PROJECT_NAME}-test PRIVATE -fno-omit-frame-pointer -fsanitize=address)
    target_link_libraries(${PROJECT_NAME}-test PRIVATE -fno-omit-frame-pointer -fsanitize=address)
endif()