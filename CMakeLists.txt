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

add_library(lua.core.obj OBJECT ${lua.core.sources})
add_library(lua.core.obj_pic OBJECT ${lua.core.sources})
set_property(TARGET lua.core.obj_pic PROPERTY POSITION_INDEPENDENT_CODE ON)

# CORE AS C++
foreach(CORE_SOURCE ${lua.core.sources})
    set(CORE_SOURCE_CPP_LOCATION "${CMAKE_CURRENT_BINARY_DIR}/generated/core_cpp/${CORE_SOURCE}.cpp")
    message("Generating ${CORE_SOURCE} -> ${CORE_SOURCE_CPP_LOCATION}")
    file(GENERATE OUTPUT ${CORE_SOURCE_CPP_LOCATION} CONTENT "#include \"${CORE_SOURCE}\"")
    list(APPEND lua.core.cpp_sources ${CORE_SOURCE_CPP_LOCATION})
endforeach()
add_library(lua.core.cpp_obj OBJECT ${lua.core.cpp_sources})
target_include_directories(lua.core.cpp_obj PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
add_library(lua.core.cpp_obj_pic OBJECT ${lua.core.cpp_sources})
target_include_directories(lua.core.cpp_obj_pic PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
set_property(TARGET lua.core.cpp_obj_pic PROPERTY POSITION_INDEPENDENT_CODE ON)


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

add_library(lua.lib.obj OBJECT ${lua.lib.sources})
add_library(lua.lib.obj_pic OBJECT ${lua.lib.sources})
set_property(TARGET lua.lib.obj_pic PROPERTY POSITION_INDEPENDENT_CODE ON)


# LIB AS C++
foreach(LIB_SOURCE ${lua.lib.sources})
    set(LIB_SOURCE_CPP_LOCATION "${CMAKE_CURRENT_BINARY_DIR}/generated/lib_cpp/${LIB_SOURCE}.cpp")
    message("Generating ${LIB_SOURCE} -> ${LIB_SOURCE_CPP_LOCATION}")
    file(GENERATE OUTPUT ${LIB_SOURCE_CPP_LOCATION} CONTENT "#include \"${LIB_SOURCE}\"")
    list(APPEND lua.lib.cpp_sources ${LIB_SOURCE_CPP_LOCATION})
endforeach()
add_library(lua.lib.cpp_obj OBJECT ${lua.lib.cpp_sources})
target_include_directories(lua.lib.cpp_obj PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
add_library(lua.lib.cpp_obj_pic OBJECT ${lua.lib.cpp_sources})
target_include_directories(lua.lib.cpp_obj_pic PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
set_property(TARGET lua.lib.cpp_obj_pic PROPERTY POSITION_INDEPENDENT_CODE ON)


add_library(lua.static STATIC)
set_property(TARGET lua.static PROPERTY OUTPUT_NAME "lua")
target_sources(lua.static PRIVATE
    $<TARGET_OBJECTS:lua.core.obj>
    $<TARGET_OBJECTS:lua.lib.obj>
)

add_library(lua.cpp.static STATIC)
set_property(TARGET lua.cpp.static PROPERTY OUTPUT_NAME "lua_cxx")
target_sources(lua.cpp.static PRIVATE
    $<TARGET_OBJECTS:lua.core.cpp_obj>
    $<TARGET_OBJECTS:lua.lib.cpp_obj>
)

add_library(lua.shared SHARED)
target_link_libraries(lua.shared PRIVATE ${LIB_MATH})
set_property(TARGET lua.shared PROPERTY OUTPUT_NAME "lua")
target_sources(lua.shared PRIVATE
    $<TARGET_OBJECTS:lua.core.obj_pic>
    $<TARGET_OBJECTS:lua.lib.obj_pic>
)

add_library(lua.cpp.shared SHARED)
target_link_libraries(lua.cpp.shared PRIVATE ${LIB_MATH})
set_property(TARGET lua.cpp.shared PROPERTY OUTPUT_NAME "lua_cxx")
target_sources(lua.cpp.shared PRIVATE
    $<TARGET_OBJECTS:lua.core.cpp_obj_pic>
    $<TARGET_OBJECTS:lua.lib.cpp_obj_pic>
)


# C EXECUTABLES
add_library(lua.exe.obj OBJECT lua.c)

add_executable(lua.exe $<TARGET_OBJECTS:lua.exe.obj>)
set_property(TARGET lua.exe PROPERTY OUTPUT_NAME "lua")
target_link_libraries(lua.exe PRIVATE lua.shared)

add_executable(lua.exe.static $<TARGET_OBJECTS:lua.exe.obj>)
set_property(TARGET lua.exe.static PROPERTY OUTPUT_NAME "lua_static")
target_link_libraries(lua.exe.static PRIVATE lua.static ${LIB_MATH})



# CPP EXECUTABLES
set(LUA_SOURCE_CPP_LOCATION "${CMAKE_CURRENT_BINARY_DIR}/generated/lua.c.cpp")
message("Generating lua.c -> ${LUA_SOURCE_CPP_LOCATION}")
file(GENERATE OUTPUT ${LUA_SOURCE_CPP_LOCATION} CONTENT "#include \"lua.c\"")
add_library(lua.cpp_exe.obj OBJECT ${LUA_SOURCE_CPP_LOCATION})
target_include_directories(lua.cpp_exe.obj PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})

add_executable(lua.cpp_exe $<TARGET_OBJECTS:lua.cpp_exe.obj>)
set_property(TARGET lua.cpp_exe PROPERTY OUTPUT_NAME "lua_cxx")
target_link_libraries(lua.cpp_exe PRIVATE lua.cpp.shared)

add_executable(lua.cpp_exe.static $<TARGET_OBJECTS:lua.cpp_exe.obj>)
set_property(TARGET lua.cpp_exe.static PROPERTY OUTPUT_NAME "lua_cxx_static")
target_link_libraries(lua.cpp_exe.static PRIVATE lua.cpp.static ${LIB_MATH})
