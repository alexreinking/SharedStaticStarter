cmake_minimum_required(VERSION 3.19)
project(example)

enable_testing()

find_package(SomeLib 1 REQUIRED ${example_SomeLib_components})

add_executable(main main.cpp)
target_link_libraries(main PRIVATE SomeLib::SomeLib)

add_test(NAME random_is_42 COMMAND main)
set_tests_properties(random_is_42 PROPERTIES
                     PASS_REGULAR_EXPRESSION "is: 42"
                     ENVIRONMENT "PATH=$<TARGET_FILE_DIR:SomeLib::SomeLib>")

