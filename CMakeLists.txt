##cmake_minimum_required(VERSION 3.15)
##include(cmake/version.cmake)
##
##option(MSDF_ATLAS_BUILD_STANDALONE "Build the msdf-atlas-gen standalone executable" ON)
##option(MSDF_ATLAS_USE_VCPKG "Use vcpkg package manager to link project dependencies" ON)
##option(MSDF_ATLAS_USE_SKIA "Build with the Skia library" ON)
##option(MSDF_ATLAS_NO_ARTERY_FONT "Disable Artery Font export and do not require its submodule" OFF)
##option(MSDF_ATLAS_MSDFGEN_EXTERNAL "Do not build the msdfgen submodule but find it as an external package" OFF)
##option(MSDF_ATLAS_INSTALL "Generate installation target" OFF)
##option(MSDF_ATLAS_DYNAMIC_RUNTIME "Link dynamic runtime library instead of static" OFF)
##option(BUILD_SHARED_LIBS "Generate dynamic library files instead of static" OFF)
##
##if(NOT MSDF_ATLAS_MSDFGEN_EXTERNAL)
##    set(MSDFGEN_CORE_ONLY OFF CACHE INTERNAL "Only build the core msdfgen library with no dependencies (disabled for msdf-atlas-gen)" FORCE)
##    set(MSDFGEN_BUILD_STANDALONE OFF CACHE BOOL "Build the msdfgen standalone executable")
##    set(MSDFGEN_USE_VCPKG ${MSDF_ATLAS_USE_VCPKG} CACHE INTERNAL "Use vcpkg package manager to link msdfgen project dependencies" FORCE)
##    set(MSDFGEN_USE_OPENMP OFF CACHE INTERNAL "Build with OpenMP support for multithreaded code (disabled for msdf-atlas-gen)" FORCE)
##    set(MSDFGEN_USE_CPP11 ON CACHE INTERNAL "Build with C++11 enabled (always enabled for msdf-atlas-gen)" FORCE)
##    set(MSDFGEN_USE_SKIA ${MSDF_ATLAS_USE_SKIA} CACHE INTERNAL "Build msdfgen with the Skia library" FORCE)
##    set(MSDFGEN_INSTALL ${MSDF_ATLAS_INSTALL} CACHE INTERNAL "Generate installation target for msdfgen" FORCE)
##    set(MSDFGEN_DYNAMIC_RUNTIME ${MSDF_ATLAS_DYNAMIC_RUNTIME} CACHE INTERNAL "Link dynamic runtime library instead of static for msdfgen" FORCE)
##endif()
##
##get_property(MULTI_CONFIG GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG)
##if(NOT MULTI_CONFIG AND NOT CMAKE_BUILD_TYPE)
##    message(STATUS "CMAKE_BUILD_TYPE not set, defaulting to Release")
##    set(CMAKE_BUILD_TYPE Release)
##endif()
##
##if(MSDF_ATLAS_DYNAMIC_RUNTIME)
##    set(MSDF_ATLAS_MSVC_RUNTIME "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL")
##else()
##    set(MSDF_ATLAS_MSVC_RUNTIME "MultiThreaded$<$<CONFIG:Debug>:Debug>")
##endif()
##
##if(BUILD_SHARED_LIBS)
##    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
##endif()
##
##if(MSDF_ATLAS_USE_VCPKG)
##    # Make sure that vcpkg toolchain file is set
##    if(NOT CMAKE_TOOLCHAIN_FILE)
##        if(DEFINED ENV{VCPKG_ROOT})
##            set(CMAKE_TOOLCHAIN_FILE "$ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake")
##        else()
##            message(SEND_ERROR "Vcpkg toolchain not configured. Either set VCPKG_ROOT environment variable or pass -DCMAKE_TOOLCHAIN_FILE=VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake to cmake")
##        endif()
##    endif()
##    # Default to statically linked vcpkg triplet on Windows
##    if(WIN32 AND NOT VCPKG_TARGET_TRIPLET AND NOT MSDF_ATLAS_DYNAMIC_RUNTIME)
##        if(CMAKE_GENERATOR_PLATFORM MATCHES "64$" AND NOT CMAKE_GENERATOR_PLATFORM STREQUAL "ARM64")
##            set(VCPKG_TARGET_TRIPLET "x64-windows-static")
##        elseif(CMAKE_GENERATOR_PLATFORM MATCHES "32$" OR CMAKE_GENERATOR_PLATFORM STREQUAL "x86")
##            set(VCPKG_TARGET_TRIPLET "x86-windows-static")
##        else()
##            if(CMAKE_GENERATOR_PLATFORM)
##                message(WARNING "Vcpkg triplet not explicitly specified and could not be deduced. Recommend using -DVCPKG_TARGET_TRIPLET=x64-windows-static or similar")
##            else()
##                message(WARNING "Vcpkg triplet not explicitly specified and could not be deduced. Recommend using -A to explicitly select platform (Win32 or x64)")
##            endif()
##        endif()
##    endif()
##    # Select project features
##    if(NOT MSDF_ATLAS_VCPKG_FEATURES_SET)
##        set(VCPKG_MANIFEST_NO_DEFAULT_FEATURES ON)
##        if(MSDF_ATLAS_USE_SKIA)
##            list(APPEND VCPKG_MANIFEST_FEATURES "geometry-preprocessing")
##        endif()
##    endif()
##    set(MSDFGEN_VCPKG_FEATURES_SET ON)
##endif()
##
### Version is specified in vcpkg.json
##project(msdf-atlas-gen VERSION ${MSDF_ATLAS_VERSION} LANGUAGES CXX)
##
##if(MSDF_ATLAS_MSDFGEN_EXTERNAL)
##    if(NOT TARGET msdfgen::msdfgen)
##        find_package(msdfgen REQUIRED)
##    endif()
##else()
##    add_subdirectory(msdfgen)
##endif()
##find_package(Threads REQUIRED)
##if(NOT TARGET PNG::PNG)
##    find_package(PNG REQUIRED)
##endif()
##
##file(GLOB_RECURSE MSDF_ATLAS_HEADERS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "msdf-atlas-gen/*.h" "msdf-atlas-gen/*.hpp")
##file(GLOB_RECURSE MSDF_ATLAS_SOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "msdf-atlas-gen/*.cpp")
##
### msdf-atlas-gen library
##add_library(msdf-atlas-gen ${MSDF_ATLAS_HEADERS} ${MSDF_ATLAS_SOURCES})
##add_library(msdf-atlas-gen::msdf-atlas-gen ALIAS msdf-atlas-gen)
##set_target_properties(msdf-atlas-gen PROPERTIES PUBLIC_HEADER "${MSDF_ATLAS_HEADERS}")
##set_property(TARGET msdf-atlas-gen PROPERTY MSVC_RUNTIME_LIBRARY "${MSDF_ATLAS_MSVC_RUNTIME}")
##target_compile_definitions(msdf-atlas-gen PUBLIC
##    MSDF_ATLAS_VERSION=${MSDF_ATLAS_VERSION}
##    MSDF_ATLAS_VERSION_MAJOR=${MSDF_ATLAS_VERSION_MAJOR}
##    MSDF_ATLAS_VERSION_MINOR=${MSDF_ATLAS_VERSION_MINOR}
##    MSDF_ATLAS_VERSION_REVISION=${MSDF_ATLAS_VERSION_REVISION}
##    MSDF_ATLAS_COPYRIGHT_YEAR=${MSDF_ATLAS_COPYRIGHT_YEAR}
##)
##target_include_directories(msdf-atlas-gen INTERFACE
##    $<INSTALL_INTERFACE:include>
##    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
##)
##if(MSDF_ATLAS_NO_ARTERY_FONT)
##    target_compile_definitions(msdf-atlas-gen PUBLIC MSDF_ATLAS_NO_ARTERY_FONT)
##else()
##    target_include_directories(msdf-atlas-gen PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/artery-font-format)
##endif()
##set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT msdf-atlas-gen)
##
##target_compile_features(msdf-atlas-gen PUBLIC cxx_std_11)
##target_link_libraries(msdf-atlas-gen PRIVATE Threads::Threads PNG::PNG)
##target_link_libraries(msdf-atlas-gen PUBLIC msdfgen::msdfgen)
##
##if(BUILD_SHARED_LIBS AND WIN32)
##    target_compile_definitions(msdf-atlas-gen PRIVATE "MSDF_ATLAS_PUBLIC=__declspec(dllexport)")
##    target_compile_definitions(msdf-atlas-gen INTERFACE "MSDF_ATLAS_PUBLIC=__declspec(dllimport)")
##else()
##    target_compile_definitions(msdf-atlas-gen PUBLIC MSDF_ATLAS_PUBLIC=)
##endif()
##
### msdf-atlas-gen standalone executable
##if(MSDF_ATLAS_BUILD_STANDALONE)
##    set(MSDF_ATLAS_STANDALONE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/msdf-atlas-gen/main.cpp")
##    if(MSVC)
##        set(MSDF_ATLAS_STANDALONE_SOURCES ${MSDF_ATLAS_STANDALONE_SOURCES} "${CMAKE_CURRENT_SOURCE_DIR}/msdf-atlas-gen.rc")
##    endif()
##    add_executable(msdf-atlas-gen-standalone ${MSDF_ATLAS_STANDALONE_SOURCES})
##    target_compile_definitions(msdf-atlas-gen-standalone PUBLIC MSDF_ATLAS_STANDALONE)
##    target_compile_definitions(msdf-atlas-gen-standalone PRIVATE MSDF_ATLAS_VERSION_UNDERLINE=${MSDF_ATLAS_VERSION_UNDERLINE})
##    set_property(TARGET msdf-atlas-gen-standalone PROPERTY MSVC_RUNTIME_LIBRARY "${MSDF_ATLAS_MSVC_RUNTIME}")
##    set_target_properties(msdf-atlas-gen-standalone PROPERTIES
##        OUTPUT_NAME msdf-atlas-gen
##        ARCHIVE_OUTPUT_NAME msdf-atlas-gen-standalone
##        # Avoid deleting msdf-atlas-gen.lib during clean
##        ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
##        RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
##    )
##    target_link_libraries(msdf-atlas-gen-standalone PRIVATE msdf-atlas-gen::msdf-atlas-gen)
##    set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT msdf-atlas-gen-standalone)
##endif()
##
### Installation
##if(MSDF_ATLAS_INSTALL)
##    include(GNUInstallDirs)
##    include(CMakePackageConfigHelpers)
##    set(MSDF_ATLAS_CONFIG_PATH "lib/cmake/msdf-atlas-gen")
##
##    # install tree package config
##    write_basic_package_version_file(
##        "${CMAKE_CURRENT_BINARY_DIR}/msdf-atlas-gen-config-version.cmake"
##        VERSION ${PROJECT_VERSION}
##        COMPATIBILITY SameMajorVersion
##    )
##
##    configure_package_config_file(
##        cmake/msdf-atlas-gen-config.cmake.in
##        ${MSDF_ATLAS_CONFIG_PATH}/msdf-atlas-gen-config.cmake
##        INSTALL_DESTINATION ${MSDF_ATLAS_CONFIG_PATH}
##        NO_CHECK_REQUIRED_COMPONENTS_MACRO
##    )
##
##    # build tree package config
##    configure_file(
##        cmake/msdf-atlas-gen-config.cmake.in
##        msdf-atlas-gen-config.cmake
##        @ONLY
##    )
##
##    install(TARGETS msdf-atlas-gen EXPORT msdf-atlas-gen-targets
##        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
##        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
##        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
##        FRAMEWORK DESTINATION ${CMAKE_INSTALL_LIBDIR}
##        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/msdf-atlas-gen
##    )
##    if(MSVC AND BUILD_SHARED_LIBS)
##        install(FILES $<TARGET_PDB_FILE:msdf-atlas-gen> DESTINATION ${CMAKE_INSTALL_BINDIR} OPTIONAL)
##    endif()
##    
##    export(EXPORT msdf-atlas-gen-targets NAMESPACE msdf-atlas-gen:: FILE "${CMAKE_CURRENT_BINARY_DIR}/msdf-atlas-gen-targets.cmake")
##    install(EXPORT msdf-atlas-gen-targets FILE msdf-atlas-gen-targets.cmake NAMESPACE msdf-atlas-gen:: DESTINATION ${MSDF_ATLAS_CONFIG_PATH})
##
##    if(MSDF_ATLAS_BUILD_STANDALONE)
##        install(TARGETS msdf-atlas-gen-standalone EXPORT msdf-atlas-gen-binary-targets DESTINATION ${CMAKE_INSTALL_BINDIR})
##        if(MSVC)
##            install(FILES $<TARGET_PDB_FILE:msdf-atlas-gen-standalone> DESTINATION ${CMAKE_INSTALL_BINDIR} OPTIONAL)
##        endif()
##        export(EXPORT msdf-atlas-gen-binary-targets NAMESPACE msdf-atlas-gen-standalone:: FILE "${CMAKE_CURRENT_BINARY_DIR}/msdf-atlas-gen-binary-targets.cmake")
##        install(EXPORT msdf-atlas-gen-binary-targets FILE msdf-atlas-gen-binary-targets.cmake NAMESPACE msdf-atlas-gen-standalone:: DESTINATION ${MSDF_ATLAS_CONFIG_PATH})
##    endif()
##
##    install(
##        FILES
##            "${CMAKE_CURRENT_BINARY_DIR}/${MSDF_ATLAS_CONFIG_PATH}/msdf-atlas-gen-config.cmake"
##            "${CMAKE_CURRENT_BINARY_DIR}/msdf-atlas-gen-config-version.cmake"
##        DESTINATION ${MSDF_ATLAS_CONFIG_PATH}
##    )
##endif()

