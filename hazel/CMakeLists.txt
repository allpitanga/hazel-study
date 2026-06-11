cmake_minimum_required(VERSION 3.20)

file(GLOB_RECURSE HAZEL_SOURCES
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
)

add_library(Hazel SHARED ${HAZEL_SOURCES})

find_package(spdlog CONFIG REQUIRED)

target_link_libraries(Hazel PRIVATE
    spdlog::spdlog
)

set_target_properties(Hazel PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${OUTPUT_SUBDIR}/Hazel"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${OUTPUT_SUBDIR}/Hazel"
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin-int/${OUTPUT_SUBDIR}/Hazel"
    PDB_OUTPUT_DIRECTORY     "${CMAKE_SOURCE_DIR}/bin/${OUTPUT_SUBDIR}/Hazel"
)

foreach(CFG Debug Release Dist)
    string(TOUPPER ${CFG} CFG_UPPER)
    set_target_properties(Hazel PROPERTIES
        RUNTIME_OUTPUT_DIRECTORY_${CFG_UPPER} "${CMAKE_SOURCE_DIR}/bin/${CFG}-${CMAKE_SYSTEM_NAME}-x64/Hazel"
        LIBRARY_OUTPUT_DIRECTORY_${CFG_UPPER} "${CMAKE_SOURCE_DIR}/bin/${CFG}-${CMAKE_SYSTEM_NAME}-x64/Hazel"
        ARCHIVE_OUTPUT_DIRECTORY_${CFG_UPPER} "${CMAKE_SOURCE_DIR}/bin-int/${CFG}-${CMAKE_SYSTEM_NAME}-x64/Hazel"
        PDB_OUTPUT_DIRECTORY_${CFG_UPPER}     "${CMAKE_SOURCE_DIR}/bin/${CFG}-${CMAKE_SYSTEM_NAME}-x64/Hazel"
    )
endforeach()

if(WIN32)
    target_compile_definitions(Hazel
        PRIVATE
            HZ_BUILD_DLL
            HZ_PLATFORM_WINDOWS
    )

    # set_property(TARGET Hazel PROPERTY
    #     MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>"
    # )

    add_custom_command(TARGET Hazel POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "$<TARGET_FILE:Hazel>"
            "${CMAKE_SOURCE_DIR}/bin/$<CONFIG>-${CMAKE_SYSTEM_NAME}-x64/Sandbox/$<TARGET_FILE_NAME:Hazel>"
    )
endif()

target_compile_definitions(Hazel
    PRIVATE
        $<$<CONFIG:Debug>:HZ_DEBUG>
        $<$<CONFIG:Release>:HZ_RELEASE>
        $<$<CONFIG:Dist>:HZ_DIST>
)

target_compile_options(Hazel PRIVATE
    $<$<AND:$<CXX_COMPILER_ID:MSVC>,$<CONFIG:Debug>>:/Od /Zi>
    $<$<AND:$<CXX_COMPILER_ID:MSVC>,$<OR:$<CONFIG:Release>,$<CONFIG:Dist>>>:/O2>
    $<$<AND:$<NOT:$<CXX_COMPILER_ID:MSVC>>,$<CONFIG:Debug>>:-O0 -g>
    $<$<AND:$<NOT:$<CXX_COMPILER_ID:MSVC>>,$<OR:$<CONFIG:Release>,$<CONFIG:Dist>>>:-O2>
)