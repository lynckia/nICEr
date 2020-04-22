cmake_minimum_required(VERSION 2.6)

find_package(PkgConfig)
#functions
function(test_lib LIB_NAME)
  if (${LIB_NAME} MATCHES "^.*-NOTFOUND")
    message(FATAL_ERROR "lib not found: " ${LIB_NAME} " check README")
    return()
  endif(${LIB_NAME} MATCHES "^.*-NOTFOUND")
endfunction(test_lib)

include(ExternalProject)

project(nicer)

set(NICER_BUILD "${CMAKE_CURRENT_BINARY_DIR}")

set(CMAKE_C_OUTPUT_EXTENSION_REPLACE 1)
set(CMAKE_CXX_OUTPUT_EXTENSION_REPLACE 1)

set (NICER_SOURCE ${CMAKE_CURRENT_SOURCE_DIR})

list(APPEND nicer__c_srcs
    "${NICER_SOURCE}/src/crypto/nr_crypto.c"
    "${NICER_SOURCE}/src/ice/ice_candidate.c"
    "${NICER_SOURCE}/src/ice/ice_candidate_pair.c"
    "${NICER_SOURCE}/src/ice/ice_codeword.c"
    "${NICER_SOURCE}/src/ice/ice_component.c"
    "${NICER_SOURCE}/src/ice/ice_ctx.c"
    "${NICER_SOURCE}/src/ice/ice_media_stream.c"
    "${NICER_SOURCE}/src/ice/ice_parser.c"
    "${NICER_SOURCE}/src/ice/ice_peer_ctx.c"
    "${NICER_SOURCE}/src/ice/ice_socket.c"
    "${NICER_SOURCE}/src/net/nr_resolver.c"
    "${NICER_SOURCE}/src/net/nr_socket_wrapper.c"
    "${NICER_SOURCE}/src/net/nr_socket.c"
    "${NICER_SOURCE}/src/net/nr_socket_local.c"
    "${NICER_SOURCE}/src/net/nr_socket_multi_tcp.c"
    "${NICER_SOURCE}/src/net/transport_addr.c"
    "${NICER_SOURCE}/src/net/transport_addr_reg.c"
    "${NICER_SOURCE}/src/net/local_addr.c"
    "${NICER_SOURCE}/src/net/nr_interface_prioritizer.c"
    "${NICER_SOURCE}/src/stun/addrs.c"
    "${NICER_SOURCE}/src/stun/ifaddrs-android.c"
    "${NICER_SOURCE}/src/stun/nr_socket_turn.c"
    "${NICER_SOURCE}/src/stun/nr_socket_buffered_stun.c"
    "${NICER_SOURCE}/src/stun/stun_build.c"
    "${NICER_SOURCE}/src/stun/stun_client_ctx.c"
    "${NICER_SOURCE}/src/stun/stun_codec.c"
    "${NICER_SOURCE}/src/stun/stun_hint.c"
    "${NICER_SOURCE}/src/stun/stun_msg.c"
    "${NICER_SOURCE}/src/stun/stun_proc.c"
    "${NICER_SOURCE}/src/stun/stun_server_ctx.c"
    "${NICER_SOURCE}/src/stun/stun_util.c"
    "${NICER_SOURCE}/src/stun/turn_client_ctx.c"
    "${NICER_SOURCE}/src/util/cb_args.c"
    "${NICER_SOURCE}/src/util/ice_util.c")