# Minimum version to run the script
cmake_minimum_required(VERSION 3.16.0)

# Setting build type
set(DEFAULT_BUILD_TYPE "Debug")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${DEFAULT_BUILD_TYPE}' as none was specified.")
  set(CMAKE_BUILD_TYPE "${DEFAULT_BUILD_TYPE}" CACHE
      STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
    "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

# Setting up the project name and version
project(Msdf-atlas-gen
        VERSION 2.26.0
        DESCRIPTION ""
        HOMEPAGE_URL ""
        LANGUAGES CXX)

# Setting the install prefix
set(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR})

# First for the generic no-config case (e.g. with mingw)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
# Second, for multi-config builds (e.g. msvc)
foreach(OUTPUT_CONFIG ${CMAKE_CONFIGURATION_TYPES})
    string( TOUPPER ${OUTPUT_CONFIG} OUTPUT_CONFIG )
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_${OUTPUT_CONFIG} ${CMAKE_BINARY_DIR}/bin)
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_${OUTPUT_CONFIG} ${CMAKE_BINARY_DIR}/lib)
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_${OUTPUT_CONFIG} ${CMAKE_BINARY_DIR}/lib)
endforeach(OUTPUT_CONFIG CMAKE_CONFIGURATION_TYPES)

# Setting project options
option(MSDF_ATLAS_GEN_MSVC_DYNAMIC_LINKED_RUNTIME "Uses MSVC dynamic linked runtime" OFF)

