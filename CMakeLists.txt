#Preamble
CMAKE_MINIMUM_REQUIRED(VERSION 3.10)
PROJECT(
    lib_dummy
    VERSION 0.0.1
    DESCRIPTION "Default cpp project template"
    LANGUAGES CXX
)

#CMAKE Options
SET(CMAKE_CXX_STANDARD 17)
SET(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_STATIC_LIBRARY_PREFIX "")
SET(CMAKE_EXPORT_COMPILE_COMMANDS ON)
SET(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH};${CMAKE_CURRENT_SOURCE_DIR}/cmake")


# Dependencies
INCLUDE(doxyTarget)
INCLUDE(listSubdirectories)
INCLUDE(styleEnforce)
INCLUDE(FetchContent)
INCLUDE(findGoogletest)
ENFORCE_STYLE(src include test)

# Static Library target
FILE(GLOB_RECURSE LIB_SOURCES src/*.cpp)
ADD_LIBRARY(static STATIC ${LIB_SOURCES})
TARGET_INCLUDE_DIRECTORIES(static PUBLIC include)
TARGET_INCLUDE_DIRECTORIES(static PRIVATE src)
SET_TARGET_PROPERTIES(static PROPERTIES 
    OUTPUT_NAME ${PROJECT_NAME}
    ARCHIVE_OUTPUT_DIRECTORY lib
    LIBRARY_OUTPUT_DIRECTORY lib
    RUNTIME_OUTPUT_DIRECTORY lib
    EXCLUDE_FROM_ALL FALSE
)
message(STATUS "Library target: '${PROJECT_NAME}_static' added" )

# Shared library target
ADD_LIBRARY(shared SHARED ${LIB_SOURCES})
TARGET_INCLUDE_DIRECTORIES(shared PUBLIC include)
TARGET_INCLUDE_DIRECTORIES(shared PRIVATE src)
SET_TARGET_PROPERTIES(shared PROPERTIES
    PREFIX ""
    OUTPUT_NAME ${PROJECT_NAME}
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}
    ARCHIVE_OUTPUT_DIRECTORY lib
    LIBRARY_OUTPUT_DIRECTORY lib
    RUNTIME_OUTPUT_DIRECTORY lib
    EXCLUDE_FROM_ALL TRUE
)
message(STATUS "Library target: '${PROJECT_NAME}_shared' added" )

#TESTING
#Build Tests
ENABLE_TESTING()
FILE(GLOB_RECURSE UNIT_TEST_SOURCES test/unit/*.cpp)
ADD_EXECUTABLE(UTEST_${PROJECT_NAME} ${UNIT_TEST_SOURCES})
TARGET_LINK_LIBRARIES(UTEST_${PROJECT_NAME} PUBLIC 
    GTest::gtest_main 
    static
)
SET_TARGET_PROPERTIES(UTEST_${PROJECT_NAME} PROPERTIES 
    RUNTIME_OUTPUT_DIRECTORY tests
)

#collect tests
INCLUDE(GoogleTest)
GTEST_DISCOVER_TESTS(UTEST_${PROJECT_NAME})
ADD_CUSTOM_TARGET(
    tests
    DEPENDS static
    COMMAND export GTEST_COLOR=1 && ctest --output-on-failure --progress
)

