include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

if (NOT DEFINED SomeLib_INSTALL_CMAKEDIR)
   set(SomeLib_INSTALL_CMAKEDIR "${CMAKE_INSTALL_LIBDIR}/cmake/SomeLib"
       CACHE STRING "Path to SomeLib CMake files")
endif ()

install(TARGETS SomeLib EXPORT SomeLib_Targets
        RUNTIME COMPONENT SomeLib_Runtime
        LIBRARY COMPONENT SomeLib_Runtime
        NAMELINK_COMPONENT SomeLib_Development
        ARCHIVE COMPONENT SomeLib_Development
        INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")

install(DIRECTORY "${SomeLib_SOURCE_DIR}/include/" "${SomeLib_BINARY_DIR}/include/"
        TYPE INCLUDE
        COMPONENT SomeLib_Development)

if (BUILD_SHARED_LIBS)
    set(type shared)
else ()
    set(type static)
endif ()

install(EXPORT SomeLib_Targets
        DESTINATION "${SomeLib_INSTALL_CMAKEDIR}"
        NAMESPACE SomeLib::
        FILE SomeLib-${type}-targets.cmake
        COMPONENT SomeLib_Development)

write_basic_package_version_file(
    SomeLibConfigVersion.cmake
    COMPATIBILITY SameMajorVersion)

install(FILES
        "${CMAKE_CURRENT_SOURCE_DIR}/SomeLibConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/SomeLibConfigVersion.cmake"
        DESTINATION "${SomeLib_INSTALL_CMAKEDIR}"
        COMPONENT SomeLib_Development)

# TODO: add additional CPack variables here
include(CPack)
