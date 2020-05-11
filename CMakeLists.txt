project(lua C)
cmake_minimum_required(VERSION 3.15)

if(NOT DEFINED LUA_STDLIB)
define(LUA_STDLIB coro db io math os str tab utf8)
endif()

set(CMAKE_C_DIALECT 99)

include_directories(.)

add_library(lua_objects OBJECT lzio.c lctype.c lopcodes.c lmem.c lundump.c ldump.c lstate.c lgc.c llex.c lcode.c lparser.c ldebug.c lfunc.c lobject.c ltm.c lstring.c ltable.c ldo.c lvm.c lapi.c  lzio.h lctype.h lopcodes.h lmem.h lundump.h ldump.h lstate.h lgc.h llex.h lcode.h lparser.h ldebug.h lfunc.h lobject.h ltm.h lstring.h ltable.h ldo.h lvm.h lapi.h)

if(LUA_STDLIB)
    add_library(luastd OBJECT loadlib.c linit.c lbaselib.c)
    target_link_libraries(luastd lua_objects)

    foreach(stdlib ${LUA_STDLIB})
        add_library(luamod_${stdlib} OBJECT l${stdlib}lib.c)
        target_link_libraries(luastd luamod_${stdlib})
    endforeach()
endif()

add_library(lua_all OBJECTS ua_objects $<$<BOOL:LUA_STDLIB>:luastd>)

add_library(lua-static STATIC $<TARGET_OBJECTS:lua_all>)
set_target_properties(lua-static PROPERTIES OUTPUT_NAME lua
        $<$<BOOL:LUA_BUILD_PIC>:POSITION_INDEPENDANT_CODE True>)

if(LUA_BUILD_LUAC)
    add_executable(luac luac.c)
    target_link_libraries(luac lua_objects)
endif()

if(LUA_BUILD_INTERP)
    add_executable(lua lua.c)
    target_link_libraries(lua lua_all)
endif()
