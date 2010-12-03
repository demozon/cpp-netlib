
# Copyright 2010 Dean Michael Berris. 
# Distributed under the Boost Software License, Version 1.0.
# (See accompanying file LICENSE_1_0.txt or copy at
# http://www.boost.org/LICENSE_1_0.txt)

include_directories(${CPP-NETLIB_SOURCE_DIR})
find_package( Boost 1.43.0 COMPONENTS unit_test_framework system regex thread filesystem )
find_package( OpenSSL )

if (OPENSSL_FOUND)
    include_directories( ${OPENSSL_INCLUDE_DIR} )
    add_definitions(-DBOOST_NETWORK_ENABLE_HTTPS)
endif()

find_package( Threads )
set(Boost_USE_STATIC_LIBS ON)
set(Boost_USE_MULTITHREADED ON)

if (Boost_FOUND)
    set ( TESTS
        client_constructor_test
        client_get_test
        client_get_different_port_test
        client_get_timeout_test
        )
    foreach ( test ${TESTS} )
        if (${CMAKE_CXX_COMPILER_ID} MATCHES GNU)
            set_source_files_properties(${test}.cpp
                PROPERTIES COMPILE_FLAGS "-Wall")
        endif()
        add_executable(cpp-netlib-http-${test} ${test}.cpp)
        target_link_libraries(cpp-netlib-http-${test} ${Boost_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
        if (OPENSSL_FOUND)
            target_link_libraries(cpp-netlib-http-${test} ${OPENSSL_LIBRARIES})
        endif()
        set_target_properties(cpp-netlib-http-${test}
            PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CPP-NETLIB_BINARY_DIR}/tests)
        add_test(cpp-netlib-http-${test} 
            %{CPP-NETLIB_BINARY_DIR}/tests/cpp-netlib-http-${test})
    endforeach (test)

    set ( SERVER_TESTS
        server_hello_world
        server_async
        )
    foreach ( test ${SERVER_TESTS} )
        if (${CMAKE_CXX_COMPILER_ID} MATCHES GNU)
            set_source_files_properties(${test}.cpp
                PROPERTIES COMPILE_FLAGS "-Wall")
        endif()
        add_executable(cpp-netlib-http-${test} ${test}.cpp)
        target_link_libraries(cpp-netlib-http-${test} ${Boost_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
        set_target_properties(cpp-netlib-http-${test}
            PROPERTIES RUNTIME_OUTPUT_DIRECTORY 
            ${CPP-NETLIB_BINARY_DIR}/tests)
        add_test(cpp-netlib-${test} 
            python 
            ${CPP-NETLIB_SOURCE_DIR}/libs/network/httplib_acceptance.py 
            ${CPP-NETLIB_BINARY_DIR}/tests/cpp-netlib-http-${test}
            ${CPP-NETLIB_BINARY_DIR}/tests/cpp-netlib-http-${test}.passed)
    endforeach (test)

endif()