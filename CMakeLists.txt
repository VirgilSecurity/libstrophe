
cmake_minimum_required(VERSION 3.11 FATAL_ERROR)

project(strophe VERSION 0.1.0 LANGUAGES C)

#
#   libstrophe options
#
option(USE_GNUTLS "Use gnutls" OFF)
option(USE_DUMMYTLS "Use dummy tls" OFF)
option(USE_OPENSSL "Use openssl for tls" ON)
option(USE_WINSSL "Use windows ssl for tls" OFF)

option(USE_LIBXML2 "Use libxml2 parser" ON)
option(USE_LIBEXPAT "Use expat parser" OFF)

# Build shared library if defined
set (BUILD_SHARED_LIBS OFF CACHE BOOL "Force to create shared libraries")

#
#   Strophe library
#
add_library(strophe)

target_sources(strophe
        PRIVATE
        # Headers

        ${CMAKE_CURRENT_LIST_DIR}/src/common.h
        ${CMAKE_CURRENT_LIST_DIR}/src/hash.h
        ${CMAKE_CURRENT_LIST_DIR}/src/md5.h
        ${CMAKE_CURRENT_LIST_DIR}/src/ostypes.h
        ${CMAKE_CURRENT_LIST_DIR}/src/parser.h
        ${CMAKE_CURRENT_LIST_DIR}/src/rand.h
        ${CMAKE_CURRENT_LIST_DIR}/src/resolver.h
        ${CMAKE_CURRENT_LIST_DIR}/src/sasl.h
        ${CMAKE_CURRENT_LIST_DIR}/src/scram.h
        ${CMAKE_CURRENT_LIST_DIR}/src/sha.h
        ${CMAKE_CURRENT_LIST_DIR}/src/sha1.h
        ${CMAKE_CURRENT_LIST_DIR}/src/sha256.h
        ${CMAKE_CURRENT_LIST_DIR}/src/sha512.h
        ${CMAKE_CURRENT_LIST_DIR}/src/snprintf.h
        ${CMAKE_CURRENT_LIST_DIR}/src/sock.h
        ${CMAKE_CURRENT_LIST_DIR}/src/tls.h
        ${CMAKE_CURRENT_LIST_DIR}/src/util.h

        # Sources
        ${CMAKE_CURRENT_LIST_DIR}/src/auth.c
        ${CMAKE_CURRENT_LIST_DIR}/src/conn.c
        ${CMAKE_CURRENT_LIST_DIR}/src/crypto.c
        ${CMAKE_CURRENT_LIST_DIR}/src/ctx.c
        ${CMAKE_CURRENT_LIST_DIR}/src/event.c
        ${CMAKE_CURRENT_LIST_DIR}/src/handler.c
        ${CMAKE_CURRENT_LIST_DIR}/src/hash.c
        ${CMAKE_CURRENT_LIST_DIR}/src/jid.c
        ${CMAKE_CURRENT_LIST_DIR}/src/md5.c
        ${CMAKE_CURRENT_LIST_DIR}/src/rand.c
        ${CMAKE_CURRENT_LIST_DIR}/src/resolver.c
        ${CMAKE_CURRENT_LIST_DIR}/src/sasl.c
        ${CMAKE_CURRENT_LIST_DIR}/src/scram.c
        ${CMAKE_CURRENT_LIST_DIR}/src/sha1.c
        ${CMAKE_CURRENT_LIST_DIR}/src/sha256.c
        ${CMAKE_CURRENT_LIST_DIR}/src/sha512.c
        ${CMAKE_CURRENT_LIST_DIR}/src/snprintf.c
        ${CMAKE_CURRENT_LIST_DIR}/src/sock.c
        ${CMAKE_CURRENT_LIST_DIR}/src/stanza.c
        ${CMAKE_CURRENT_LIST_DIR}/src/util.c
        ${CMAKE_CURRENT_LIST_DIR}/src/uuid.c

        $<$<BOOL:${USE_LIBEXPAT}>:${CMAKE_CURRENT_LIST_DIR}/src/parser_expat.c>
        $<$<BOOL:${USE_LIBXML2}>:${CMAKE_CURRENT_LIST_DIR}/src/parser_libxml2.c>

        $<$<BOOL:${USE_WINSSL}>:${CMAKE_CURRENT_LIST_DIR}/src/tls_schannel.c>
        $<$<BOOL:${USE_OPENSSL}>:${CMAKE_CURRENT_LIST_DIR}/src/tls_openssl.c>
        $<$<BOOL:${USE_GNUTLS}>:${CMAKE_CURRENT_LIST_DIR}/src/tls_gnutls.c>
        )

#
#   Find required libraries and packages
#
if(USE_LIBEXPAT)
    find_package(LibExpat REQUIRED)
endif()

if(USE_LIBXML2)
    find_package(LibXml2 REQUIRED)
endif()

if(USE_OPENSSL)
    find_package(OpenSSL REQUIRED)
endif()

if(USE_GNUTLS)
    find_package(GnuTLS REQUIRED)
endif()

if(NOT WIN32)
    find_library(RESOLV_LIBRARY NAMES resolv)
endif()

#
#   Enable position independent code
#
set_property(TARGET strophe PROPERTY POSITION_INDEPENDENT_CODE TRUE)

#
#   -Wall -Werror
#
if (NOT MINGW)
    set_target_properties(strophe PROPERTIES COMPILE_FLAGS "-Wall -Werror")
endif()

target_link_libraries(strophe
        PUBLIC
        $<$<BOOL:${USE_LIBEXPAT}>:${LIBEXPAT_LIBRARIES}>
        $<$<BOOL:${USE_LIBXML2}>:${LIBXML2_LIBRARIES}>
#        $<$<BOOL:${USE_OPENSSL}>:${OPENSSL_LIBRARIES}>
        ${OPENSSL_LIBRARIES}
        $<$<BOOL:${USE_GNUTLS}>:${GNUTLS_LIBRARIES}>
        $<$<NOT:$<BOOL:${WIN32}>>:${RESOLV_LIBRARY}>
        $<$<BOOL:${WIN32}>:ws2_32>
        $<$<BOOL:${WIN32}>:winmm>
        )
#
#   Common include directories
#
target_include_directories(strophe
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/src>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>

        PRIVATE
        $<BUILD_INTERFACE:$<$<BOOL:${USE_LIBEXPAT}>:${LIBEXPAT_INCLUDE_DIR}>>
        $<BUILD_INTERFACE:$<$<BOOL:${USE_LIBXML2}>:${LIBXML2_INCLUDE_DIR}>>
        $<BUILD_INTERFACE:$<$<BOOL:${USE_LIBXML2}>:${LIBXML2_INCLUDE_DIR}/../>>
        $<BUILD_INTERFACE:$<$<BOOL:${USE_OPENSSL}>:${OPENSSL_INCLUDE_DIR}>>
        $<BUILD_INTERFACE:$<$<BOOL:${USE_GNUTLS}>:${GNUTLS_INCLUDE_DIR}>>
        $<BUILD_INTERFACE:${VIRGIL_IOT_CONFIG_DIRECTORY}>
        )

install(TARGETS strophe
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        )

install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/include/virgil
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        )

#
#   Add examples
#
add_subdirectory(examples)
