cmake_minimum_required(VERSION 3.12)
project(Lua)

enable_language(C)

include_directories("${PROJECT_SOURCE_DIR}")
set(LUA_CORE_SRCS lapi.c lcode.c lctype.c ldebug.c ldo.c ldump.c
        lfunc.c lgc.c llex.c lmem.c lobject.c lopcodes.c
        lparser.c lstate.c lstring.c ltable.c ltm.c lundump.c
        lvm.c lzio.c)
set(LUA_LIB_SRCS lauxlib.c lbaselib.c lcorolib.c ldblib.c liolib.c
        lmathlib.c loslib.c ltablib.c lstrlib.c loadlib.c linit.c lutf8lib.c)
file(GLOB ALL_HEADERS "${PROJECT_SOURCE_DIR}/*.h")
set(LUA_HEADERS lua.h luaconf.h lualib.h lauxlib.h)
if (MSVC OR APPLE)
    source_group("Lua Headers" FILES ${ALL_HEADERS})
    source_group("Lua Source Files" FILES ${LUA_CORE_SRCS} ${LUA_LIB_SRCS})
    if (APPLE)
        set(EXTRA_LIBRARIES m readline)
    endif ()
elseif (NOT WIN32)
    # On Linux we need to link libdl to get access to
    # functions like dlopen()
    # ubsan can be added to get -fsanitize=undefined
    set(EXTRA_LIBRARIES m dl readline)
endif ()
set(LIBLUA_BUILD_TYPE STATIC)
set(LIB_NAME lua54)
add_library(${LIB_NAME} ${LIBLUA_BUILD_TYPE} 
        ${ALL_HEADERS}
        ${LUA_LIB_SRCS} 
        ${LUA_CORE_SRCS}
        )
target_link_libraries(${LIB_NAME} ${EXTRA_LIBRARIES})
add_executable(lua lua.c)
target_link_libraries(lua ${LIB_NAME})