# Include dirs
set(MSDF_ATLAS_GEN_INCLUDE_DIRS
    ${CMAKE_CURRENT_SOURCE_DIR}
)

# Header files
set(MSDF_ATLAS_GEN_HEADERS

)

# Source files
set(MSDF_ATLAS_GEN_SOURCES
    "msdf-atlas-gen/Charset.cpp"
    "msdf-atlas-gen/FontGeometry.cpp"
    "msdf-atlas-gen/GlyphGeometry.cpp"
    "msdf-atlas-gen/RectanglePacker.cpp"
    "msdf-atlas-gen/TightAtlasPacker.cpp"
    "msdf-atlas-gen/Workload.cpp"
    "msdf-atlas-gen/artery-font-export.cpp"
    "msdf-atlas-gen/bitmap-blit.cpp"
    "msdf-atlas-gen/charset-parser.cpp"
    "msdf-atlas-gen/csv-export.cpp"
    "msdf-atlas-gen/glyph-generators.cpp"
    "msdf-atlas-gen/image-encode.cpp"
    "msdf-atlas-gen/json-export.cpp"
    "msdf-atlas-gen/main.cpp"
    "msdf-atlas-gen/shadron-preview-generator.cpp"
    "msdf-atlas-gen/size-selectors.cpp"
    "msdf-atlas-gen/utf8.cpp"
)

# MSDF_ATLAS_GEN library
add_library(msdf_atlas_gen ${MSDF_ATLAS_GEN_SOURCES})

# Check for MSVC
if(MSVC)
    # Check for dynamic runtime
    if(MSDF_ATLAS_GEN_MSVC_DYNAMIC_LINKED_RUNTIME)
        set_property(TARGET msdf_atlas_gen PROPERTY
            MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL"
        )
    else()
        set_property(TARGET msdf_atlas_gen PROPERTY
            MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>"
        )
    endif()
endif()

# MSDF_ATLAS_GEN include dirs
target_include_directories(msdf_atlas_gen
PUBLIC
    ${MSDF_ATLAS_GEN_INCLUDE_DIRS}
)

target_link_libraries(msdf_atlas_gen 
PUBLIC
    msdfgen
)

# Setting the install command to the app
install(TARGETS msdf_atlas_gen RUNTIME DESTINATION bin)
install(TARGETS msdf_atlas_gen LIBRARY DESTINATION lib)
