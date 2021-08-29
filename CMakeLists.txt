cmake_minimum_required(VERSION 3.9 FATAL_ERROR)
project(optimization VERSION 0.0.1 DESCRIPTION "DlibNetwork with GeneticOptimization v1.0" LANGUAGES C CXX)

set(DEFAULT_BUILD_TYPE "Debug")
#set(CMAKE_BUILD_TYPE "Debug")

set(DEFAULT_BUILD_TYPE "Release")
#set(CMAKE_BUILD_TYPE "Release")

#STNDARD STUFF
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${FLAGS_COMMON}")

if (UNIX)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -lX11 -Wall" )
endif (UNIX)

if (WIN32)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP /O2" )
endif (WIN32)

set(TARGET_NAME "optimization")

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

file(MAKE_DIRECTORY "${CMAKE_BINARY_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")

macro(copy_json jsonname)
	configure_file(${jsonname} ${CMAKE_BINARY_DIR}/${jsonname})
	configure_file(${jsonname} ${CMAKE_BUILD_TYPE}/${jsonname})
	configure_file(${jsonname} ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/${jsonname})
endmacro()

copy_json("DlibModule/configs/config.json")
copy_json("DlibModule/configs/preprocess.json")
copy_json("DlibModule/configs/postprocess.json")
copy_json("DlibModule/configs/dataset.json")

copy_json("GeneticOptimizationModule/configs/config.json")
copy_json("GeneticOptimizationModule/configs/preprocess.json")

copy_json("GeneticOptimizationModule/gtest/test_addmultipledron.json")
copy_json("GeneticOptimizationModule/gtest/test.json")

copy_json("GeneticOptimizationModule/gtest_it/test_dataset.json")
copy_json("GeneticOptimizationModule/gtest_it/test_graph_config_postprocesssing.json")
copy_json("GeneticOptimizationModule/gtest_it/test_graph_config.json")
copy_json("GeneticOptimizationModule/gtest_it/test_graph_postprocessing.json")
copy_json("GeneticOptimizationModule/gtest_it/test_graph.json")
copy_json("GeneticOptimizationModule/gtest_it/test_preprocess.json")

include_directories( "DlibModule/include" )
include_directories( "GeneticOptimizationModule/include" )
include_directories( "GeneticOptimizationModule/gtest" )
include_directories( "GeneticOptimizationModule/gtest_it" )
include_directories( "PostProcessingModules/include" )
include_directories( "ProcessingModules/include" )
include_directories( "DataManagement/include" )
include_directories( "ProcessingGraph/include" )
include_directories( "Utils/include" )
include_directories( "ProcessingModules/src/Subtractor/subtractors" )
include_directories( "ProcessingModules/src/Filter" )



if (WIN32)
	set(QT5_PATH  "C:\\Qt\\5.15.2\\msvc2019_64")
	list(APPEND CMAKE_PREFIX_PATH "C:\\Qt\\5.15.2\\msvc2019_64\\lib\\cmake\\Qt5Core")
	list(APPEND CMAKE_PREFIX_PATH ${QT5_PATH})
	list(APPEND CMAKE_PREFIX_PATH "C:\\repo\\opencv_build")
	#list(APPEND CMAKE_PREFIX_PATH "C:\\repo\\opencvBuild\\bin\\Debug")
	#list(APPEND CMAKE_PREFIX_PATH "C:\\repo\\opencvBuild\\install\\include")
	#list(APPEND CMAKE_PREFIX_PATH "C\repo\\opencvBuild\\install")
	#list(APPEND CMAKE_PREFIX_PATH "C:\\repo\\dlib\\build\\installDebug\\lib\\cmake\\dlib")
endif (WIN32)

if (UNIX)
	find_package(X11 REQUIRED)
endif (UNIX)

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)
find_package(Qt5Core REQUIRED)
find_package(Qt5Test REQUIRED)
find_package(OpenCV REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS})
find_package(OpenCV QUIET COMPONENTS core highgui imgproc imgcodecs videoio)


enable_testing()

