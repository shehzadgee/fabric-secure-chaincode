set(SOURCE_FILES
    trusted_ledger.c
    enclave_u.c
)

set(cleanup_files
    ${CMAKE_CURRENT_SOURCE_DIR}/enclave_u.c
    ${CMAKE_CURRENT_SOURCE_DIR}/enclave_u.h
    ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/include
    ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/lib)
set_directory_properties(PROPERTIES ADDITIONAL_MAKE_CLEAN_FILES "${cleanup_files}")

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/enclave_u.c
    COMMAND ${SGX_EDGER8R} --untrusted enclave.edl
    --search-path ../enclave/ # enclave.edl
    --search-path ${COMMON_SOURCE_DIR}/enclave
    --search-path ${SGX_SDK}/include
    --search-path ${SGX_SSL}/include
    COMMENT "Generating enclave_u.{c,h}"
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)

add_library(libtl SHARED ${SOURCE_FILES})

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${COMMON_SOURCE_DIR}
    ${COMMON_SOURCE_DIR}/base64
    ${SGX_SDK}/include
)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${SGX_COMMON_CFLAGS} -fPIC -Wall -Wno-attributes -std=c11")

set_target_properties(libtl PROPERTIES PREFIX "")

target_link_libraries(libtl
    -L${SGX_LIBRARY_PATH} -l${SGX_URTS_LIB} -l${SGX_USVC_LIB} -L${SGX_SSL_LIBRARY_PATH} -lsgx_usgxssl)

add_dependencies(libtl enclave)

add_custom_command(TARGET libtl
    POST_BUILD
    COMMAND mkdir -p ${CMAKE_BINARY_DIR}/include
    COMMAND cp ${CMAKE_CURRENT_SOURCE_DIR}/trusted_ledger.h ${CMAKE_BINARY_DIR}/include
    COMMAND cp ${COMMON_SOURCE_DIR}/types.h ${CMAKE_BINARY_DIR}/include
    COMMAND cp ${CMAKE_BINARY_DIR}/libtl.so ${CMAKE_BINARY_DIR}/lib
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    COMMENT "Add include to build")

