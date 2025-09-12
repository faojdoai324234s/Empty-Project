cmake_minimum_required( VERSION 3.19 )

set( library_name Empty-Project )

project( ${library_name} CXX )

set( CMAKE_INCLUDE_CURRENT_DIR ON )
set( CMAKE_CXX_STANDARD 17 )
set( CMAKE_CXX_STANDARD_REQUIRED ON )
set( CMAKE_CXX_EXTENSIONS OFF )
set( CMAKE_AUTOMOC ON )
set( CMAKE_AUTORCC ON )

if( MSVC )
	add_compile_options(
    $<$<CONFIG:>:/MT> #---------|
    $<$<CONFIG:Debug>:/MTd> #---|-- Statically link the runtime libraries
    $<$<CONFIG:Release>:/MT> #--|
  )
endif()

set( OPENSSL_USE_STATIC_LIBS TRUE )
if( MSVC )
	set( OPENSSL_MSVC_STATIC_RT TRUE )
endif()

find_package( Qt5 REQUIRED COMPONENTS Core Gui Network Svg Widgets )
find_package( OpenSSL REQUIRED )
if( UNIX AND NOT APPLE )
  find_package( botan CONFIG REQUIRED )
endif()

file( GLOB SRC_FILES
  ${PROJECT_SOURCE_DIR}/src/*.cpp
  ${PROJECT_SOURCE_DIR}/src/*.hpp
  ${PROJECT_SOURCE_DIR}/src/*.qrc
  ${PROJECT_SOURCE_DIR}/src/player/*.cpp
  ${PROJECT_SOURCE_DIR}/src/player/*.hpp
  ${PROJECT_SOURCE_DIR}/src/player/internal/*.cc
  ${PROJECT_SOURCE_DIR}/src/player/internal/*.cpp
  ${PROJECT_SOURCE_DIR}/src/player/internal/*.h
  ${PROJECT_SOURCE_DIR}/src/player/internal/*.hpp
)

link_directories( ${PROJECT_SOURCE_DIR}/lib )

add_executable( ${library_name} ${SRC_FILES} )

set_property( DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${library_name} )

if( UNIX AND NOT APPLE )
  set( LINK_LIBS
  # libavcodec.so libavdevice.so libavfilter.so libavformat.so libavutil.so libpostproc.so libswresample.so libswscale.so
  libprotobuf.so
  Botan::Botan-static
  )
elseif( APPLE )
  set( LINK_LIBS
  # libavcodec.dylib libavdevice.dylib libavfilter.dylib libavformat.dylib libavutil.dylib libpostproc.dylib libswresample.dylib libswscale.dylib
  )
elseif( WIN32 )
  set( LINK_LIBS
  # avcodec.lib avdevice.lib avfilter.lib avformat.lib avutil.lib postproc.lib swresample.lib swscale.lib
  libprotobuf
  botan
  Crypt32 Ws2_32
  )
endif()

target_link_libraries( ${library_name} PRIVATE OpenSSL::Crypto OpenSSL::SSL Qt5::Core Qt5::Gui Qt5::Network Qt5::Svg Qt5::Widgets ${LINK_LIBS} )

target_include_directories( ${library_name} PRIVATE ${CMAKE_CURRENT_BINARY_DIR} ${PROJECT_SOURCE_DIR}/include )
