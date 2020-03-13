cmake_minimum_required (VERSION 3.11 FATAL_ERROR)

project(libstrophe-examples VERSION 0.1.0 LANGUAGES C)

add_executable (libstrophe-example-bot ${CMAKE_CURRENT_LIST_DIR}/bot.c)
add_executable (libstrophe-example-active ${CMAKE_CURRENT_LIST_DIR}/active.c)
add_executable (libstrophe-example-basic ${CMAKE_CURRENT_LIST_DIR}/basic.c)
add_executable (libstrophe-example-roster ${CMAKE_CURRENT_LIST_DIR}/roster.c)

target_link_libraries(libstrophe-example-bot strophe)
target_link_libraries(libstrophe-example-active strophe)
target_link_libraries(libstrophe-example-basic strophe)
target_link_libraries(libstrophe-example-roster strophe)