macro(clone_build libname)
	configure_file(clone_build/${libname}.in ${libname}-download/CMakeLists.txt)
	execute_process(COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" .
    WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/${libname}-download")
	execute_process(COMMAND "${CMAKE_COMMAND}" --build .
	WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/${libname}-download")
	add_subdirectory("${CMAKE_BINARY_DIR}/${libname}-src"
                 "${CMAKE_BINARY_DIR}/${libname}-build")
endmacro()

clone_build(spdlog)
clone_build(googletest)
# Prevent GoogleTest from overriding our compiler/linker options
# when building with Visual Studio
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)


clone_build(dlib)


#find_package(dlib REQUIRED)

file(GLOB_RECURSE SRC_UTILS Utils/src/*.cc)
file(GLOB_RECURSE HEADERS_UTILS Utils/include/*.h)

set(SRC_GRAPH ProcessingGraph/src/graph.cc)
set(HEADERS_GRAPH ProcessingGraph/include/graph.h)

set(SRC_DATA
	DataManagement/src/data.cc
	DataManagement/src/loaddata.cc
)
set(HEADERS_DATA
	DataManagement/include/data.h
	DataManagement/include/loaddata.h
)

file(GLOB_RECURSE SRC_PROCESSING ProcessingModules/*.cc)
file(GLOB_RECURSE HEADERS_PROCESSING ProcessingModules/*.h)

file(GLOB_RECURSE SRC_POST_PROCESSING PostProcessingModules/*.cc)
file(GLOB_RECURSE HEADERS_POST_PROCESSING PostProcessingModules/*.h)

add_library(utils_lib STATIC
	${SRC_UTILS} ${HEADERS_UTILS})

add_library(data_lib STATIC
	${SRC_DATA} ${HEADERS_DATA})

add_library(graph_lib STATIC
	${SRC_GRAPH} ${HEADERS_GRAPH})

add_library(${PROJECT_NAME}_processing_lib
	${SRC_PROCESSING} ${HEADERS_PROCESSING})

add_library(${PROJECT_NAME}_postprocessing_lib
	${SRC_POST_PROCESSING} ${HEADERS_POST_PROCESSING})

message(" cv_libs:     " ${OpenCV_LIBS})
message(" cv_includes: " ${OPENCV_INCLUDE_DIRS})

target_link_libraries(utils_lib ${QT_LIBRARIES} spdlog Qt5::Core Threads::Threads)
target_link_libraries(data_lib ${QT_LIBRARIES} spdlog Qt5::Core ${OpenCV_LIBS} Threads::Threads)
target_link_libraries(graph_lib ${QT_LIBRARIES} spdlog Qt5::Core ${OpenCV_LIBS} Threads::Threads)
target_link_libraries(${PROJECT_NAME}_processing_lib ${QT_LIBRARIES} spdlog Qt5::Core ${OpenCV_LIBS} Threads::Threads)
target_link_libraries(${PROJECT_NAME}_postprocessing_lib ${QT_LIBRARIES} spdlog Qt5::Core ${OpenCV_LIBS} Threads::Threads)

##### DLIB MODULE:
set(SRC_DLIB_MODULE_MAIN DlibModule/main.cc)
file(GLOB_RECURSE SRC_DLIB_MODULE 
DlibModule/src/train/*.cc 
DlibModule/include/train/*.h
DlibModule/include/network/*.h)

add_library(${PROJECT_NAME}_dlib_train_test_lib STATIC
	${SRC_DLIB_MODULE})

add_executable(${PROJECT_NAME}_dlib_train_test
    ${SRC_DLIB_MODULE_MAIN})

target_link_libraries(${PROJECT_NAME}_dlib_train_test_lib utils_lib data_lib graph_lib dlib::dlib)
target_link_libraries(${PROJECT_NAME}_dlib_train_test_lib ${PROJECT_NAME}_processing_lib ${PROJECT_NAME}_postprocessing_lib)
target_link_libraries(${PROJECT_NAME}_dlib_train_test ${PROJECT_NAME}_dlib_train_test_lib)

##### Genetic:
set(SRC_GENETIC_MAIN GeneticOptimizationModule/main.cc)

set(SRC_GENETIC
	GeneticOptimizationModule/src/genetic/case.cc
	GeneticOptimizationModule/src/genetic/genetic.cc
	GeneticOptimizationModule/src/genetic/geneticoperation.cc
	GeneticOptimizationModule/src/genetic/mainloop.cc)

set(HEADERS_GENETIC
	GeneticOptimizationModule/include/genetic/case.h
	GeneticOptimizationModule/include/genetic/genetic.h
	GeneticOptimizationModule/include/genetic/geneticoperation.h
	GeneticOptimizationModule/include/genetic/mainloop.h
	GeneticOptimizationModule/include/genetic/structures.h)

add_library(${PROJECT_NAME}_genetic_train_test_lib STATIC
	${SRC_GENETIC} ${HEADERS_GENETIC})

add_executable(${PROJECT_NAME}_genetic_train_test
    ${SRC_GENETIC_MAIN})

target_link_libraries(${PROJECT_NAME}_genetic_train_test_lib utils_lib data_lib graph_lib)
target_link_libraries(${PROJECT_NAME}_genetic_train_test_lib ${PROJECT_NAME}_processing_lib ${PROJECT_NAME}_postprocessing_lib)
if (UNIX)
	target_link_libraries(${PROJECT_NAME}_genetic_train_test_lib X11)
endif (UNIX)
target_link_libraries(${PROJECT_NAME}_genetic_train_test ${PROJECT_NAME}_genetic_train_test_lib)

##### Genetic tests:
file(GLOB_RECURSE SRC_HEADERS_TEST GeneticOptimizationModule/gtest/*.cc GeneticOptimizationModule/gtest/*.h)
file(GLOB_RECURSE SRC_HEADERS_TEST_IT GeneticOptimizationModule/gtest_it/*.cc GeneticOptimizationModule/gtest_it/*.h)

add_executable(g_test ${SRC_HEADERS_TEST})
add_executable(g_test_it ${SRC_HEADERS_TEST_IT})


target_link_libraries(g_test ${PROJECT_NAME}_genetic_train_test_lib gtest gmock )
target_link_libraries(g_test_it ${PROJECT_NAME}_genetic_train_test_lib gtest gmock)


##### Test median2:
set(SRC_TEST_MEDIAN2
	GeneticOptimizationModule/test_median2/main.cc)
add_executable(median2 ${SRC_TEST_MEDIAN2})
target_link_libraries(median2 ${PROJECT_NAME}_genetic_train_test_lib)

message("CMAKE_BUILD_TYPE: ${CMAKE_BUILD_TYPE} ")

if (CMAKE_BUILD_TYPE STREQUAL "Debug" AND WIN32)
	set (CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
	set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
endif()

if (CMAKE_BUILD_TYPE STREQUAL "Release" AND WIN32)
	set (CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
	set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
endif()



if (WIN32)
    get_target_property(QT5_QMAKE_EXECUTABLE Qt5::qmake IMPORTED_LOCATION)
    get_filename_component(QT5_WINDEPLOYQT_EXECUTABLE ${QT5_QMAKE_EXECUTABLE} PATH)
    set(QT5_WINDEPLOYQT_EXECUTABLE "${QT5_WINDEPLOYQT_EXECUTABLE}/windeployqt.exe")

    add_custom_command(TARGET ${PROJECT_NAME}_genetic_train_test_lib POST_BUILD
       COMMAND ${QT5_WINDEPLOYQT_EXECUTABLE} --qmldir ${CMAKE_SOURCE_DIR} $<TARGET_FILE_DIR:${PROJECT_NAME}_genetic_train_test_lib>)
endif(WIN32)


if (WIN32)
	set(OPENCV_PREFIX_BUILD_TYPE "")
	message("Copy OpenCV file:" )
	set(OPENCV_VERSION_WINDOWS "453")
	if(${CMAKE_BUILD_TYPE} STREQUAL  "Release")
		set(OPENCV_PREFIX_BUILD_TYPE "")
	endif()
	if(${CMAKE_BUILD_TYPE} STREQUAL  "Debug")
		set(OPENCV_PREFIX_BUILD_TYPE "d")
	endif()

	add_custom_command(TARGET ${PROJECT_NAME}_genetic_train_test_lib POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${QT5_PATH}/bin/Qt5Test${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${QT5_PATH}/bin/Qt5Gui${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${QT5_PATH}/bin/Qt5Core${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_stitching${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_bgsegm${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_tracking${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_videoio${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_video${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_imgcodecs${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_imgproc${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_core${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_calib3d${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_features2d${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_flann${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_dnn${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_highgui${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_highgui${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_stitching${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/

		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_bgsegm${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_tracking${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_videoio${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_video${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_imgcodecs${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_imgproc${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_core${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_calib3d${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_features2d${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_flann${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_dnn${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_highgui${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_highgui${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_gapi${OPENCV_VERSION_WINDOWS}${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/

		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_videoio_ffmpeg${OPENCV_VERSION_WINDOWS}_64.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_cudev${OPENCV_VERSION_WINDOWS}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_structured_light${OPENCV_VERSION_WINDOWS}.dll   ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/
			
	)
endif(WIN32)