cmake_minimum_required(VERSION 2.8.12)

project(arachne-pnr)

if(WIN32)
    execute_process(COMMAND powershell "git log --oneline | wc -l" OUTPUT_VARIABLE COMMIT_COUNT OUTPUT_STRIP_TRAILING_WHITESPACE)
    execute_process(COMMAND powershell "git diff --name-only HEAD | wc -l" OUTPUT_VARIABLE LOCAL_CHANGED OUTPUT_STRIP_TRAILING_WHITESPACE)
    execute_process(COMMAND powershell "git rev-parse --verify --short HEAD" OUTPUT_VARIABLE GIT_REV OUTPUT_STRIP_TRAILING_WHITESPACE)
else()
    execute_process(COMMAND echo `git log --oneline | wc -l` OUTPUT_VARIABLE COMMIT_COUNT OUTPUT_STRIP_TRAILING_WHITESPACE)
    execute_process(COMMAND echo `git diff --name-only HEAD | wc -l` OUTPUT_VARIABLE LOCAL_CHANGED OUTPUT_STRIP_TRAILING_WHITESPACE)
    execute_process(COMMAND git rev-parse --verify --short HEAD OUTPUT_VARIABLE GIT_REV OUTPUT_STRIP_TRAILING_WHITESPACE)
endif()

configure_file(version.cc.in version.cc)

set(ARACHNE_PNR_SOURCES
    src/arachne-pnr.cc
    src/blif.cc
    src/chipdb.cc
    src/configuration.cc
    src/constant.cc
    src/designstate.cc
    src/global.cc
    src/io.cc
    src/line_parser.cc
    src/location.cc
    src/netlist.cc
    src/pack.cc
    src/pcf.cc
    src/place.cc
    src/route.cc
    src/util.cc
    ${CMAKE_CURRENT_BINARY_DIR}/version.cc
)

add_executable(arachne-pnr ${ARACHNE_PNR_SOURCES})

set_target_properties(arachne-pnr PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)
set_target_properties(arachne-pnr PROPERTIES RUNTIME_OUTPUT_DIRECTORY_DEBUG ${PROJECT_SOURCE_DIR}/bin)
set_target_properties(arachne-pnr PROPERTIES RUNTIME_OUTPUT_DIRECTORY_RELEASE ${PROJECT_SOURCE_DIR}/bin)
set_target_properties(arachne-pnr PROPERTIES RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL ${PROJECT_SOURCE_DIR}/bin)
set_target_properties(arachne-pnr PROPERTIES RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO ${PROJECT_SOURCE_DIR}/bin)

target_include_directories(arachne-pnr PUBLIC src)
if(MSVC)
    target_compile_options(arachne-pnr PUBLIC /W4 /WX /wd4800 /wd4100 /wd4244 /wd4702 /wd4189)
    target_compile_definitions(arachne-pnr PUBLIC _CRT_SECURE_NO_WARNINGS)
else()
    target_compile_options(arachne-pnr PUBLIC -std=c++11 -MD -Wall -Wshadow -Wsign-compare -Werror)
    target_link_libraries(arachne-pnr m)
endif()

add_custom_command(TARGET arachne-pnr POST_BUILD
    COMMAND $<TARGET_FILE:arachne-pnr> -d 1k -c ${ICEBOX}/chipdb-1k.txt --write-binary-chipdb ${PROJECT_SOURCE_DIR}/share/arachne-pnr/chipdb-1k.bin)
add_custom_command(TARGET arachne-pnr POST_BUILD
    COMMAND $<TARGET_FILE:arachne-pnr> -d 8k -c ${ICEBOX}/chipdb-8k.txt --write-binary-chipdb ${PROJECT_SOURCE_DIR}/share/arachne-pnr/chipdb-8k.bin)
