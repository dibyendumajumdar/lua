cmake_minimum_required(VERSION 2.8)
project(Lua)

enable_language(C)

option(STATIC_BUILD "Build static version of Lua, default is OFF" OFF)
option(LTESTS "Controls whether ltests are enabled in Debug mode" OFF)

if (STATIC_BUILD)
    message(STATUS "STATIC library build enabled")
else()
    message(STATUS "DYNAMIC library build enabled")
endif()

if (MSVC)
    set(CMAKE_C_FLAGS_DEBUG "/Od /D_DEBUG /MDd /Zi /RTC1 /EHsc")
    set(CMAKE_C_FLAGS_RELEASE "/DNDEBUG /O2 /MD /EHsc")
    set(CMAKE_C_FLAGS_RELWITHDEBINFO "/DNDEBUG /O2 /MD /Zi /EHsc")
    set(CMAKE_C_FLAGS_MINSIZEREL "/DNDEBUG /O2 /MD /EHsc")

    set(CMAKE_CXX_FLAGS "/EHsc")
    set(CMAKE_CXX_FLAGS_DEBUG "/Od /D_DEBUG /MDd /Zi /RTC1 /EHsc")
    set(CMAKE_CXX_FLAGS_RELEASE "/DNDEBUG /O2 /MD /EHsc")
    set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "/DNDEBUG /O2 /MD /Zi /EHsc")
    set(CMAKE_CXX_FLAGS_MINSIZEREL "/DNDEBUG /O2 /MD /EHsc")

    # Disable signed/unsigned int conversion warnings.
    add_definitions("/wd4018")
    # Disable warning about using struct/class for the same symobl.
    add_definitions("/wd4099")
    # Disable performance warning about int-to-bool conversion.
    add_definitions("/wd4800")
    # Disable signed/unsigned int conversion warnings.
    add_definitions("/wd4018")
    # Disable warning about the insecurity of using "std::copy".
    add_definitions("/wd4996")
    add_definitions("/wd4291")
    add_definitions("/wd4624")
    add_definitions("/wd4141")
    add_definitions("/DLUA_COMPAT_5_2")
    add_definitions("/DLUA_COMPAT_5_1")
endif ()


if ((CMAKE_C_COMPILER_ID MATCHES "GNU" OR CMAKE_C_COMPILER_ID MATCHES "Clang") AND NOT APPLE)
    if (NOT WIN32)
        # assume Linux
        set(OS_FLAGS "-DLUA_USE_LINUX")
    endif ()
    # set(SANITIZER_FLAGS "-fsanitize=address")
    set(LUA_COMPAT_FLAGS "-DLUA_COMPAT_5_2 -DLUA_COMPAT_5_1")
    set(CMAKE_C_FLAGS "-std=c99 -O2 -fomit-frame-pointer -Wall -Wextra -Winline ${LUA_COMPAT_FLAGS} ${OS_FLAGS}")
    set(CMAKE_C_FLAGS_DEBUG "${SANITIZER_FLAGS} -fno-omit-frame-pointer -std=c99 -O0 -g3 -Wall -Wextra ${LUA_COMPAT_FLAGS} ${OS_FLAGS}")
    set(CMAKE_C_FLAGS_RELEASE "-std=c99 -O2 -fomit-frame-pointer -Wall -Wextra -Winline ${LUA_COMPAT_FLAGS} ${OS_FLAGS}")
    set(CMAKE_C_FLAGS_RELWITHDEBINFO "${SANITIZER_FLAGS} -std=c99 -O1 -g3 -Wall -Wextra ${LUA_COMPAT_FLAGS} ${OS_FLAGS}")
    set(CMAKE_CXX_FLAGS "-fno-rtti -O2 -fomit-frame-pointer -Wall -Wno-sign-compare -Winline -std=c++11 -fno-exceptions ${LUA_COMPAT_FLAGS} ${OS_FLAGS}")
    set(CMAKE_CXX_FLAGS_RELEASE "-fno-rtti -O2 -fomit-frame-pointer -Wall -Wno-sign-compare -Winline -std=c++11 -fno-exceptions ${LUA_COMPAT_FLAGS} ${OS_FLAGS}")
    set(CMAKE_CXX_FLAGS_DEBUG "${SANITIZER_FLAGS} -fno-omit-frame-pointer -fno-rtti -O0 -g3 -Wall -Wno-sign-compare -std=c++11 -fno-exceptions ${LUA_COMPAT_FLAGS} ${OS_FLAGS}")
    set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${SANITIZER_FLAGS} -fno-rtti -O1 -g3 -Wall -Wno-sign-compare -std=c++11 -fno-exceptions ${LUA_COMPAT_FLAGS} ${OS_FLAGS}")
