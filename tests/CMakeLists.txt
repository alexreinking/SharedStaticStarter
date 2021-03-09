cmake_minimum_required(VERSION 3.19)
project(tests LANGUAGES NONE)

enable_testing()

if (CMAKE_HOST_WIN32)
    set(static_opts -DCMAKE_RELEASE_POSTFIX=_static)
    set(LDD dumpbin /dependents)
    set(EXT .exe)
elseif (CMAKE_HOST_APPLE)
    set(LDD otool -L)
else ()
    set(LDD ldd)
endif ()

# Set up install trees for subsequent tests

add_test(NAME build_shared
         COMMAND ${CMAKE_CTEST_COMMAND}
         --build-and-test "${CMAKE_CURRENT_LIST_DIR}/.." "${CMAKE_CURRENT_BINARY_DIR}/build-shared"
         --build-generator Ninja
         --build-options -DBUILD_SHARED_LIBS=YES -DCMAKE_BUILD_TYPE=Release)

add_test(NAME build_static
         COMMAND ${CMAKE_CTEST_COMMAND}
         --build-and-test "${CMAKE_CURRENT_LIST_DIR}/.." "${CMAKE_CURRENT_BINARY_DIR}/build-static"
         --build-generator Ninja
         --build-options -DBUILD_SHARED_LIBS=NO ${static_opts} -DCMAKE_BUILD_TYPE=Release)

add_test(NAME install_shared
         COMMAND ${CMAKE_COMMAND} --install "${CMAKE_CURRENT_BINARY_DIR}/build-shared" --prefix "${CMAKE_CURRENT_BINARY_DIR}/install-shared")

add_test(NAME install_shared_both
         COMMAND ${CMAKE_COMMAND} --install "${CMAKE_CURRENT_BINARY_DIR}/build-shared" --prefix "${CMAKE_CURRENT_BINARY_DIR}/install-both")

add_test(NAME install_static
         COMMAND ${CMAKE_COMMAND} --install "${CMAKE_CURRENT_BINARY_DIR}/build-static" --prefix "${CMAKE_CURRENT_BINARY_DIR}/install-static")

add_test(NAME install_static_both
         COMMAND ${CMAKE_COMMAND} --install "${CMAKE_CURRENT_BINARY_DIR}/build-static" --prefix "${CMAKE_CURRENT_BINARY_DIR}/install-both")

set_tests_properties(build_shared build_static PROPERTIES FIXTURES_SETUP "builds")

set_tests_properties(install_shared install_static install_shared_both install_static_both
                     PROPERTIES FIXTURES_SETUP "installs" FIXTURES_REQUIRED "builds")

# Function to encapsulate test creation

function(add_integration_test)
    cmake_parse_arguments(arg "" "NAME;ROOT;EXPECTED" "OPTIONS" ${ARGN})

    add_test(NAME ${arg_NAME}
             COMMAND ${CMAKE_CTEST_COMMAND}
             --build-and-test "${CMAKE_CURRENT_LIST_DIR}/integration" "${CMAKE_CURRENT_BINARY_DIR}/${arg_NAME}"
             --build-generator ${CMAKE_GENERATOR}
             --build-options ${arg_OPTIONS} "-DSomeLib_ROOT=${CMAKE_CURRENT_BINARY_DIR}/${arg_ROOT}"
             --test-command ${CMAKE_CTEST_COMMAND})

    add_test(NAME ${arg_NAME}_linkage_check
             COMMAND ${LDD} "${CMAKE_CURRENT_BINARY_DIR}/${arg_NAME}/main${EXT}")

    set_tests_properties(${arg_NAME} PROPERTIES
                         FIXTURES_SETUP "${arg_NAME}"
                         FIXTURES_REQUIRED "installs")

    set_tests_properties(${arg_NAME}_linkage_check PROPERTIES
                         FIXTURES_REQUIRED "${arg_NAME};installs")

    if (arg_EXPECTED STREQUAL "shared")
        set_tests_properties(${arg_NAME}_linkage_check PROPERTIES PASS_REGULAR_EXPRESSION "SomeLib")
    elseif (arg_EXPECTED STREQUAL "static")
        set_tests_properties(${arg_NAME}_linkage_check PROPERTIES FAIL_REGULAR_EXPRESSION "SomeLib")
    else ()
        set_tests_properties(${arg_NAME} PROPERTIES
                             WILL_FAIL TRUE
                             FAIL_REGULAR_EXPRESSION "${arg_EXPECTED}")
        set_tests_properties(${arg_NAME}_linkage_check PROPERTIES WILL_FAIL TRUE)
    endif ()
