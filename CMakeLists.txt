cmake_minimum_required(VERSION 3.4.1)

project (OpenGLPractice)

# find_package查找时需要用到CMAKE_MODULE_PATH
# set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules/")

if(WIN32)
	Add_Definitions(-DPLATFORM_WINDOWS=1)
endif(WIN32)

Add_Definitions(-DCSF_ZIP_SUPPORT=1)


list(APPEND CMAKE_CXX_FLAGS "-std=c++11")

set(Include_Path 
	${CMAKE_SOURCE_DIR}/src

	${CMAKE_SOURCE_DIR}/thirdParty/GLFW/include
	${CMAKE_SOURCE_DIR}/thirdParty/glad/include
	${CMAKE_SOURCE_DIR}/thirdParty/stbImage/include
	${CMAKE_SOURCE_DIR}/thirdParty/assimp/include
	${CMAKE_SOURCE_DIR}/thirdParty/imgui
	${CMAKE_SOURCE_DIR}/thirdParty
)

set(Lib_Path
	${CMAKE_SOURCE_DIR}/thirdParty/GLFW/lib
	${CMAKE_SOURCE_DIR}/thirdParty/assimp/lib/x64
)

if(WIN32)
  set(LIBS glfw3 opengl32 assimp-vc140-mt)
endif(WIN32)

configure_file(configuration/root_directory.h.in configuration/root_directory.h)
include_directories(${CMAKE_BINARY_DIR}/configuration)

# first create relevant static libraries requried for other projects
add_library(STB_IMAGE "thirdParty/stbImage/src/stb_image.cpp")
set(LIBS ${LIBS} STB_IMAGE)

# glad
add_library(GLAD "thirdParty/glad/src/glad.c")
set(LIBS ${LIBS} GLAD)

# imgui
file(GLOB IMGUI_SOURCE
	"thirdParty/imgui/*.h"
	"thirdParty/imgui/*.cpp"
)

set(CHAPTERS
    basic
	advance
)

set(basic
	initTriangle
	drawInstance
)

set(advance
	Occlusion
)

include_directories(${Include_Path})
link_directories(${Lib_Path})

foreach(CHAPTER ${CHAPTERS})
    foreach(DEMO ${${CHAPTER}})
        file(GLOB_RECURSE SOURCE
            "src/${CHAPTER}/${DEMO}/*.h"
			"src/${CHAPTER}/${DEMO}/*.hpp"
            "src/${CHAPTER}/${DEMO}/*.cpp"
            "src/${CHAPTER}/${DEMO}/*.vs"
            "src/${CHAPTER}/${DEMO}/*.fs"
            "src/${CHAPTER}/${DEMO}/*.gs"
        )

		file(GLOB_RECURSE COMMON_SOURCE
			"src/common/*.h"
			"src/common/*.hpp"
			"src/common/*.cpp"
		)

		set(NAME "${CHAPTER}__${DEMO}")
		add_executable(${NAME} ${SOURCE} ${COMMON_SOURCE} ${IMGUI_SOURCE})
        target_link_libraries(${NAME} ${LIBS})
		if(WIN32)
			# Set work directory for Visual Studio 
			set_property(TARGET ${NAME} PROPERTY VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/data")
            #set_target_properties(${NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${CHAPTER}")
		endif(WIN32)

		foreach(SHADER ${SHADERS})
            if(WIN32)
                # configure_file(${SHADER} "test")
                add_custom_command(TARGET ${NAME} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E copy ${SHADER} $<TARGET_FILE_DIR:${NAME}>)
            endif(WIN32)
        endforeach(SHADER)

	endforeach(DEMO)
endforeach(CHAPTER)

# 将文件归类到文件夹中
source_group(common FILES 
	${COMMON_SOURCE}
)

source_group(imgui FILES 
	${IMGUI_SOURCE}
)
