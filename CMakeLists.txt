cmake_minimum_required(VERSION 2.8)
project(tango)
include(CTest)

enable_testing()
#need to define the version of the library
set(MAJOR_VERSION "9")
set(MINOR_VERSION "2")
set(PATCH_VERSION "5")

IF (WIN32)
add_definitions(-DHAVE_STRUCT_TIMESPEC)
add_definitions(-D_TIMESPEC_DEFINED)
add_definitions(-DSTRUCT_TIMESPEC_HAS_TV_SEC)
add_definitions(-DSTRUCT_TIMESPEC_HAS_TV_NSEC)
add_definitions(-D_WIN32)
add_definitions(-D__OSVERSION__=4)
add_definitions(-D__POSIX_NT__)
add_definitions(-D__x86__)
#add_definitions(-DLOG4TANGO_USE_MSTHREADS)
add_definitions(-D_CRT_SECURE_NO_WARNINGS)
add_definitions(-DOMNI_UNLOADABLE_STUBS)
add_definitions(-D_TANGO_LIB)
add_definitions(-DJPG_USE_ASM)
ENDIF()

#convenient versions
set(LIBRARY_VERSION "${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_VERSION}")
set(SO_VERSION "${MAJOR_VERSION}")


message("System TANGO_HOST=$ENV{TANGO_HOST}")
set(TANGO_HOST $ENV{TANGO_HOST})
include(configure/CMakeLists.txt)

include_directories(generated)
#required for generated config.h
include_directories(${PROJECT_BINARY_DIR}/include)

add_subdirectory(include)
add_subdirectory(doxygen)
#source code
add_subdirectory(generated/idl)
include_directories(include)
add_subdirectory(src)
#add_subdirectory(test)
IF(WIN32)
include_directories(${INCLUDE_OBJECT_MS})
set(WIN32_LIBS "ws2_32.lib;mswsock.lib;advapi32.lib;comctl32.lib;odbc32.lib;")
set(PTHREAD_WIN32_LIBS "pthreadVC2.lib;")
link_directories($ENV{PTHREAD_WIN}/Pre-built.2/lib/x86/)
add_library(tango SHARED    $<TARGET_OBJECTS:log4tango_objects>
                            $<TARGET_OBJECTS:idl_objects>
                            $<TARGET_OBJECTS:client_objects>
                            $<TARGET_OBJECTS:jpeg_objects>
                            $<TARGET_OBJECTS:jpeg_mmx_objects>
                            $<TARGET_OBJECTS:server_objects>)
target_link_libraries(tango PUBLIC ${WIN32_LIBS} ${OMNIORB_PKG_LIBRARIES} ${ZMQ_PKG_LIBRARIES} ${PTHREAD_WIN32_LIBS} ${CMAKE_DL_LIBS})
set_property(TARGET tango PROPERTY LINK_FLAGS "/force:multiple")
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/Release)
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/Release)
ELSE()
add_library(tango SHARED    $<TARGET_OBJECTS:log4tango_objects>
                            $<TARGET_OBJECTS:client_objects>
                            $<TARGET_OBJECTS:idl_objects>
                            $<TARGET_OBJECTS:jpeg_objects>
                            $<TARGET_OBJECTS:jpeg_mmx_objects>
                            $<TARGET_OBJECTS:server_objects>
                            $<TARGET_OBJECTS:include_objects>)
target_link_libraries(tango PUBLIC ${ZMQ_PKG_LIBRARIES} ${OMNIORB_PKG_LIBRARIES} ${OMNICOS_PKG_LIBRARIES} ${OMNIDYN_PKG_LIBRARIES} ${CMAKE_DL_LIBS})
target_compile_options(tango PRIVATE -fPIC)
ENDIF()
target_include_directories(tango PUBLIC ${ZMQ_PKG_INCLUDE_DIRS} ${OMNIORB_PKG_INCLUDE_DIRS} ${OMNIDYN_PKG_INCLUDE_DIRS})

target_compile_options(tango PUBLIC ${ZMQ_PKG_CFLAGS_OTHER} ${OMNIORB_PKG_CFLAGS_OTHER} ${OMNICOS_PKG_CFLAGS_OTHER} ${OMNIDYN_PKG_CFLAGS_OTHER})
target_compile_definitions(tango PRIVATE OMNI_UNLOADABLE_STUBS _TANGO_LIB)

