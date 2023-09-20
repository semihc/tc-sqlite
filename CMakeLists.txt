# Top level CMake project
PROJECT(sqlite3)
cmake_minimum_required(VERSION 3.15)

# Enforce out-of-source build, fail otherwise
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)
set(CMAKE_DISABLE_SOURCE_CHANGES  ON)

# Project source directory
set(PrjSrc ${CMAKE_SOURCE_DIR}/src)

include_directories(${PrjSrc})

# Library sources
set(LibSrc src/sqlite3.c src/sqlite3.h src/sqlite3ext.h)

# Executable (shell) sources
set(ExeSrc src/shell.c ${LibSrc})

add_library(lib_static STATIC ${LibSrc})
set_target_properties(lib_static PROPERTIES OUTPUT_NAME sqlite3)

add_library(lib_shared SHARED ${LibSrc})
set_target_properties(lib_shared PROPERTIES OUTPUT_NAME sqlite3dl)
set_property(TARGET lib_shared PROPERTY POSITION_INDEPENDENT_CODE 1)


add_executable(exe ${ExeSrc})
set_target_properties(exe PROPERTIES OUTPUT_NAME sqlite3)

add_definitions(-DSQLITE_ENABLE_RTREE)
add_definitions(-DSQLITE_ENABLE_FTS4)
add_definitions(-DSQLITE_ENABLE_FTS5)
add_definitions(-DSQLITE_ENABLE_JSON1)
add_definitions(-DSQLITE_ENABLE_RBU)
add_definitions(-DSQLITE_ENABLE_STAT4)
# Uncomment this for single-threaded variant (faster)
#add_definitions(-DSQLITE_THREADSAFE=0)

if(WIN32)
  #Todo: Exe installation doesn't work 
  add_custom_command(TARGET exe POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:exe> ${CMAKE_BINARY_DIR}/sqlite3.exe
    DEPENDS exe
  )
  install(FILES sqlite3.exe DESTINATION bin) #$<TARGET_FILE:exe>
  #?install(TARGETS exe RUNTIME DESTINATION bin)
else()
  include(FindThreads)
  target_link_libraries(exe m ${CMAKE_THREAD_LIBS_INIT} ${CMAKE_DL_LIBS})
  install(TARGETS exe RUNTIME DESTINATION bin)
endif()

install(FILES src/sqlite3.h src/sqlite3ext.h DESTINATION include)
install(TARGETS lib_static LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
install(TARGETS lib_shared LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)