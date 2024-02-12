set (CMAKE_C_STANDARD 99)
set (CMAKE_CXX_STANDARD 17)

add_library(
    rugpt2 ${WEBROGUE_MOD_LIB_TYPE}

    rugpt2.cpp
    common.cpp
    utf.cpp
)

target_include_directories(
    rugpt2 PRIVATE 
    external/ggml/include
    external/ggml/include/ggml
)

if(WEBROGUE_NATIVE_MODS)
    if(${WEBROGUE_MOD_LIB_TYPE} STREQUAL "STATIC")
        set(GGML_STATIC TRUE)
    elseif(${WEBROGUE_MOD_LIB_TYPE} STREQUAL "SHARED")
        set(BUILD_SHARED_LIBS TRUE)
    else()
        message(FATAL_ERROR)
    endif()
endif()

exec_program(git ${CMAKE_CURRENT_LIST_DIR}/external/ggml ARGS apply ../ggml.patch OUTPUT_VARIABLE v)
add_subdirectory(external/ggml/src ggml)

if(WEBROGUE_NATIVE_MODS)
    target_link_libraries(
        rugpt2 ggml
    )
else()
    add_custom_command(
        TARGET rugpt2
        POST_BUILD
        COMMAND ${CMAKE_AR} qLs $<TARGET_FILE:rugpt2> $<TARGET_FILE:ggml>
    )
    add_dependencies(rugpt2 ggml)
endif()

register_mod(
    NAME rugpt2
)
