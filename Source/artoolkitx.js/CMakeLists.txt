include_directories(${PROJECT_SOURCE_DIR}/ARX/include)
include_directories(${PROJECT_SOURCE_DIR}/ARX)
include_directories(${PROJECT_SOURCE_DIR}/ARX/AR/include)
include_directories(${PROJECT_SOURCE_DIR}/ARX/AR2/include)
include_directories(${PROJECT_SOURCE_DIR}/ARX/ARUtil/include)
include_directories(${PROJECT_SOURCE_DIR}/ARX/ARVideo/include)
include_directories(${PROJECT_SOURCE_DIR}/ARX/ARG/include)
include_directories(${PROJECT_SOURCE_DIR}/ARX/KPM/include)

include_directories(${CMAKE_CURRENT_BINARY_DIR}/../ARX/AR/include)

add_definitions("-std=c++11")
set(OPENCV_EM "../../depends/emscripten/build_opencv-em")

link_directories(${CMAKE_CURRENT_BINARY_DIR}/../ARX ${CMAKE_CURRENT_BINARY_DIR}/../ARX/AR ${CMAKE_CURRENT_BINARY_DIR}/../ARX/ARUtil ${CMAKE_CURRENT_BINARY_DIR}/../ARX/AR2 ${CMAKE_CURRENT_BINARY_DIR}/../ARX/KPM ${CMAKE_CURRENT_BINARY_DIR}/../ARX/ARVideo ${CMAKE_CURRENT_BINARY_DIR}/../ARX/ARG ${CMAKE_CURRENT_BINARY_DIR}/../ARX/OCVT ${CMAKE_CURRENT_BINARY_DIR}/${OPENCV_EM}/lib ${CMAKE_CURRENT_BINARY_DIR}/${OPENCV_EM}/3rdparty/lib)
link_libraries(libARX libARVideo libAR libARG libARUtil libAR2 libKPM libOCVT libopencv_highgui libopencv_imgcodecs libopencv_videoio libopencv_calib3d libopencv_features2d libopencv_flann libopencv_imgproc libopencv_video libopencv_core libpng libjpeg libwebp libprotobuf IlmImf zlib)
add_executable(artoolkitx ${PROJECT_SOURCE_DIR}/artoolkitx.js/ARX_js.cpp ${PROJECT_SOURCE_DIR}/artoolkitx.js/ARX_bindings.cpp)

set(EXPORT_FUNCTIONS "'_arwUpdateAR', '_arwCapture', '_arwGetProjectionMatrix', '_arwQueryTrackableVisibilityAndTransformation', '_arwGetTrackablePatternConfig', '_arwGetTrackablePatternImage', '_arwLoadOpticalParams'")
set(EXPORTED_RUNTIME_FUNCTIONS "'ccall', 'cwrap', 'FS', 'setValue'")
set(MODULE_OPTIONS "-s MODULARIZE_INSTANCE=1 -s EXPORT_NAME='artoolkitXjs' -s EXPORT_ES6=1" )

#math(EXPR MEM 512*1024*1024)
set(MEM 16777216) #This is default value
message("CMAKE_BUILD_TYPE: ${CMAKE_BUILD_TYPE}")
set(MEMORY "-s TOTAL_MEMORY=${MEM} -s ALLOW_MEMORY_GROWTH=1")

set(WASM_LOADER "--pre-js ${PROJECT_SOURCE_DIR}/artoolkitx.js/ARX_wasmLoader.js")
#To generate source maps the -g4 needs to be passed to the LINK_FLAGS
if(${CMAKE_BUILD_TYPE} STREQUAL "Debug")
    # https://github.com/kripken/emscripten/blob/master/src/settings.js#L85
    # set(TOTAL_STACK "-s TOTAL_STACK=5*1024*1024)"
    set(DEBUG_EMCC "-g4 -s SAFE_HEAP=1 -s ASSERTIONS=1 --js-opts 0 -s DEMANGLE_SUPPORT=0" )
    # set(NO_WASM "-s WASM=0")
else()
    # set(OPTIMIZE_EMCC "--memory-init-file 0 -s INVOKE_RUN=0 --llvm-lto 1 -s ASSERTIONS=0 -s PRECISE_I64_MATH=0 -s PRECISE_F32=2")
    # set(NO_WASM "-s WASM=0")
    # If WASM add a pre-js to register the wasm loader
endif()

# set(OPTIMIZE_OPTIONS "-s NO_EXIT_RUNTIME=1 -s INVOKE_RUN=0")
set_target_properties(artoolkitx PROPERTIES LINK_FLAGS "${OPTIMIZE_OPTIONS} ${MODULE_OPTIONS} ${NO_WASM} ${WASM_LOADER} ${MEMORY} ${DEBUG_EMCC} ${OPTIMIZE_EMCC} --bind --post-js ${PROJECT_SOURCE_DIR}/artoolkitx.js/ARX_additions.js -s EXPORTED_FUNCTIONS='[${EXPORT_FUNCTIONS}]' -s EXTRA_EXPORTED_RUNTIME_METHODS='[${EXPORTED_RUNTIME_FUNCTIONS}]'")

if(${CMAKE_BUILD_TYPE} STREQUAL "Debug")
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/artoolkitx.js
    DESTINATION ${PROJECT_SOURCE_DIR}/../SDK/lib
)
else()
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/artoolkitx.js ${CMAKE_CURRENT_BINARY_DIR}/artoolkitx.wasm
    DESTINATION ${PROJECT_SOURCE_DIR}/../SDK/lib
)
endif()
