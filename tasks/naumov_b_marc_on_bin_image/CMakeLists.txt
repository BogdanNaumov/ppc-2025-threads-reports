get_filename_component(ProjectId ${CMAKE_CURRENT_SOURCE_DIR} NAME)

set(LATEX_OUTPUT_PATH "${CMAKE_BINARY_DIR}/bin")
if (NOT EXISTS ${LATEX_OUTPUT_PATH})
    file(MAKE_DIRECTORY ${LATEX_OUTPUT_PATH})
endif()

if (USE_LATEX)
    message(STATUS "-- Building report: ${ProjectId}")
    
    # Главный TeX-файл должен называться как папка
    set(MAIN_TEX ${ProjectId}.tex)
    
    # Цель для предварительной компиляции
    add_custom_command(
        OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${ProjectId}.aux
        COMMAND ${PDFLATEX_COMPILER} -draftmode -interaction=nonstopmode ${MAIN_TEX}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        DEPENDS ${MAIN_TEX}
        COMMENT "First LaTeX pass"
    )
    
    # Цель для финальной компиляции
    add_custom_command(
        OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/${ProjectId}.pdf
        COMMAND ${PDFLATEX_COMPILER} -interaction=nonstopmode ${MAIN_TEX}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/${ProjectId}.aux
        COMMENT "Final LaTeX pass"
    )
    
    # Главная цель сборки
    add_custom_target(${ProjectId} ALL
        DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/${ProjectId}.pdf
    )
    
    # Перенос результатов в bin/
    add_custom_command(
        TARGET ${ProjectId}
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/${ProjectId}.pdf ${LATEX_OUTPUT_PATH}/${ProjectId}.pdf
        COMMAND ${CMAKE_COMMAND} -E remove ${CMAKE_CURRENT_SOURCE_DIR}/${ProjectId}.aux
        COMMAND ${CMAKE_COMMAND} -E remove ${CMAKE_CURRENT_SOURCE_DIR}/${ProjectId}.log
    )
    
else()
    message(STATUS "-- ${ProjectId} - SKIPPED (LaTeX disabled)")
endif()