cmake_minimum_required (VERSION 2.8)
cmake_policy(VERSION 2.8)

project (LearnOpenGL)

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Debug CACHE STRING "Choose the type of build (Debug or Release)" FORCE)
ENDIF(NOT CMAKE_BUILD_TYPE)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules/")

if(WIN32)
	set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)
endif(WIN32)

link_directories(${CMAKE_SOURCE_DIR}/lib)

list(APPEND CMAKE_CXX_FLAGS "-std=c++11")

# find the required packages
find_package(GLM REQUIRED)
message(STATUS "GLM included at ${GLM_INCLUDE_DIR}")
find_package(GLFW3 REQUIRED)
message(STATUS "Found GLFW3 in ${GLFW3_INCLUDE_DIR}")
find_package(ASSIMP REQUIRED)
message(STATUS "Found ASSIMP in ${ASSIMP_INCLUDE_DIR}")
find_package(SOIL REQUIRED)
message(STATUS "Found SOIL in ${SOIL_INCLUDE_DIR}")
find_package(GLEW REQUIRED)
message(STATUS "Found GLEW in ${GLEW_INCLUDE_DIR}")

if(WIN32)
  set(LIBS glfw3 opengl32 glew32s SOIL assimp)
elseif(UNIX)
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -Wall")
  # Linux packages native to CMake
  find_package(OpenGL REQUIRED)
  set(LIBS ${OPENGL_gl_LIBRARY}) # setting LIBS for the first time
  add_definitions(${OPENGL_DEFINITIONS})
  find_package(X11 REQUIRED)
  list(APPEND LIBS ${X11_Xrandr_LIB} ${X11_Xxf86vm_LIB} ${X11_Xi_LIB})
  find_library(RT_LIB rt)
  list(APPEND LIBS ${RT_LIB})
  # append non-native packages
  list(APPEND LIBS ${GLFW3_LIBRARY})
  list(APPEND LIBS ${GLEW_LIBRARY})
  list(APPEND LIBS ${SOIL_LIBRARY})
  list(APPEND LIBS ${ASSIMP_LIBRARY})
else()
  set(LIBS )
endif(WIN32)

IF(APPLE)
  INCLUDE_DIRECTORIES(/System/Library/Frameworks)
  FIND_LIBRARY(COCOA_LIBRARY Cocoa)
  FIND_LIBRARY(OpenGL_LIBRARY OpenGL)
  FIND_LIBRARY(IOKit_LIBRARY IOKit)
  MARK_AS_ADVANCED(COCOA_LIBRARY OpenGL_LIBRARY)
  SET(APPLE_LIBS ${COCOA_LIBRARY} ${IOKit_LIBRARY} ${OpenGL_LIBRARY})
  SET(APPLE_LIBS ${APPLE_LIBS} /usr/local/lib/libglfw.a)
  set(LIBS ${LIBS} ${APPLE_LIBS})
ENDIF(APPLE)

set(CHAPTERS
    1.getting_started
    2.lighting
    3.model_loading
    4.advanced_opengl
)

set(1.getting_started   
    1.hello_window
    2.hello_triangle
    3.shaders
    4.textures
    5.transformations
    6.coordinate_systems
    7.camera
)

set(2.lighting
    1.colors
    2.basic_lighting
    3.materials
    4.lighting_maps
    5.light_casters
    6.multiple_lights
)

set(3.model_loading
    1.model_loading
)

set(4.advanced_opengl
    1.depth_testing
    2.stencil_testing
    3.1.blending_discard
    3.2.blending_sort
    5.framebuffers
    6.cubemaps
    8.advanced_glsl
    9.geometry_shader
    10.instancing
    11.anti_aliasing
)


foreach(CHAPTER ${CHAPTERS})
    foreach(DEMO ${${CHAPTER}})
        file(GLOB SOURCE
            "src/${CHAPTER}/${DEMO}/*.h"
            "src/${CHAPTER}/${DEMO}/*.cpp"
        )
        add_executable(${DEMO} ${SOURCE})
        target_link_libraries(${DEMO} ${LIBS})
		if(WIN32)
        	set_target_properties(${DEMO} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${CHAPTER}")
		elseif(UNIX)
			set_target_properties(${DEMO} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/bin/${CHAPTER}")
		endif(WIN32)
        # copy shader files to build directory
        file(GLOB SHADERS 
                 "src/${CHAPTER}/${DEMO}/*.vs"
                 "src/${CHAPTER}/${DEMO}/*.frag"
                 "src/${CHAPTER}/${DEMO}/*.gs"
        )
        foreach(SHADER ${SHADERS})
			if(WIN32)
            	# configure_file(${SHADER} "test")
            	add_custom_command(TARGET ${DEMO} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E copy ${SHADER} $<TARGET_FILE_DIR:${DEMO}>)
			elseif(UNIX)
				file(COPY ${SHADER} DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bin/${CHAPTER})			
			endif(WIN32)

        endforeach(SHADER)
      # if compiling for visual studio, also use configure file for each project (specifically to set up working directory)
      if(MSVC)
          configure_file(${CMAKE_SOURCE_DIR}/configuration/visualstudio.vcxproj.user.in ${CMAKE_CURRENT_BINARY_DIR}/${DEMO}.vcxproj.user @ONLY)
      endif(MSVC)
    endforeach(DEMO)
endforeach(CHAPTER)

include_directories(${CMAKE_SOURCE_DIR}/includes)
