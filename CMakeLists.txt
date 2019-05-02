cmake_minimum_required(VERSION 3.13.0)

project(crunch)   

    #looking for  files
    file(GLOB_RECURSE SOURCE_FILES "crnlib/*.cpp" "crnlib/*.h" )
	list(REMOVE_ITEM SOURCE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/crnlib/lzham_timer.cpp")
	list(REMOVE_ITEM SOURCE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/crnlib/lzham_win32_threading.cpp")
	

    message (STATUS ${CMAKE_CURRENT_SOURCE_DIR})
    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/crnlib
	)

	add_compile_definitions(  _LIB _UNICODE)

	#making sure to add the common cpp flags, that are defined in the main cpp file
    set(COMMON_CXX_FLAGS "${COMMON_CXX_FLAGS}  /std:c++17 /W4 /arch:AVX2 /fp:fast /MP")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${COMMON_CXX_FLAGS}")


	MACRO(SOURCE_GROUP_BY_FOLDER target)
	  SET(SOURCE_GROUP_DELIMITER "/")
	  SET(last_dir "")
	  SET(files "")
	  FOREACH(file ${SOURCE_FILES})
		file(RELATIVE_PATH relative_file "${PROJECT_SOURCE_DIR}/vendors/{target}" ${file})
		GET_FILENAME_COMPONENT(dir "${relative_file}" PATH)
		IF (NOT "${dir}" STREQUAL "${last_dir}")
		  IF (files)
			SOURCE_GROUP("${last_dir}" FILES ${files})
		  ENDIF (files)
		  SET(files "")
		ENDIF (NOT "${dir}" STREQUAL "${last_dir}")
		SET(files ${files} ${file})
		SET(last_dir "${dir}")
	  ENDFOREACH(file)
	  IF (files)
		SOURCE_GROUP("${last_dir}" FILES ${files})
	  ENDIF (files)
	ENDMACRO(SOURCE_GROUP_BY_FOLDER)

	SOURCE_GROUP_BY_FOLDER(crunch)


    #adding the executable
    add_library(${PROJECT_NAME} STATIC ${SOURCE_FILES} ${INCLUDES_FILES})

	#setting the plug-in folder in the solution
	set_target_properties (${PROJECT_NAME} PROPERTIES
    FOLDER vendors)

	SET(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)

	#target_link_libraries(${PROJECT_NAME} d3dcompiler DXGI D3D12 imgui)

    file(GLOB_RECURSE TO_COPY  "crnlib/*.h" )
    file(GLOB_RECURSE TO_COPY_2  "inc/*.h" )
	file(COPY ${TO_COPY} DESTINATION "${CMAKE_SOURCE_DIR}/ResourceCompilerPlugins/TextureCompilerPlugin/vendors/includes/crnlib" )
	file(COPY ${TO_COPY_2} DESTINATION "${CMAKE_SOURCE_DIR}/ResourceCompilerPlugins/TextureCompilerPlugin/vendors/includes/crnlib" )