endfunction()

## Joint install directory tests

add_integration_test(
    NAME default
    ROOT install-both
    EXPECTED static
)

add_integration_test(
    NAME default-static
    ROOT install-both
    OPTIONS -DBUILD_SHARED_LIBS=NO
    EXPECTED static
)

add_integration_test(
    NAME default-shared
    ROOT install-both
    OPTIONS -DBUILD_SHARED_LIBS=YES
    EXPECTED shared
)

add_integration_test(
    NAME cache-shared
    ROOT install-both
    OPTIONS -DSomeLib_SHARED_LIBS=YES
    EXPECTED shared
)

add_integration_test(
    NAME cache-static-override
    ROOT install-both
    OPTIONS -DSomeLib_SHARED_LIBS=NO -DBUILD_SHARED_LIBS=YES
    EXPECTED static
)

add_integration_test(
    NAME cache-shared-override
    ROOT install-both
    OPTIONS -DSomeLib_SHARED_LIBS=YES -DBUILD_SHARED_LIBS=NO
    EXPECTED shared
)

add_integration_test(
    NAME component-shared
    ROOT install-both
    OPTIONS -Dexample_SomeLib_components=shared
    EXPECTED shared
)

add_integration_test(
    NAME component-static-override
    ROOT install-both
    OPTIONS -Dexample_SomeLib_components=static -DSomeLib_SHARED_LIBS=YES -DBUILD_SHARED_LIBS=YES
    EXPECTED static
)

add_integration_test(
    NAME component-shared-override
    ROOT install-both
    OPTIONS -Dexample_SomeLib_components=shared -DSomeLib_SHARED_LIBS=NO -DBUILD_SHARED_LIBS=NO
    EXPECTED shared
)

add_integration_test(
    NAME fail-component-both
    ROOT install-both
    OPTIONS "-Dexample_SomeLib_components=shared$<SEMICOLON>static"
    EXPECTED "SomeLib `static` and `shared` components are mutually exclusive."
)

add_integration_test(
    NAME fail-invalid-component
    ROOT install-both
    OPTIONS "-Dexample_SomeLib_components=foo"
    EXPECTED "SomeLib does not recognize component `foo`."
)

## Shared only

add_integration_test(
    NAME default-shared-only
    ROOT install-shared
    EXPECTED shared
)

add_integration_test(
    NAME cache-shared-only
    ROOT install-shared
    OPTIONS -DSomeLib_SHARED_LIBS=YES -DBUILD_SHARED_LIBS=NO
    EXPECTED shared
)

add_integration_test(
    NAME fail-cache-static-shared-only
    ROOT install-shared
    OPTIONS -DSomeLib_SHARED_LIBS=NO
    EXPECTED "SomeLib `static` libraries were requested but not found"
)

add_integration_test(
    NAME fail-component-static-shared-only
    ROOT install-shared
    OPTIONS -Dexample_SomeLib_components=static
    EXPECTED "SomeLib `static` libraries were requested but not found"
)

## Static only

add_integration_test(
    NAME default-static-only
    ROOT install-static
    EXPECTED static
)

add_integration_test(
    NAME cache-static-only
    ROOT install-static
    OPTIONS -DSomeLib_SHARED_LIBS=NO -DBUILD_SHARED_LIBS=YES
    EXPECTED static
)

add_integration_test(
    NAME fail-cache-shared-static-only
    ROOT install-static
    OPTIONS -DSomeLib_SHARED_LIBS=YES
    EXPECTED "SomeLib `shared` libraries were requested but not found"
)

add_integration_test(
    NAME fail-component-shared-static-only
    ROOT install-static
    OPTIONS -Dexample_SomeLib_components=shared
    EXPECTED "SomeLib `shared` libraries were requested but not found"
)
