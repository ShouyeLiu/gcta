# Remove FetchContent here! It's already declared in test/CMakeLists.txt

find_package(LAPACK REQUIRED)

# Collect test sources
file(GLOB Test_SRC *.cpp)
set(MAIN_TEST "main.cpp")
list(REMOVE_ITEM Test_SRC "${MAIN_TEST}")

# Set paths
set(TEST_BED "${CMAKE_CURRENT_SOURCE_DIR}/data/test")
set(TOY_BED "${CMAKE_CURRENT_SOURCE_DIR}/data/toy")

# Output and include config
file(MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/log_out")
configure_file(test_config.h.in ${CMAKE_BINARY_DIR}/generated/test_config.h)
include_directories(${CMAKE_BINARY_DIR}/generated/)

# Define test generation function
function(addTestItem test_name source lib_name run_param)
    add_executable(${test_name} ${MAIN_TEST} ${source})
    target_link_libraries(${test_name}
        ${lib_name}
        gtest
        gtest_main
        ${LAPACK_LIBRARIES} 
        -Wl,-Bstatic z -Wl,--start-group ${MKL_LIB} -Wl,--end-group gomp -Wl,-Bdynamic pthread m dl
    )
    add_test(NAME ${test_name} COMMAND ${test_name} "${run_param}")
endfunction()

# Add your test items
addTestItem(chisq_test test_chisq.cpp "statlib" "")
addTestItem(covar_test test_covar.cpp "covar" "")