list(APPEND nicer__other_srcs
    "${NICER_SOURCE}/src/crypto/nr_crypto.h"
    "${NICER_SOURCE}/src/ice/ice_candidate.h"
    "${NICER_SOURCE}/src/ice/ice_candidate_pair.h"
    "${NICER_SOURCE}/src/ice/ice_codeword.h"
    "${NICER_SOURCE}/src/ice/ice_component.h"
    "${NICER_SOURCE}/src/ice/ice_ctx.h"
    "${NICER_SOURCE}/src/ice/ice_handler.h"
    "${NICER_SOURCE}/src/ice/ice_media_stream.h"
    "${NICER_SOURCE}/src/ice/ice_peer_ctx.h"
    "${NICER_SOURCE}/src/ice/ice_reg.h"
    "${NICER_SOURCE}/src/ice/ice_socket.h"
    "${NICER_SOURCE}/src/net/nr_resolver.h"
    "${NICER_SOURCE}/src/net/nr_socket_wrapper.h"
    "${NICER_SOURCE}/src/net/nr_socket.h"
    "${NICER_SOURCE}/src/net/nr_socket_local.h"
    "${NICER_SOURCE}/src/net/nr_socket_multi_tcp.h"
    "${NICER_SOURCE}/src/net/transport_addr.h"
    "${NICER_SOURCE}/src/net/transport_addr_reg.h"
    "${NICER_SOURCE}/src/net/local_addr.h"
    "${NICER_SOURCE}/src/net/nr_interface_prioritizer.h"
    "${NICER_SOURCE}/src/stun/addrs.h"
    "${NICER_SOURCE}/src/stun/ifaddrs-android.h"
    "${NICER_SOURCE}/src/stun/nr_socket_turn.h"
    "${NICER_SOURCE}/src/stun/nr_socket_buffered_stun.h"
    "${NICER_SOURCE}/src/stun/stun.h"
    "${NICER_SOURCE}/src/stun/stun_build.h"
    "${NICER_SOURCE}/src/stun/stun_client_ctx.h"
    "${NICER_SOURCE}/src/stun/stun_codec.h"
    "${NICER_SOURCE}/src/stun/stun_hint.h"
    "${NICER_SOURCE}/src/stun/stun_msg.h"
    "${NICER_SOURCE}/src/stun/stun_proc.h"
    "${NICER_SOURCE}/src/stun/stun_reg.h"
    "${NICER_SOURCE}/src/stun/stun_server_ctx.h"
    "${NICER_SOURCE}/src/stun/stun_util.h"
    "${NICER_SOURCE}/src/stun/turn_client_ctx.h"
    "${NICER_SOURCE}/src/util/cb_args.h"
    "${NICER_SOURCE}/src/util/ice_util.h")

add_library(nicer STATIC ${nicer__c_srcs} ${nicer__other_srcs})

# OPENSSL
find_library(SSL ssl HINTS "${THIRD_PARTY_LIB}")
test_lib(${SSL})
find_library(CRYPTO crypto HINTS "${THIRD_PARTY_LIB}")
test_lib(${CRYPTO})

IF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  set (ADDITIONAL_CFLAGS "")
ELSE()
  set (ADDITIONAL_CFLAGS "-DUSE_EPOLL=1")
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

set(NRAPPKIT_BUILD "${CMAKE_CURRENT_BINARY_DIR}/third_party/nrappkit")
ExternalProject_Add(project_nrappkit
  GIT_REPOSITORY "https://github.com/lynckia/nrappkit.git"
  GIT_TAG "1.5"
  PREFIX ${NRAPPKIT_BUILD}
  CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:PATH=${NRAPPKIT_BUILD} -DCMAKE_C_FLAGS="${ADDITIONAL_CFLAGS}" -DTHIRD_PARTY_LIB=${THIRD_PARTY_LIB}
  DOWNLOAD_DIR "${NRAPPKIT_BUILD}/src"
  INSTALL_DIR "${NRAPPKIT_BUILD}/out"
)
set(NRAPPKIT_INCLUDE "${CMAKE_CURRENT_BINARY_DIR}/third_party/nrappkit/include")

add_library(nrappkit STATIC IMPORTED)
set_property(TARGET nrappkit PROPERTY IMPORTED_LOCATION ${NRAPPKIT_BUILD}/src/project_nrappkit-build/nrappkit/libnrappkit.a)

add_dependencies(nrappkit project_nrappkit)
add_dependencies(nicer nrappkit)

set_target_properties(nicer PROPERTIES EXCLUDE_FROM_ALL "FALSE")

set_source_files_properties(${nicer__other_srcs} PROPERTIES HEADER_FILE_ONLY "TRUE")

