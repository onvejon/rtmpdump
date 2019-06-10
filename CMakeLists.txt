cmake_minimum_required (VERSION 2.6)
project(rtmpdump)
set(VERSION v2.4)
option(BUILD_WITH_TEST "build with demo " OFF)
option(BUILD_WITH_OPENSSL "build with openssl " OFF)
option(BUILD_WITH_STATIC "build with static" ON)
option(BUILD_WITH_SHARD "build with shard" OFF)

add_definitions("-DRTMPDUMP_VERSION=\"${VERSION}\"" "-DNO_CRYPTO")

if(BUILD_WITH_OPENSSL)
find_package()
 # TODO: Add support for STATIC also.
find_package(OpenSSL REQUIRED)
message("OpenSSL include dir: ${OPENSSL_INCLUDE_DIR}")
message("OpenSSL libraries: ${OPENSSL_LIBRARIES}")
include_directories(${OPENSSL_INCLUDE_DIR})
list(APPEND LIB_LIST ${OPENSSL_LIBRARIES})
add_definitions("-DUSE_OPENSSL")
endif()

set(SRC_ROOT ${CMAKE_CURRENT_LIST_DIR})
include_directories(${SRC_ROOT}/librtmp)
include_directories(${SRC_ROOT})

file(GLOB LIBRTMP_HEADERS "${SRC_ROOT}/librtmp/*.h*" )
file(GLOB LIBRTMP_SRCS "${SRC_ROOT}/librtmp/*.c*")

set(LIB_NAME librtmp)

if(BUILD_WITH_STATIC)
option(BUILD_IMP_DISABLE " Disable IMP lib" ON)
if(BUILD_IMP_DISABLE)
    set(CompilerFlags
        CMAKE_CXX_FLAGS
        CMAKE_CXX_FLAGS_DEBUG
        CMAKE_CXX_FLAGS_RELEASE
        CMAKE_C_FLAGS
        CMAKE_C_FLAGS_DEBUG
        CMAKE_C_FLAGS_RELEASE
    )
        
    foreach(CompilerFlag ${CompilerFlags})
            string(REPLACE "/MD" "/MT" ${CompilerFlag} "${${CompilerFlag}}")
            string(REPLACE "/Ob0" "" ${CompilerFlag} "${${CompilerFlag}}")
            string(REPLACE "/TC" "" ${CompilerFlag} "${${CompilerFlag}}")
    endforeach()
endif()

    add_library(${LIB_NAME} STATIC 
	${LIBRTMP_HEADERS}
	${LIBRTMP_SRCS})
    install(TARGETS ${LIB_NAME}
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib )
endif()

if(BUILD_WITH_SHARD)
    
    if(BUILD_WITH_STATIC)
        set(LIB_NAME librtmp_shared)
    endif()

    add_library(${LIB_NAME} SHARED 
	${LIBRTMP_HEADERS}
	${LIBRTMP_SRCS})

    install(TARGETS ${LIB_NAME}
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib )
endif()

if(BUILD_WITH_TEST)


include_directories(${SRC_ROOT}/win32helpers)


file(GLOB WIN32HELPERS_HEADERS "${SRC_ROOT}/win32helpers/*.h*" )
file(GLOB WIN32HELPERS_SRCS "${SRC_ROOT}/win32helpers/*.c*")

if(UNIX)
	set(EXTRA_LIBS ${EXTRA_LIBS} stdc++ z pthread dl)
elseif(MSVC)
	 set(EXTRA_LIBS ${EXTRA_LIBS} ws2_32 winmm vfw32 wininet Userenv)
endif()

add_executable(rtmpdump ${CMAKE_CURRENT_LIST_DIR}/rtmpdump.c ${WIN32HELPERS_HEADERS} ${WIN32HELPERS_SRCS})
TARGET_LINK_LIBRARIES(rtmpdump ${LIB_NAME} ${EXTRA_LIBS} ${LIB_LIST})

add_executable(rtmpsuck 
${CMAKE_CURRENT_LIST_DIR}/rtmpsuck.c
${CMAKE_CURRENT_LIST_DIR}/thread.c
${WIN32HELPERS_HEADERS} ${WIN32HELPERS_SRCS}
)
TARGET_LINK_LIBRARIES(rtmpsuck ${LIB_NAME} ${EXTRA_LIBS} ${LIB_LIST})

add_executable(rtmpsrv
${CMAKE_CURRENT_LIST_DIR}/rtmpsrv.c
${CMAKE_CURRENT_LIST_DIR}/thread.c
 ${WIN32HELPERS_HEADERS} ${WIN32HELPERS_SRCS}
)
TARGET_LINK_LIBRARIES(rtmpsrv ${LIB_NAME} ${EXTRA_LIBS} ${LIB_LIST})

add_executable(rtmpgw 
${CMAKE_CURRENT_LIST_DIR}/rtmpgw.c
${CMAKE_CURRENT_LIST_DIR}/thread.c
 ${WIN32HELPERS_HEADERS} ${WIN32HELPERS_SRCS}
)
TARGET_LINK_LIBRARIES(rtmpgw ${LIB_NAME} ${EXTRA_LIBS} ${LIB_LIST})


endif()

install(FILES ${LIBRTMP_HEADERS} DESTINATION include )
