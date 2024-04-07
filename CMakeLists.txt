cmake_minimum_required(VERSION 3.20)

project(lua)

find_library(LIB_MATH m REQUIRED)

list(APPEND lua.core.sources
    lapi.c
    lcode.c
    lctype.c
    ldebug.c
    ldo.c
    ldump.c
    lfunc.c
    lgc.c
    llex.c
    lmem.c
    lobject.c
    lopcodes.c
    lparser.c
    lstate.c
    lstring.c
    ltable.c
    ltests.c
    ltm.c
    lundump.c
    lvm.c
    lzio.c
)

list(APPEND lua.lib.sources
    lbaselib.c
    lcorolib.c
    ldblib.c
    linit.c
    liolib.c
    lmathlib.c
    loadlib.c
    loslib.c
    lstrlib.c
    ltablib.c
    lutf8lib.c
    lauxlib.c
)

add_library(lua.core.obj OBJECT ${lua.core.sources})
add_library(lua.core.obj_pic OBJECT ${lua.core.sources})
set_property(TARGET lua.core.obj_pic PROPERTY POSITION_INDEPENDENT_CODE ON)

add_library(lua.lib.obj OBJECT ${lua.lib.sources})
add_library(lua.lib.obj_pic OBJECT ${lua.lib.sources})
set_property(TARGET lua.lib.obj_pic PROPERTY POSITION_INDEPENDENT_CODE ON)

add_library(lua.static STATIC)
set_property(TARGET lua.static PROPERTY OUTPUT_NAME "lua")
target_sources(lua.static PRIVATE
    $<TARGET_OBJECTS:lua.core.obj>
    $<TARGET_OBJECTS:lua.lib.obj>
)

add_library(lua.shared SHARED)
target_link_libraries(lua.shared PRIVATE ${LIB_MATH})
set_property(TARGET lua.shared PROPERTY OUTPUT_NAME "lua")
target_sources(lua.shared PRIVATE
    $<TARGET_OBJECTS:lua.core.obj_pic>
    $<TARGET_OBJECTS:lua.lib.obj_pic>
)

add_library(lua.exe.obj OBJECT lua.c)

add_executable(lua.exe $<TARGET_OBJECTS:lua.exe.obj>)
set_property(TARGET lua.exe PROPERTY OUTPUT_NAME "lua")
target_link_libraries(lua.exe PRIVATE lua.shared)

add_executable(lua.exe.static $<TARGET_OBJECTS:lua.exe.obj>)
set_property(TARGET lua.exe.static PROPERTY OUTPUT_NAME "lua_static")
target_link_libraries(lua.exe.static PRIVATE lua.static ${LIB_MATH})
