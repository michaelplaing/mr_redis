cmake_minimum_required(VERSION 3.22)

# Specify search path for CMake modules to be loaded by include() and find_package()
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules")

project(
    mr_redis VERSION 0.1.0
    DESCRIPTION "MQTT5 for Redis"
    LANGUAGES C CXX
)

find_package(Git QUIET)
if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
# Update submodules as needed
    option(GIT_SUBMODULE "Check submodules during build" ON)

    if(GIT_SUBMODULE)
        message(STATUS "Submodule update")
        execute_process(
            COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive --remote
            WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
            RESULT_VARIABLE GIT_SUBMOD_RESULT
        )

        if(NOT GIT_SUBMOD_RESULT EQUAL "0")
            message(
                FATAL_ERROR "\
                    git submodule update --init --recursive failed with \
                    ${GIT_SUBMOD_RESULT}, please checkout submodules.\
                "
            )
        endif()
    endif()
endif()

if(NOT EXISTS "${PROJECT_SOURCE_DIR}/extern/mister/CMakeLists.txt")
    message(
        FATAL_ERROR "\
            The submodules were not downloaded! GIT_SUBMODULE was turned off \
            or failed. Please update submodules and try again.\
        "
    )
endif()

# Only do these if this is the main project, and not if it is included through add_subdirectory
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
    # Optionally set things like CMAKE_CXX_STANDARD, CMAKE_POSITION_INDEPENDENT_CODE here
    set(CMAKE_C_STANDARD 23)
    set(CMAKE_C_EXTENSIONS OFF)

    # Let's nicely support folders in IDEs
    set_property(GLOBAL PROPERTY USE_FOLDERS ON)

    # Testing only available if this is the main app
    # Note this needs to be done in the main CMakeLists
    # since it calls enable_testing, which must be in the
    # main CMakeLists.
    include(CTest)

    # Docs only available if this is the main app
    find_package(
        Doxygen
        REQUIRED dot
        OPTIONAL_COMPONENTS mscgen dia
    )
    if(Doxygen_FOUND)
        add_subdirectory(docs)
    else()
        message(STATUS "Doxygen not found, not building docs")
    endif()
endif()

### Require out-of-source builds
file(TO_CMAKE_PATH "${PROJECT_BINARY_DIR}/CMakeLists.txt" LOC_PATH)

if(EXISTS "${LOC_PATH}")
    message(
        FATAL_ERROR "You cannot build in a source directory "
        "(or any directory with a CMakeLists.txt file). "
        "Please make a build subdirectory and use that. "
        "Remove the CMakeCache.txt and CMakeFiles/ just created."
    )
endif()

option(UNIT_TESTING "Build with unit tests" OFF) # not yet implmented
option(MODULE_TESTING "Build with module tests" OFF) # not yet implmented

if (UNIT_TESTING OR MODULE_TESTING)
  set(BUILD_STATIC_LIB ON)
endif()

if (UNIT_TESTING)
    add_subdirectory(tests)
endif ()

add_subdirectory(apps)

add_subdirectory(module)

add_subdirectory(extern/mister)

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)

message(STATUS "********************************************")
message(STATUS "********** ${PROJECT_NAME} build options : **********")

message(STATUS "Unit testing: ${UNIT_TESTING}")
message(STATUS "Module code testing: ${MODULE_TESTING}")

message(STATUS "********************************************")
