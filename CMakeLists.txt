cmake_minimum_required(VERSION 3.1.0 FATAL_ERROR)
PROJECT(sioclient)

function(get_lib_name out_lib_name component)
if(MSVC)
set(${out_lib_name} "${component}.lib" PARENT_SCOPE)
else()
set(${out_lib_name} "lib${component}.a" PARENT_SCOPE)
endif()
endfunction()

function(select_lib output_lib lib_list configuration)
list(LENGTH lib_list LIB_LEN )
if(LIB_LEN GREATER 1)
set(LIB_MET OFF)
foreach(lib_i ${lib_list})
	if(LIB_MET)
		set(${output_lib} ${lib_i} PARENT_SCOPE)
		return()
	endif()
	if(NOT lib_i MATCHES ".*[Bb]oost.*" )
		if(lib_i STREQUAL "debug" )
			if(configuration STREQUAL "Debug")
				set(${LIB_MET} ON)
			endif()
		else()
			if(configuration STREQUAL "Release")
				set(${LIB_MET} ON)
			endif()
		endif()
	endif()
endforeach(lib_i)
else()
set(${output_lib} ${lib_list} PARENT_SCOPE)
endif()
endfunction()

function(install_libs configuration lib_dir)


get_lib_name(renamed_system "boost_system")
select_lib(src_lib_x ${Boost_SYSTEM_LIBRARY} ${configuration} )
install(FILES ${src_lib_x}
    CONFIGURATIONS ${configuration} DESTINATION ${lib_dir} RENAME ${renamed_system}
)
get_lib_name(renamed_date_time "boost_date_time")
select_lib(src_lib_y ${Boost_SYSTEM_LIBRARY} ${configuration} )
install(FILES ${src_lib_y}
    CONFIGURATIONS ${configuration} DESTINATION ${lib_dir} RENAME ${renamed_date_time}
)
get_lib_name(renamed_random "boost_random")
select_lib(src_lib_z ${Boost_SYSTEM_LIBRARY} ${configuration} )
install(FILES ${src_lib_z}
    CONFIGURATIONS ${configuration} DESTINATION ${lib_dir} RENAME ${renamed_random}
)

install(TARGETS sioclient
    CONFIGURATIONS ${configuration} DESTINATION ${lib_dir}
)
endfunction()

MESSAGE(STATUS ${CMAKE_CURRENT_LIST_DIR})
if(NOT CMAKE_BUILD_TYPE )
MESSAGE(STATUS "not define build type, set to release" )
set(CMAKE_BUILD_TYPE Release )
endif()

set(BOOST_VER "1.55.0" CACHE STRING "boost version" )
set(BOOST_PATH "__boost__" CACHE STRING "boost root path" )

if(${BOOST_PATH} STREQUAL "__boost__" )
    set(BOOST_INCLUDE "__boost__" CACHE STRING "boost include path" )
    set(BOOST_LIB "__boost__" CACHE STRING "boost lib path" )
    MESSAGE(STATUS "BOOST_PATH not set, find BOOST_INCLUDE and BOOST_LIB" )

    if(${BOOST_INCLUDE} STREQUAL "__boost__" OR ${BOOST_LIB} STREQUAL "__boost__" )
        MESSAGE(SEND_ERROR "You need to specify either BOOST_PATH or BOOST_INCLUDE & BOOST_LIB" )
        return()
    else()
        MESSAGE(STATUS "got include " ${BOOST_INCLUDE} )
        MESSAGE(STATUS "got lib " ${BOOST_LIB} )
        set(BOOST_INCLUDEDIR ${BOOST_INCLUDE} )
        set(BOOST_LIBRARYDIR ${BOOST_LIB} )
    endif()
else()
    MESSAGE(STATUS "got path " ${BOOST_PATH} )
    set(BOOST_ROOT ${BOOST_PATH} )
endif()
set(Boost_DEBUG 1)
set(Boost_USE_STATIC_LIBS ON) 
set(Boost_USE_MULTITHREADED ON) 
set(Boost_USE_STATIC_RUNTIME OFF) 
find_package(Boost ${BOOST_VER} REQUIRED COMPONENTS system date_time random) 

aux_source_directory(${CMAKE_CURRENT_LIST_DIR}/src ALL_SRC)
aux_source_directory(${CMAKE_CURRENT_LIST_DIR}/src/internal ALL_SRC)
file(GLOB ALL_HEADERS ${CMAKE_CURRENT_LIST_DIR}/src/*.h )
MESSAGE(STATUS ${ALL_HEADERS} )

add_library(sioclient STATIC ${ALL_SRC}) 
target_include_directories(sioclient PRIVATE ${Boost_INCLUDE_DIRS} 
    ${CMAKE_CURRENT_LIST_DIR}/src 
    ${CMAKE_CURRENT_LIST_DIR}/lib/websocketpp 
    ${CMAKE_CURRENT_LIST_DIR}/lib/rapidjson/include
)

set_property(TARGET sioclient PROPERTY CXX_STANDARD 11)
set_property(TARGET sioclient PROPERTY CXX_STANDARD_REQUIRED ON)
target_link_libraries(sioclient PRIVATE ${Boost_LIBRARIES})
install(FILES ${ALL_HEADERS} 
    DESTINATION "${CMAKE_CURRENT_LIST_DIR}/build/include"
    )
set(LIB_DIR_DEBUG "${CMAKE_CURRENT_LIST_DIR}/build/lib/debug")
set(LIB_DIR_RELEASE "${CMAKE_CURRENT_LIST_DIR}/build/lib/release")
install_libs(Debug ${LIB_DIR_DEBUG})
install_libs(Release ${LIB_DIR_RELEASE})