set_target_properties(nicer PROPERTIES ARCHIVE_OUTPUT_DIRECTORY "nicer")
set_target_properties(nicer PROPERTIES PREFIX "lib")
set_target_properties(nicer PROPERTIES ARCHIVE_OUTPUT_NAME "nicer")
set_target_properties(nicer PROPERTIES SUFFIX ".a")

set_source_files_properties(nicer/libnicer.a PROPERTIES GENERATED "TRUE")

list(APPEND nicer__include_dirs
  "${NRAPPKIT_INCLUDE}/event"
  "${NRAPPKIT_INCLUDE}/log"
  "${NRAPPKIT_INCLUDE}/plugin"
  "${NRAPPKIT_INCLUDE}/registry"
  "${NRAPPKIT_INCLUDE}/share"
  "${NRAPPKIT_INCLUDE}/stats"
  "${NRAPPKIT_INCLUDE}/util"
  "${NRAPPKIT_INCLUDE}/util/libekr"
  "${NRAPPKIT_INCLUDE}/port/generic/include"
  "${NRAPPKIT_INCLUDE}/port/generic/include/sys"
  "${NICER_SOURCE}/src/crypto"
  "${NICER_SOURCE}/src/ice"
  "${NICER_SOURCE}/src/net"
  "${NICER_SOURCE}/src/stun"
  "${NICER_SOURCE}/src/util")

target_include_directories(nicer PUBLIC ${nicer__include_dirs})

IF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  set (NICER_OS DARWIN)
  list(APPEND nicer__include_dirs "${NRAPPKIT_INCLUDE}/port/darwin/include"
  "${NRAPPKIT_INCLUDE}/port/darwin/include/sys")
ELSE()
  set (NICER_OS LINUX)
  list(APPEND nicer__include_dirs "${NRAPPKIT_INCLUDE}/port/linux/include"
  "${NRAPPKIT_INCLUDE}/port/linux/include/sys")
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

set_target_properties(nicer PROPERTIES COMPILE_FLAGS "-g -fPIC -Wall -Wno-parentheses -Wno-strict-prototypes -Wmissing-prototypes -DUSE_TURN -DUSE_ICE -DUSE_RFC_3489_BACKWARDS_COMPATIBLE -DUSE_STUND_0_96 -DUSE_STUN_PEDANTIC -Drestrict=__restrict__ -DNO_REG_RPC=1 -DHAVE_VFPRINTF=1 -DRETSIGTYPE=void -DNEW_STDIO -DHAVE_STRDUP=1 -DHAVE_STRLCPY=1 -DHAVE_LIBM=1 -DHAVE_SYS_TIME_H=1 -DTIME_WITH_SYS_TIME_H=1 -D${NICER_OS} -DNOLINUXIF -D__UNUSED__=\"__attribute__((unused))\"")

target_link_libraries(nicer PUBLIC ${SSL} ${CRYPTO} PRIVATE nrappkit)


list(APPEND nicer__include_public_dirs
  "${NRAPPKIT_INCLUDE}/event"
  "${NRAPPKIT_INCLUDE}/log"
  "${NRAPPKIT_INCLUDE}/plugin"
  "${NRAPPKIT_INCLUDE}/registry"
  "${NRAPPKIT_INCLUDE}/share"
  "${NRAPPKIT_INCLUDE}/stats"
  "${NRAPPKIT_INCLUDE}/util"
  "${NRAPPKIT_INCLUDE}/port"
  "${NICER_SOURCE}/src/crypto"
  "${NICER_SOURCE}/src/ice"
  "${NICER_SOURCE}/src/net"
  "${NICER_SOURCE}/src/stun"
  "${NICER_SOURCE}/src/util")

set_property(TARGET nicer APPEND PROPERTY INCLUDE_DIRECTORIES ${nicer__include_dirs})

install(TARGETS nicer
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib)
install(DIRECTORY ${nicer__include_public_dirs} DESTINATION include  FILES_MATCHING PATTERN "**.h")
