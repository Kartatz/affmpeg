cmake_minimum_required(VERSION 3.19)

project(
	something
	VERSION 0.1
	DESCRIPTION "..."
	HOMEPAGE_URL "..."
	LANGUAGES C CXX
)

set(CMAKE_POLICY_DEFAULT_CMP0069 NEW)
set(CMAKE_POLICY_DEFAULT_CMP0048 NEW)
set(CMAKE_POLICY_DEFAULT_CMP0077 NEW)

set(CMAKE_PLATFORM_NO_VERSIONED_SONAME ON)

set(SHARED_LIBRARY_DIRECTORY "lib")
set(SOVERSION_SEPARATOR ".")

set(FFMPEG_SOURCE_DIRECTORY "${CMAKE_SOURCE_DIR}/submodules/ffmpeg")
set(FFMPEG_BUILD_DIRECTORY "${CMAKE_BINARY_DIR}/ffmpeg-build")
set(FFMPEG_INSTALL_PREFIX "${FFMPEG_BUILD_DIRECTORY}/binaries")

set(FFMPEG_INCLUDE_DIRS "${FFMPEG_INSTALL_PREFIX}/include")

set(
	FFMPEG_CONFIGURE_FLAGS
	--disable-all
	--disable-alsa
	--disable-appkit
	--disable-asm
	--disable-audiotoolbox
	--disable-avfoundation
	--disable-bzlib
	--disable-coreimage
	--disable-debug
	--disable-doc
	--disable-iconv
	--disable-libxcb
	--disable-lzma
	--disable-neon
	--disable-network
	--disable-schannel
	--disable-sdl2
	--disable-securetransport
	--disable-static
	--disable-symver
	--disable-videotoolbox
	--disable-vulkan
	--disable-w32threads
	--disable-xlib
	--disable-zlib
	--enable-libvpx
	--enable-avcodec
	--enable-avformat
	--enable-avutil
	--enable-swresample
	--enable-swscale
	--enable-pthreads
	--enable-decoder=libvpx_vp9,opus,mp3,h264,mpeg4,mjpeg,gif
	--enable-encoder=libvpx_vp9
	--enable-demuxer=alac,mov,gif,ogg,matroska,mp3
	--enable-muxer=matroska
	--enable-bsf=vp9_superframe,vp9_raw_reorder
	--enable-hwaccels
	--enable-pic
	--enable-protocol=file
	--enable-shared
	--enable-small
	--enable-version3
	--extra-cflags=-w
	--extra-ldflags=-fPIC
	--prefix="${FFMPEG_INSTALL_PREFIX}"
	--cross-prefix="${TOOLCHAIN_PREFIX}-"
	--target-os=android
	--arch="${CMAKE_SYSTEM_PROCESSOR}"
	--as="${ANDROID_TOOLCHAIN_ROOT}/bin/${TOOLCHAIN_PREFIX}-clang"
	--ld="${ANDROID_TOOLCHAIN_ROOT}/bin/${TOOLCHAIN_PREFIX}-clang"
	--cc="${ANDROID_TOOLCHAIN_ROOT}/bin/${TOOLCHAIN_PREFIX}-clang"
	--nm="${CMAKE_NM}"
	--strip="${CMAKE_STRIP}"
)
message("${FFMPEG_CONFIGURE_FLAGS}")
if (NOT EXISTS "${FFMPEG_BUILD_DIRECTORY}")
	file(MAKE_DIRECTORY "${FFMPEG_BUILD_DIRECTORY}")
endif()

if (NOT EXISTS "${FFMPEG_BUILD_DIRECTORY}/Makefile")
	message("-- Configuring FFmpeg")
	
	execute_process(
		COMMAND bash
		"${FFMPEG_SOURCE_DIRECTORY}/configure"
		${FFMPEG_CONFIGURE_FLAGS}
		WORKING_DIRECTORY "${FFMPEG_BUILD_DIRECTORY}"
		COMMAND_ERROR_IS_FATAL ANY
	)
endif()

if (NOT EXISTS "${FFMPEG_INSTALL_PREFIX}")
	add_custom_command(
		OUTPUT "${FFMPEG_INSTALL_PREFIX}"
		COMMAND make
		COMMAND make install
		WORKING_DIRECTORY "${FFMPEG_BUILD_DIRECTORY}"
		COMMENT "-- Building FFmpeg"
	)
endif()

add_custom_target(ffmpeg ALL DEPENDS "${FFMPEG_INSTALL_PREFIX}")

add_subdirectory(../alibvpx EXCLUDE_FROM_ALL)