set_target_properties(tango PROPERTIES
        VERSION ${LIBRARY_VERSION}
        SOVERSION ${SO_VERSION})

#install code
IF (WIN32)
install(TARGETS tango
        ARCHIVE DESTINATION "${CMAKE_INSTALL_FULL_LIBDIR}"
        LIBRARY DESTINATION "${CMAKE_INSTALL_FULL_LIBDIR}"
        RUNTIME DESTINATION "${CMAKE_INSTALL_FULL_BINDIR}")
ELSE()
install(TARGETS tango LIBRARY DESTINATION "${CMAKE_INSTALL_FULL_LIBDIR}")
ENDIF()
configure_file(tango.pc.cmake tango.pc @ONLY)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/tango.pc"
        DESTINATION "${CMAKE_INSTALL_FULL_LIBDIR}/pkgconfig")

#TODO windows
set(CPACK_PACKAGE_FILE_NAME libtango_${LIBRARY_VERSION}_amd64) #TODO platform
IF (WIN32)
SET(CPACK_GENERATOR "NSIS")
set(CPACK_SOURCE_GENERATOR "ZIP")
#
# CPACK macros below here
#
set (CPACK_PACKAGE_NAME "Tango")
set (CPACK_PACKAGE_VENDOR "www.tango-controls.org")
set (CPACK_PACKAGE_DESCRIPTION_SUMMARY "Tango - Connecting Things Together")
set (CPACK_PACKAGE_VERSION "1.0.0")
set (CPACK_PACKAGE_VERSION_MAJOR "1")
set (CPACK_PACKAGE_VERSION_MINOR "0")
set (CPACK_PACKAGE_VERSION_PATCH "0")
set (CPACK_PACKAGE_INSTALL_DIRECTORY "tango")

# Define components and their display names
set (CPACK_COMPONENTS_ALL tango libraries headers)
set (CPACK_COMPONENT_APPLICATIONS_DISPLAY_NAME "Tango Applications")
set (CPACK_COMPONENT_LIBRARIES_DISPLAY_NAME "Libraries")
set (CPACK_COMPONENT_HEADERS_DISPLAY_NAME "C++ Headers")

# Human readable component descriptions
set (CPACK_COMPONENT_APPLICATIONS_DESCRIPTION
  "Applications using tango")
set (CPACK_COMPONENT_LIBRARIES_DESCRIPTION
  "Static libraries used to build programs with Tango")
set (CPACK_COMPONENT_HEADERS_DESCRIPTION
  "C/C++ header files for use with Tango")

# Define dependencies between components
set (CPACK_COMPONENT_HEADERS_DEPENDS tango)

# Define groups
set(CPACK_COMPONENT_APPLICATIONS_GROUP "Runtime")
set(CPACK_COMPONENT_LIBRARIES_GROUP "Development")
set(CPACK_COMPONENT_HEADERS_GROUP "Development")

set(CPACK_COMPONENT_GROUP_DEVELOPMENT_DESCRIPTION
   "All of the tools you'll ever need to develop software")

# Define NSIS installation types
set(CPACK_ALL_INSTALL_TYPES Full Developer)
set(CPACK_COMPONENT_LIBRARIES_INSTALL_TYPES Developer Full)
set(CPACK_COMPONENT_HEADERS_INSTALL_TYPES Developer Full)
set(CPACK_COMPONENT_APPLICATIONS_INSTALL_TYPES Full)
ELSE()
SET(CPACK_GENERATOR "DEB")
set(CPACK_DEBIAN_PACKAGE_NAME libtango-dev)
set(CPACK_DEBIAN_PACKAGE_VERSION ${LIBRARY_VERSION})
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libcos4-dev, libzmq5-dev | libzmq3-dev")
set(CPACK_DEBIAN_PACKAGE_CONFLICTS "libtango7-dev, libtango8-dev, libtango9-dev")
SET(CPACK_DEBIAN_PACKAGE_MAINTAINER "TANGO team <tango@esrf.fr>") #required
ENDIF()
include(CPack)
