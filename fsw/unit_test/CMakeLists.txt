include(CTest)
find_program( MEMORYCHECK_COMMAND valgrind )
find_package(Threads)
set( MEMORYCHECK_COMMAND_OPTIONS "--trace-children=yes --leak-check=full" )

# Set include directories
include_directories(
	../mission_inc
	../platform_inc
	$ENV{CFE_DIR}/fsw/mission_inc
	$ENV{CFE_DIR}/fsw/platform_inc/cpu1
	${CMAKE_CURRENT_SOURCE_DIR}
	${CFE_INC_DIRS}
	${OSAL_INC_DIRS}
	${UTF_INCS}
	${APP_INC_DIRS})

add_definitions(-DCFE_LINUX)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -O0 -Wformat=0 -Wno-pointer-to-int-cast -Wno-int-to-pointer-cast -fprofile-arcs -ftest-coverage")

SET(UTF_SRC 
	$ENV{UTF_SRC}/src/utf_cfe_es_api.c
	$ENV{UTF_SRC}/src/utf_cfe_es_cds.c
	$ENV{UTF_SRC}/src/utf_cfe_esmempool.c
	$ENV{UTF_SRC}/src/utf_cfe_evs.c
	$ENV{UTF_SRC}/src/utf_cfe_psp_eeprom.c
	$ENV{UTF_SRC}/src/utf_cfe_psp_memory.c
	$ENV{UTF_SRC}/src/utf_cfe_psp_memrange.c
	$ENV{UTF_SRC}/src/utf_cfe_psp_memutils.c
	$ENV{UTF_SRC}/src/utf_cfe_psp_port.c
	$ENV{UTF_SRC}/src/utf_cfe_psp_ram.c
	$ENV{UTF_SRC}/src/utf_cfe_psp_support.c
	$ENV{UTF_SRC}/src/utf_cfe_sb_api.c
	$ENV{UTF_SRC}/src/utf_cfe_sb.c
	$ENV{UTF_SRC}/src/utf_cfe_tbl_api.c
	$ENV{UTF_SRC}/src/utf_cfe_tbl_internal.c
	$ENV{UTF_SRC}/src/utf_cfe_time_api.c
	$ENV{UTF_SRC}/src/utf_osapi.c
	$ENV{UTF_SRC}/src/utf_osfileapi.c
	$ENV{UTF_SRC}/src/utf_osfilesys.c
	$ENV{UTF_SRC}/src/utf_osloader.c
	$ENV{UTF_SRC}/src/utf_ostimer.c
	$ENV{UTF_SRC}/src/utf_sim.c
	$ENV{UTF_SRC}/src/utf_cfe_time_utils.c
	$ENV{CFE_CORE_SRC}/es/cfe_es_cds_mempool.c
	$ENV{CFE_CORE_SRC}/fs/cfe_fs_api.c
	$ENV{CFE_CORE_SRC}/sb/ccsds.c
	$ENV{CFE_CORE_SRC}/sb/cfe_sb_util.c
	$ENV{CFE_PSP_SRC}/${PSP}/src/cfe_psp_watchdog.c)

# Add the executables
add_executable(HS-UT 
		${CMAKE_CURRENT_SOURCE_DIR}/utf_cfe_es_genctr.c
		${CMAKE_CURRENT_SOURCE_DIR}/utf_test_hs.c
		${CMAKE_CURRENT_SOURCE_DIR}/cfe_psp_timer.c
		${APP_SRC}
		${APP_CUSTOM_SRC}
		${UTF_SRC})
target_link_libraries(HS-UT -fprofile-arcs gcov ${CMAKE_THREAD_LIBS_INIT})

# We need to both run the test and compare the as-run with the expected output
# with a diff command. 
add_test(NAME hs-ut COMMAND $<TARGET_FILE:HS-UT>)

# Copy input files to build directory
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmd.in ${CMAKE_CURRENT_BINARY_DIR} COPYONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/event.in ${CMAKE_CURRENT_BINARY_DIR} COPYONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/wakeup.in ${CMAKE_CURRENT_BINARY_DIR} COPYONLY)
    
# Build tables for the tests
add_subdirectory(cf/apps cf/apps)

