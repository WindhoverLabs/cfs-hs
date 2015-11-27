include(CTest)
find_program( MEMORYCHECK_COMMAND valgrind )
find_package(Threads)
set( MEMORYCHECK_COMMAND_OPTIONS "--trace-children=yes --leak-check=full" )

# Set include directories
include_directories(
	${CMAKE_CURRENT_SOURCE_DIR}
	${CFE_INC_DIRS}
	${OSAL_INC_DIRS}
	${UTF_INCS}
	${APP_INC_DIRS})

add_definitions(-DCFE_LINUX)

set(CMAKE_C_FLAGS "-g -O0 -Wformat=0 -Wno-pointer-to-int-cast -Wno-int-to-pointer-cast -fprofile-arcs -ftest-coverage -m32")

# Add the executables
add_executable(HS-UT 
		${CMAKE_CURRENT_SOURCE_DIR}/utf_cfe_es_genctr.c
		${CMAKE_CURRENT_SOURCE_DIR}/utf_test_hs.c
		${APP_SRC}
		${APP_CUSTOM_SRC}
		${UTF_SRC})
target_link_libraries(HS-UT -fprofile-arcs gcov ${CMAKE_THREAD_LIBS_INIT})

# We need to both run the test and compare the as-run with the expected output
# with a diff command. 
add_test(NAME hs-ut
    COMMAND ${CMAKE_COMMAND}
    	-DTEST_PROG=$<TARGET_FILE:HS-UT>
    	-DEXPECTED_OUTPUT=${UNIT_TEST_INPUT_DIR}/hs_utf_test.out
    	-DACTUAL_OUTPUT=${CMAKE_CURRENT_BINARY_DIR}/hs_utf_test.out
    -P ${CMAKE_CURRENT_SOURCE_DIR}/runtest.cmake)

# Copy input files to build directory
configure_file(${UNIT_TEST_INPUT_DIR}/cmd.in ${CMAKE_CURRENT_BINARY_DIR} COPYONLY)
configure_file(${UNIT_TEST_INPUT_DIR}/event.in ${CMAKE_CURRENT_BINARY_DIR} COPYONLY)
configure_file(${UNIT_TEST_INPUT_DIR}/wakeup.in ${CMAKE_CURRENT_BINARY_DIR} COPYONLY)
    
# Build tables for the tests
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/cf/apps)
SET(CMAKE_CXX_ARCHIVE_CREATE "<CMAKE_AR> <OBJECTS> ${CMAKE_CURRENT_BINARY_DIR}/cf/apps")
SET(CMAKE_C_ARCHIVE_CREATE "<CMAKE_AR> <OBJECTS> ${CMAKE_CURRENT_BINARY_DIR}/cf/apps")
add_library(ut_hs_amt_bad STATIC ${UNIT_TEST_INPUT_DIR}/cf/apps/hs_amt_bad.c)
add_library(ut_hs_amt_tbl STATIC ${UNIT_TEST_INPUT_DIR}/cf/apps/hs_amt_tbl.c)
add_library(ut_hs_emt_bad STATIC ${UNIT_TEST_INPUT_DIR}/cf/apps/hs_emt_bad.c)
add_library(ut_hs_emt_tbl STATIC ${UNIT_TEST_INPUT_DIR}/cf/apps/hs_emt_tbl.c)
add_library(ut_hs_mat_bad STATIC ${UNIT_TEST_INPUT_DIR}/cf/apps/hs_mat_bad.c)
add_library(ut_hs_mat_tbl STATIC ${UNIT_TEST_INPUT_DIR}/cf/apps/hs_mat_tbl.c)
add_library(ut_hs_xct_bad STATIC ${UNIT_TEST_INPUT_DIR}/cf/apps/hs_xct_bad.c)
add_library(ut_hs_xct_tbl STATIC ${UNIT_TEST_INPUT_DIR}/cf/apps/hs_xct_tbl.c)
set_target_properties(ut_hs_amt_bad PROPERTIES PREFIX "" SUFFIX ".tbl")
set_target_properties(ut_hs_amt_tbl PROPERTIES PREFIX "" SUFFIX ".tbl")
set_target_properties(ut_hs_emt_bad PROPERTIES PREFIX "" SUFFIX ".tbl")
set_target_properties(ut_hs_emt_tbl PROPERTIES PREFIX "" SUFFIX ".tbl")
set_target_properties(ut_hs_mat_bad PROPERTIES PREFIX "" SUFFIX ".tbl")
set_target_properties(ut_hs_mat_tbl PROPERTIES PREFIX "" SUFFIX ".tbl")
set_target_properties(ut_hs_xct_bad PROPERTIES PREFIX "" SUFFIX ".tbl")
set_target_properties(ut_hs_xct_tbl PROPERTIES PREFIX "" SUFFIX ".tbl")

