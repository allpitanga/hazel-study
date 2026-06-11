cmake_minimum_required(VERSION 3.20)

file(GLOB_RECURSE SANDBOX_SOURCES
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
)

add_executable(Sandbox ${SANDBOX_SOURCES})

find_package(spdlog CONFIG REQUIRED)

target_include_directories(Sandbox
    PRIVATE
        ${CMAKE_SOURCE_DIR}/Hazel/src
)

target_link_libraries(Sandbox
    PRIVATE
        Hazel
        spdlog::spdlog
)

set_target_properties(Sandbox PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${OUTPUT_SUBDIR}/Sandbox"
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin-int/${OUTPUT_SUBDIR}/Sandbox"
    PDB_OUTPUT_DIRECTORY     "${CMAKE_SOURCE_DIR}/bin/${OUTPUT_SUBDIR}/Sandbox"
)

foreach(CFG Debug Release Dist)
    string(TOUPPER ${CFG} CFG_UPPER)
    set_target_properties(Sandbox PROPERTIES
        RUNTIME_OUTPUT_DIRECTORY_${CFG_UPPER} "${CMAKE_SOURCE_DIR}/bin/${CFG}-${CMAKE_SYSTEM_NAME}-x64/Sandbox"
        ARCHIVE_OUTPUT_DIRECTORY_${CFG_UPPER} "${CMAKE_SOURCE_DIR}/bin-int/${CFG}-${CMAKE_SYSTEM_NAME}-x64/Sandbox"
        PDB_OUTPUT_DIRECTORY_${CFG_UPPER}     "${CMAKE_SOURCE_DIR}/bin/${CFG}-${CMAKE_SYSTEM_NAME}-x64/Sandbox"
    )
endforeach()

if(WIN32)
    target_compile_definitions(Sandbox
        PRIVATE
            HZ_PLATFORM_WINDOWS
    )

    # # Static CRT (mirrors staticruntime "On")
    # set_property(TARGET Sandbox PROPERTY
    #     MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>"
    # )
endif()

target_compile_definitions(Sandbox
    PRIVATE
        $<$<CONFIG:Debug>:HZ_DEBUG>
        $<$<CONFIG:Release>:HZ_RELEASE>
        $<$<CONFIG:Dist>:HZ_DIST>
)

target_compile_options(Sandbox PRIVATE
    $<$<AND:$<CXX_COMPILER_ID:MSVC>,$<CONFIG:Debug>>:/Od /Zi>
    $<$<AND:$<CXX_COMPILER_ID:MSVC>,$<CONFIG:Release>>:/O2>
    $<$<AND:$<CXX_COMPILER_ID:MSVC>,$<CONFIG:Dist>>:/O2>
    $<$<AND:$<NOT:$<CXX_COMPILER_ID:MSVC>>,$<CONFIG:Debug>>:-O0 -g>
    $<$<AND:$<NOT:$<CXX_COMPILER_ID:MSVC>>,$<CONFIG:Release>>:-O2>
    $<$<AND:$<NOT:$<CXX_COMPILER_ID:MSVC>>,$<CONFIG:Dist>>:-O2>
)
