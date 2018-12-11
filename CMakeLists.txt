file(GLOB SQLITE_HEADER_FILES_INTERNAL
    LIST_DIRECTORIES false
    sqlite3.h
    sqlite3ext.h
    )
file(GLOB SQLITE_SOURCE_FILES
    LIST_DIRECTORIES false
    sqlite3.c
    )

set(SQLITE_HEADER_FILES
    ${SQLITE_HEADER_FILES_INTERNAL}
    CACHE INTERNAL ""
    )

option(USE_STATIC_SQLITE
    "Use static version of the SQLite library."
    OFF
    )

if(USE_STATIC_SQLITE)
    set(LIB_TYPE "STATIC")
else(USE_STATIC_SQLITE)
    set(LIB_TYPE "SHARED")
endif(USE_STATIC_SQLITE)

if(WIN32 AND (NOT USE_STATIC_SQLITE))
    if(CMAKE_CL_64)
        set(SQLITE_EXPORT_DEFS "sqlite3_x64.def")
    else(CMAKE_CL_64)
        set(SQLITE_EXPORT_DEFS "sqlite3_win32.def")
    endif(CMAKE_CL_64)
endif(WIN32 AND (NOT USE_STATIC_SQLITE))

if(UNIX OR MINGW)
    list(APPEND PLATFORM_LIBRARIES m pthread dl)
endif(UNIX OR MINGW)

add_library(sqlite3
    ${LIB_TYPE}
    ${SQLITE_HEADER_FILES}
    ${SQLITE_SOURCE_FILES}
    ${SQLITE_EXPORT_DEFS}
    )
add_executable(sqlite3-cli
    shell.c
    )

target_compile_definitions(sqlite3
    PUBLIC -DSQLITE_ENABLE_FTS3=1
    PUBLIC -DSQLITE_ENABLE_FTS5=1
    PUBLIC -DSQLITE_ENABLE_RTREE=1
    PUBLIC -DSQLITE_ENABLE_DBSTAT_VTAB=1
    PUBLIC -DSQLITE_ENABLE_JSON1=1
    PUBLIC -DSQLITE_ENABLE_RBU=1
    PUBLIC -DSQLITE_ENABLE_DESERIALIZE=1
    PUBLIC -DSQLITE_ENABLE_COLUMN_METADATA=1
    )
target_link_libraries(sqlite3-cli
    sqlite3
    ${PLATFORM_LIBRARIES}
    )
set_target_properties(sqlite3-cli
    PROPERTIES
    OUTPUT_NAME sqlite3
    )
