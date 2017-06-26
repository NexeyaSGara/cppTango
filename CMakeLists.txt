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
IF(CMAKE_CL_64)
add_definitions(-D_64BITS)
ELSE()
add_definitions(-DJPG_USE_ASM)
ENDIF()
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
IF(CMAKE_CL_64)
link_directories($ENV{PTHREAD_WIN}/Pre-built.2/lib/x64/)
ELSE()
link_directories($ENV{PTHREAD_WIN}/Pre-built.2/lib/x86/)
ENDIF()
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

message(${INCLUDE_OBJECT_MS})

install(TARGETS tango
        ARCHIVE DESTINATION lib COMPONENT static
        RUNTIME DESTINATION bin COMPONENT dynamic)

set(LOG4TANGO_HEADERS
        include/log4tango/Appender.hh
        include/log4tango/AppenderAttachable.hh
        include/log4tango/LayoutAppender.hh
        include/log4tango/FileAppender.hh
        include/log4tango/RollingFileAppender.hh
        include/log4tango/OstreamAppender.hh
        include/log4tango/Layout.hh
        include/log4tango/PatternLayout.hh
        include/log4tango/XmlLayout.hh
        include/log4tango/Logger.hh
        include/log4tango/LogSeparator.hh
        include/log4tango/LoggerStream.hh
        include/log4tango/LogStream.hh
        include/log4tango/LogStreambuf.hh
        include/log4tango/LoggingEvent.hh
        include/log4tango/Level.hh
        include/log4tango/NDC.hh
        include/log4tango/TimeStamp.hh
        include/log4tango/Filter.hh
        include/log4tango/Export.hh
        include/log4tango/Portability.hh)

set(LOG4TANGO_THREADING
        include/log4tango/threading/DummyThreads.hh
        include/log4tango/threading/PThreads.hh
        include/log4tango/threading/MSThreads.hh
        include/log4tango/threading/Threading.hh)

set(CLIENT_HEADERS
        include/tango/client/accessproxy.h
        include/tango/client/apiexcept.h
        include/tango/client/cbthread.h
        include/tango/client/dbapi.h
        include/tango/client/devapi.h
        include/tango/client/devasyn.h
        include/tango/client/event.h
        include/tango/client/eventconsumer.h
        include/tango/client/filedatabase.h
        include/tango/client/group.h
        include/tango/client/lockthread.h
        include/tango/client/Database.h
        include/tango/client/DbDevice.h
        include/tango/client/ApiUtil.h
        include/tango/client/DeviceData.h
        include/tango/client/DeviceAttribute.h
        include/tango/client/DevicePipe.h
        include/tango/client/Connection.h
        include/tango/client/DeviceProxy.h
        include/tango/client/AttributeProxy.h)

set(CLIENT_HELPERS
        tango/client/helpers/DeviceProxyHelper.h
        tango/client/helpers/PogoHelper.h
        tango/client/helpers/TangoExceptionsHelper.h
        tango/client/helpers/Xstring.h)

set(SERVER_HEADERS 
        include/tango/server/attrdesc.h
        include/tango/server/attrprop.h
        include/tango/server/attrmanip.h
        include/tango/server/attribute.h
        include/tango/server/auto_tango_monitor.h
        include/tango/server/basiccommand.h
        include/tango/server/blackbox.h
        include/tango/server/classattribute.h
        include/tango/server/classpipe.h
        include/tango/server/command.h
        include/tango/server/coutappender.h
        include/tango/server/coutbuf.h
        include/tango/server/device.h
        include/tango/server/device_2.h
        include/tango/server/device_3.h
        include/tango/server/device_4.h
        include/tango/server/device_5.h
        include/tango/server/deviceclass.h
        include/tango/server/devintr.h
        include/tango/server/dintrthread.h
        include/tango/server/dserver.h
        include/tango/server/dserverclass.h
        include/tango/server/dserversignal.h
        include/tango/server/encoded_attribute.h
        include/tango/server/encoded_format.h
        include/tango/server/eventsupplier.h
        include/tango/server/except.h
        include/tango/server/fwdattrdesc.h
        include/tango/server/fwdattribute.h
        include/tango/server/log4tango.h
        include/tango/server/logcmds.h
        include/tango/server/logging.h
        include/tango/server/logstream.h
        include/tango/server/multiattribute.h
        include/tango/server/ntservice.h
        include/tango/server/pipe.h
        include/tango/server/pipedesc.h
        include/tango/server/pollcmds.h
        include/tango/server/pollext.h
        include/tango/server/pollobj.h
        include/tango/server/pollring.h
        include/tango/server/pollthread.h
        include/tango/server/readers_writers_lock.h
        include/tango/server/rootattreg.h
        include/tango/server/seqvec.h
        include/tango/server/tango_config.h
        include/tango/server/tango_const.h
        include/tango/server/tango_monitor.h
        include/tango/server/tangoappender.h
        include/tango/server/tangorollingfileappender.h
        include/tango/server/utils.h
        include/tango/server/w32win.h
        include/tango/server/w_attribute.h
        include/tango/server/w_pipe.h
        include/tango/server/subdev_diag.h)     
        