endif ()

if (APPLE)
    set(LUA_COMPAT_FLAGS "-DLUA_COMPAT_5_2 -DLUA_COMPAT_5_1")
    set(CMAKE_C_FLAGS "-std=c99 -O3 -Wall -Wextra ${LUA_COMPAT_FLAGS} -DLUA_USE_MACOSX")
    set(CMAKE_C_FLAGS_DEBUG "-std=c99 -O0 -g3 -Wall -Wextra ${LUA_COMPAT_FLAGS} -DLUA_USE_MACOSX")
    set(CMAKE_C_FLAGS_RELEASE "-std=c99 -O3 -Wall -Wextra ${LUA_COMPAT_FLAGS} -DLUA_USE_MACOSX")
    set(CMAKE_C_FLAGS_RELWITHDEBINFO "-std=c99 -O1 -g3 -Wall -Wextra ${LUA_COMPAT_FLAGS} -DLUA_USE_MACOSX")
    set(CMAKE_CXX_FLAGS "-O3 -Wall -fno-rtti -Wno-sign-compare -std=c++11 -fno-exceptions ${LUA_COMPAT_FLAGS} -DLUA_USE_MACOSX")
    set(CMAKE_CXX_FLAGS_RELEASE "-fno-rtti -O3 -Wall -Wno-sign-compare -std=c++11 -fno-exceptions ${LUA_COMPAT_FLAGS} -DLUA_USE_MACOSX")
    set(CMAKE_CXX_FLAGS_DEBUG "-fno-rtti -O0 -g3 -Wall -Wno-sign-compare -std=c++11 -fno-exceptions ${LUA_COMPAT_FLAGS} -DLUA_USE_MACOSX")
    set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-fno-rtti -O1 -g3 -Wall -Wno-sign-compare -std=c++11 -fno-exceptions ${LUA_COMPAT_FLAGS} -DLUA_USE_MACOSX")
endif ()

include_directories("${PROJECT_SOURCE_DIR}")
if (CMAKE_BUILD_TYPE STREQUAL "DEBUG" AND LTESTS)
    message(STATUS "Enabling Lua extended test harness 'ltests'")
    add_definitions(-DLUA_USER_H="ltests.h")
endif ()

# define the lua core source files
set(LUA_CORE_SRCS lapi.c lcode.c lctype.c ldebug.c ldo.c ldump.c
        lfunc.c lgc.c llex.c lmem.c lobject.c lopcodes.c
        lparser.c lstate.c lstring.c ltable.c ltm.c lundump.c
        lvm.c lzio.c)
if (LTESTS)
    append(LUA_CORE_SRCS ltests.c)
endif()

# define the lua lib source files
set(LUA_LIB_SRCS lauxlib.c lbaselib.c lbitlib.c lcorolib.c ldblib.c liolib.c
        lmathlib.c loslib.c ltablib.c lstrlib.c loadlib.c linit.c lutf8lib.c bit.c)

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

if (NOT STATIC_BUILD)
    set(LIBLUA_BUILD_TYPE SHARED)
endif()

set(LIB_NAME lua53)
#Main library
add_library(${LIB_NAME} ${LIBLUA_BUILD_TYPE} 
        ${ALL_HEADERS}
        ${LUA_LIB_SRCS} 
        ${LUA_CORE_SRCS}
        )
if (NOT STATIC_BUILD)
    if (WIN32)
        # enable DLL export
        set_target_properties(${LIB_NAME} PROPERTIES DEFINE_SYMBOL "LUA_BUILD_AS_DLL")
    else()
        set_target_properties(${LIB_NAME} PROPERTIES PREFIX "")
    endif ()
endif()
target_link_libraries(${LIB_NAME} ${EXTRA_LIBRARIES})

# Lua executable
add_executable(lua lua.c)
target_link_libraries(lua ${LIB_NAME})

install(FILES ${LUA_HEADERS}
        DESTINATION include)
install(TARGETS ${LIB_NAME} lua
        RUNTIME DESTINATION bin
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib)
