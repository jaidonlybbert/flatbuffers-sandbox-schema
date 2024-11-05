include(CMakePrintHelpers)
cmake_minimum_required(VERSION 3.16)

add_subdirectory(flatbuffers)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)
project(SandboxSchema LANGUAGES CXX)

# generate compile_commands.json
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

add_library(sbx_schema INTERFACE)

# Generate schema code interface using flatbuffers compiler
# function(compile_schema_for_sbx SRC_FBS OPT)
# 	compile_schema("${SRC_FBS}" "${OPT}" GEN_FILE)
# 	target_sources(sbx_schema PRIVATE ${GEN_FILE})
# endfunction()

# compile_schema_for_sbx(src/Input.fbs "${FLATC_OPT_COMP}")
# compile_schema_for_sbx(src/Types.fbs "${FLATC_OPT_COMP}")

set(fbs_files
Input.fbs
Types.fbs
)

set(FLATBUFFERS_FLATC_EXECUTABLE ${CMAKE_CURRENT_SOURCE_DIR}/build/flatbuffers/flatc)

set(fbs_path ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(FLATC_OPT --cpp --filename-suffix \"\")
cmake_print_variables(fbs_files)
cmake_print_variables(fbs_path)
cmake_print_variables(FLATBUFFERS_FLATC_EXECUTABLE)
# --gen-mutable --gen-object-api --reflect-names)
foreach(fbs_file ${fbs_files})
	string(REGEX REPLACE "\\.fbs$" ".h" GEN_HEADER ${fbs_file})
	cmake_print_variables(fbs_file)
	cmake_print_variables(GEN_HEADER)
	add_custom_command(
	    OUTPUT ${GEN_HEADER}
	    COMMAND "${FLATBUFFERS_FLATC_EXECUTABLE}"
	      ${FLATC_OPT}
	      -o "${CMAKE_CURRENT_SOURCE_DIR}/include"
	      "${fbs_path}/${fbs_file}"
	    DEPENDS "${fbs_path}/${fbs_file}"
	    COMMENT "flatc generation: `${fbs_file}` -> `${GEN_HEADER}`"
	)
	list(APPEND schema_outputs "${CMAKE_CURRENT_SOURCE_DIR}/include/${GEN_HEADER}")
	cmake_print_variables(schema_outputs)
endforeach()
add_custom_target(schema_compile ALL DEPENDS ${schema_outputs} VERBATIM)

# target_include_directories(schema INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include)
# add_library(schema STATIC generated/Input_generated.h)
target_link_libraries(sbx_schema INTERFACE FlatBuffers)
target_include_directories(sbx_schema INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include)
add_dependencies(sbx_schema schema_compile)
