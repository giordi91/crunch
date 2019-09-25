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
	target_include_directories(${PROJECT_NAME} PUBLIC ${INCLUDES_FILES})

	if(APPLE)
	elseif(UNIX)
	elseif(WIN32)
		#copying built lib to the output
		add_custom_command(
		TARGET ${PROJECT_NAME} 
		POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy
		  $<TARGET_FILE:${PROJECT_NAME}>
		  ${CMAKE_SOURCE_DIR}/../builtDependencies/windows/$<CONFIGURATION>/$<TARGET_FILE_NAME:${PROJECT_NAME}>)		
	endif()

    file(GLOB_RECURSE TO_COPY  "crnlib/*.h" )
    file(GLOB_RECURSE TO_COPY_2  "inc/*.h" )
	file(COPY ${TO_COPY} DESTINATION ${CMAKE_SOURCE_DIR}/../builtDependencies/includes/crnlib )
	file(COPY ${TO_COPY_2} DESTINATION ${CMAKE_SOURCE_DIR}/../builtDependencies/includes/crnlib )
