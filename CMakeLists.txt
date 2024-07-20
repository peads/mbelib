cmake_minimum_required(VERSION 2.8.12)
project(mbe)
SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake/")

option(NATIVE OFF)

if (NOT "Debug" STREQUAL "${CMAKE_BUILD_TYPE}")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DNDEBUG -Ofast -Wall -fno-dwarf2-cfi-asm -fno-asynchronous-unwind-tables -fomit-frame-pointer -ftree-vectorize")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DNDEBUG -Ofast -Wall -fno-dwarf2-cfi-asm -fno-asynchronous-unwind-tables -fomit-frame-pointer -ftree-vectorize")
    include(TargetArch)
    target_architecture(TARGET_ARCH)
    if (NATIVE)
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -march=native -mtune=native")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native -mtune=native")
        if (TARGET_ARCH STREQUAL "aarch64")
            set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mcpu=native")
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mcpu=native")
        endif()
    endif()

    if (TARGET_ARCH MATCHES "(aarch64|x86)")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -momit-leaf-frame-pointer")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -momit-leaf-frame-pointer")
    endif()
endif()

if(MSVC)
    # needed for M_PI macro
    add_definitions(-D_USE_MATH_DEFINES)
endif()

FILE(GLOB SRCS *.c)

include_directories("${PROJECT_SOURCE_DIR}")

ADD_LIBRARY(mbe-static STATIC ${SRCS})
ADD_LIBRARY(mbe-shared SHARED ${SRCS})
if(NOT WIN32)
    TARGET_LINK_LIBRARIES(mbe-static m)
    TARGET_LINK_LIBRARIES(mbe-shared m)
endif()

include(GNUInstallDirs)

set_target_properties(mbe-static mbe-shared
                      PROPERTIES
                      OUTPUT_NAME mbe
                      VERSION 1.3
                      SOVERSION 1
                      INSTALL_NAME_DIR ${CMAKE_INSTALL_FULL_LIBDIR}
                      PUBLIC_HEADER "mbelib.h")

configure_file("${CMAKE_CURRENT_SOURCE_DIR}/libmbe.pc.in"
               "${CMAKE_CURRENT_BINARY_DIR}/libmbe.pc"
               @ONLY)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/libmbe.pc"
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")

install(TARGETS mbe-static mbe-shared
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# uninstall target
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)

option(DISABLE_TEST "Disable building of test framework." OFF)

if (NOT DISABLE_TEST)
    enable_testing()
    add_subdirectory(test)
endif()

message("-- CMAKE_C_FLAGS=${CMAKE_C_FLAGS}")
message("-- CMAKE_CXX_FLAGS=${CMAKE_CXX_FLAGS}")
