cmake_minimum_required(VERSION 3.15)

project(hello C)

add_executable(${PROJECT_NAME}
        main.c
)

target_link_libraries(${PROJECT_NAME} PRIVATE
        pspdebug
        pspdisplay
        pspge
)

create_pbp_file(
        TARGET ${PROJECT_NAME}
        ICON_PATH NULL
        BACKGROUND_PATH NULL
        PREVIEW_PATH NULL
        TITLE ${PROJECT_NAME}
        VERSION 01.00
)

option(DEPLOY_TO_PPSSPP "Copy EBOOT.PBP to PPSSPP after build" ON)

if(DEPLOY_TO_PPSSPP AND DEFINED ENV{WSL_DISTRO_NAME})
    execute_process(
            COMMAND cmd.exe /c echo %USERPROFILE%
            OUTPUT_VARIABLE WINDOWS_USERPROFILE
            OUTPUT_STRIP_TRAILING_WHITESPACE
    )

    execute_process(
            COMMAND wslpath "${WINDOWS_USERPROFILE}"
            OUTPUT_VARIABLE WINDOWS_USERPROFILE_WSL
            OUTPUT_STRIP_TRAILING_WHITESPACE
    )

    set(PPSSPP_GAME_DIR
            "${WINDOWS_USERPROFILE_WSL}/Documents/PPSSPP/PSP/GAME/FalseFate"
    )

    add_custom_command(
            TARGET ${PROJECT_NAME}
            POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E make_directory "${PPSSPP_GAME_DIR}"
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "${CMAKE_CURRENT_BINARY_DIR}/EBOOT.PBP"
            "${PPSSPP_GAME_DIR}/EBOOT.PBP"
            COMMENT "Deploying EBOOT.PBP to PPSSPP"
    )
endif()