set(INCLUDE_OBJECT_MS
        include/tango.h
        ${SERVER_HEADERS}
        ${CLIENT_HEADERS}
        ${CLIENT_HELPERS} 
        ${LOG4TANGO_HEADERS} 
        ${LOG4TANGO_THREADING})
        
install(FILES ${INCLUDE_OBJECT_MS} DESTINATION include COMPONENT headers)
configure_file(tango.pc.cmake tango.pc @ONLY)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/tango.pc"
        DESTINATION include/pkgconfig COMPONENT headers)
ELSE()
install(TARGETS tango LIBRARY DESTINATION "${CMAKE_INSTALL_FULL_LIBDIR}")
configure_file(tango.pc.cmake tango.pc @ONLY)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/tango.pc"
        DESTINATION "${CMAKE_INSTALL_FULL_LIBDIR}/pkgconfig")
ENDIF()

IF (WIN32)
IF(CMAKE_CL_64)
set(CPACK_PACKAGE_FILE_NAME libtango_${LIBRARY_VERSION}_${CMAKE_VS_PLATFORM_TOOLSET}_x64) #TODO platform
ELSE()
set(CPACK_PACKAGE_FILE_NAME libtango_${LIBRARY_VERSION}_${CMAKE_VS_PLATFORM_TOOLSET}_x86) #TODO platform
ENDIF()
#
# CPACK macros below here
#
set (CPACK_PACKAGE_NAME "Tango")
set (CPACK_PACKAGE_VENDOR "www.tango-controls.org")
set (CPACK_PACKAGE_DESCRIPTION_SUMMARY "Tango - Connecting Things Together")
set (CPACK_PACKAGE_VERSION "${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_VERSION}")
set (CPACK_PACKAGE_VERSION_MAJOR ${MAJOR_VERSION})
set (CPACK_PACKAGE_VERSION_MINOR ${MINOR_VERSION})
set (CPACK_PACKAGE_VERSION_PATCH ${PATCH_VERSION})

# Define components and their display names
set (CPACK_COMPONENTS_ALL static dynamic headers)

# Human readable component descriptions
set (CPACK_COMPONENT_STATIC_DISPLAY_NAME
  "Tango.lib")
set (CPACK_COMPONENT_DYNAMIC_DISPLAY_NAME
  "Tango.dll")
set (CPACK_COMPONENT_HEADERS_DISPLAY_NAME
  "c++ Headers")
  set(CPACK_COMPONENT_STATIC_DESCRIPTION 
   "Tango.lib - for building projects")
 set(CPACK_COMPONENT_DYNAMIC_DESCRIPTION
   "Tango.dll - for running Tango executables")
 set(CPACK_COMPONENT_HEADERS_DESCRIPTION
   "C/C++ header files for use with Tango")

# Define NSIS installation types
set(CPACK_ALL_INSTALL_TYPES Development Runtime)
set(CPACK_COMPONENT_STATIC_GROUP "Development" )
set(CPACK_COMPONENT_DYNAMIC_GROUP "Runtime")
set(CPACK_COMPONENT_HEADERS_GROUP "Development")

set(CPACK_COMPONENT_STATIC_INSTALL_TYPES Development)
set(CPACK_COMPONENT_HEADERS_INSTALL_TYPES Development)
set(CPACK_COMPONENT_DYNAMIC_INSTALL_TYPES Development Runtime)

set(CPACK_COMPONENT_GROUP_DEVELOPMENT_DESCRIPTION
   "All of the tools you'll ever need to develop Tango software")
set(CPACK_COMPONENT_GROUP_RUNTIME_DESCRIPTION
   "All of the required libraries to launch Tango software")

set(CPACK_RESOURCE_FILE_LICENSE ${PROJECT_BINARY_DIR}/LICENSE)
set(CPACK_NSIS_HELP_LINK "http://www.tango-controls.org.org")
set(CPACK_NSIS_URL_INFO_ABOUT "http://www.tango-controls.org.org")
set(CPACK_NSIS_MODIFY_PATH ON)
set(CPACK_NSIS_ENABLE_UNINSTALL_BEFORE_INSTALL ON)
set(CPACK_NSIS_MENU_LINKS
    "http://tango-controls.readthedocs.io/en/latest/" "Tango Doc")
ELSE()
SET(CPACK_GENERATOR "DEB")
set(CPACK_DEBIAN_PACKAGE_NAME libtango-dev)
set(CPACK_DEBIAN_PACKAGE_VERSION ${LIBRARY_VERSION})
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libcos4-dev, libzmq5-dev | libzmq3-dev")
set(CPACK_DEBIAN_PACKAGE_CONFLICTS "libtango7-dev, libtango8-dev, libtango9-dev")
SET(CPACK_DEBIAN_PACKAGE_MAINTAINER "TANGO team <tango@esrf.fr>") #required
ENDIF()
include(CPack)
