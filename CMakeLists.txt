cmake_minimum_required(VERSION 2.8)
project(tfs)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

include(cotire)

if(MSVC)
    add_definitions(/W3)
else()
    add_compile_options(-Wall -pipe -fvisibility=hidden)
endif()

include(FindCXX11)
if(MSVC)
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)
else()
	include(FindLTO)
endif()

find_package(Crypto++ REQUIRED)
find_package(PugiXML REQUIRED)
find_package(GMP REQUIRED)
find_package(MySQL)
find_package(Threads)

if(DEFINED USE_LUAJIT AND NOT USE_LUAJIT)
    set(FORCE_LUAJIT ${USE_LUAJIT})
else()
    find_package(LuaJIT)
    set(FORCE_LUAJIT ${LuaJIT_FOUND})
endif()
option(USE_LUAJIT "Use LuaJIT" ${FORCE_LUAJIT})

if(FORCE_LUAJIT)
    # No changes for Windows needed
else()
    find_package(Lua REQUIRED)
endif()

find_package(Boost 1.53.0 COMPONENTS system filesystem iostreams REQUIRED)

add_subdirectory(src)
add_executable(tfs ${tfs_SRC})

include_directories(${MYSQL_INCLUDE_DIR} ${LUA_INCLUDE_DIR} ${Boost_INCLUDE_DIRS} ${PUGIXML_INCLUDE_DIR} ${Crypto++_INCLUDE_DIR} ${GMP_INCLUDE_DIRS})
target_link_libraries(tfs ${MYSQL_CLIENT_LIBS} ${LUA_LIBRARIES} ${Boost_LIBRARIES} ${Boost_FILESYSTEM_LIBRARY} ${PUGIXML_LIBRARIES} ${Crypto++_LIBRARIES} ${GMP_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})

set_target_properties(tfs PROPERTIES COTIRE_CXX_PREFIX_HEADER_INIT "src/otpch.h")
set_target_properties(tfs PROPERTIES COTIRE_ADD_UNITY_BUILD FALSE)
cotire(tfs)