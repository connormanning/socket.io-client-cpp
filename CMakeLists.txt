cmake_minimum_required(VERSION 3.1.0 FATAL_ERROR)
PROJECT(sioclient)

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/modules" ${CMAKE_MODULE_PATH})

if(NOT CMAKE_BUILD_TYPE )
MESSAGE(STATUS "not define build type, set to release" )
set(CMAKE_BUILD_TYPE Release )
elseif(NOT (${CMAKE_BUILD_TYPE} STREQUAL "Release" OR ${CMAKE_BUILD_TYPE} STREQUAL "Debug" ))
MESSAGE(SEND_ERROR "CMAKE_BUILD_TYPE must be either Release or Debug")
return()
endif()

set(BOOST_VER "1.55.0" CACHE STRING "boost version" )

set(Boost_USE_STATIC_LIBS ON)
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_RUNTIME OFF)
find_package(Boost ${BOOST_VER} REQUIRED COMPONENTS system date_time random)

aux_source_directory(${CMAKE_CURRENT_LIST_DIR}/src ALL_SRC)
aux_source_directory(${CMAKE_CURRENT_LIST_DIR}/src/internal ALL_SRC)
file(GLOB ALL_HEADERS ${CMAKE_CURRENT_LIST_DIR}/src/*.h )
set(SIO_INCLUDEDIR ${CMAKE_CURRENT_LIST_DIR})

add_library(sioclient STATIC ${ALL_SRC})
target_include_directories(sioclient PRIVATE ${Boost_INCLUDE_DIRS}
    ${CMAKE_CURRENT_LIST_DIR}/src
    ${CMAKE_CURRENT_LIST_DIR}/lib/websocketpp
    ${CMAKE_CURRENT_LIST_DIR}/lib/rapidjson/include
)

set_property(TARGET sioclient PROPERTY CXX_STANDARD 11)
set_property(TARGET sioclient PROPERTY CXX_STANDARD_REQUIRED ON)
target_link_libraries(sioclient PRIVATE ${Boost_LIBRARIES})
list(APPEND TARGET_LIBRARIES sioclient)

find_package(OpenSSL)
if(OPENSSL_FOUND)
add_library(sioclient_tls STATIC ${ALL_SRC})
target_include_directories(sioclient_tls PRIVATE ${Boost_INCLUDE_DIRS}
    ${CMAKE_CURRENT_LIST_DIR}/src
    ${CMAKE_CURRENT_LIST_DIR}/lib/websocketpp
    ${CMAKE_CURRENT_LIST_DIR}/lib/rapidjson/include
    ${OPENSSL_INCLUDE_DIR}
)

set_property(TARGET sioclient_tls PROPERTY CXX_STANDARD 11)
set_property(TARGET sioclient_tls PROPERTY CXX_STANDARD_REQUIRED ON)
target_link_libraries(sioclient_tls PRIVATE ${Boost_LIBRARIES} ${OPENSSL_LIBRARIES} )
target_compile_definitions(sioclient_tls PRIVATE -DSIO_TLS)
list(APPEND TARGET_LIBRARIES sioclient_tls)

endif()

include(GNUInstallDirs)

install(FILES ${ALL_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/sioclient
)

install(TARGETS ${TARGET_LIBRARIES}
    DESTINATION ${CMAKE_INSTALL_LIBDIR}
    EXPORT sioclient-targets
)

export(EXPORT sioclient-targets FILE "${PROJECT_BINARY_DIR}/sioclient-targets.cmake")
configure_file(cmake/sioclient-config.cmake.in "${PROJECT_BINARY_DIR}/sioclient-config.cmake" @ONLY)
install(EXPORT sioclient-targets FILE sioclient-targets.cmake NAMESPACE SioClient:: DESTINATION lib/cmake/sioclient)
install(FILES "${PROJECT_BINARY_DIR}/sioclient-config.cmake" DESTINATION lib/cmake/sioclient)

