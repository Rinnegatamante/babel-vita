cmake_minimum_required(VERSION 2.8)

if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  if(DEFINED ENV{VITASDK})
    set(CMAKE_TOOLCHAIN_FILE "$ENV{VITASDK}/share/vita.toolchain.cmake" CACHE PATH "toolchain file")
  else()
    message(FATAL_ERROR "Please define VITASDK to point to your SDK path!")
  endif()
endif()

project(babel C CXX)
include("${VITASDK}/share/vita.cmake" REQUIRED)
find_package(PkgConfig REQUIRED)
pkg_check_modules(sndfile REQUIRED IMPORTED_TARGET sndfile)
pkg_check_modules(vorbisfile REQUIRED IMPORTED_TARGET vorbisfile)
pkg_check_modules(freetype2 REQUIRED IMPORTED_TARGET freetype2)
pkg_check_modules(libzip REQUIRED IMPORTED_TARGET libzip)
pkg_check_modules(libpng REQUIRED IMPORTED_TARGET libpng)
pkg_check_modules(libwebp REQUIRED IMPORTED_TARGET libwebp)
pkg_check_modules(libjpeg REQUIRED IMPORTED_TARGET libjpeg)
set(VITA_APP_NAME "Babel Rising 3D")
set(VITA_TITLEID  "BABEL003D")
set(VITA_MKSFOEX_FLAGS "-d ATTRIBUTE2=12")

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -Wl,-q,--allow-multiple-definition,--wrap,sceCtrlPeekBufferPositiveExt2 -D_GNU_SOURCE -Wall -O3 -fdiagnostics-color=always -fno-optimize-sibling-calls -mfloat-abi=softfp")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} -std=c++11 -Wno-write-strings -fpermissive -fno-rtti")

add_executable(babel
  loader/main.c
  loader/dialog.c
  loader/so_util.c
  loader/sha1.c
  loader/ctype_patch.c
)

set_target_properties(babel PROPERTIES LINKER_LANGUAGE CXX)

target_link_libraries(babel
  -Wl,--whole-archive pthread -Wl,--no-whole-archive
  PkgConfig::sndfile
  PkgConfig::vorbisfile
  PkgConfig::freetype2
  PkgConfig::libzip
  PkgConfig::libpng
  PkgConfig::libwebp
  PkgConfig::libjpeg
  openal
  freetype
  png
  webp
  jpeg
  mikmod
  z
  zip
  zstd
  lzma
  bz2
  ssl
  crypto
  sndfile
  vorbisfile
  vorbis
  vorbisenc
  modplug
  opusfile
  opus
  ogg
  FLAC
  m
  imgui
  vitaGL
  vitashark
  SceShaccCgExt
  mathneon
  mpg123
  taihen_stub
  kubridge_stub
  SceNet_stub
  SceNetCtl_stub
  SceAppMgr_stub
  SceAppUtil_stub
  SceAudio_stub
  SceAudioIn_stub
  SceCtrl_stub
  SceCommonDialog_stub
  SceDisplay_stub
  SceFios2_stub
  SceGxm_stub
  SceIme_stub
  SceShaccCg_stub
  SceSysmodule_stub
  ScePower_stub
  SceTouch_stub
  SceMotion_stub
  SceHid_stub
  SceVshBridge_stub
  SceKernelDmacMgr_stub
)

vita_create_self(eboot.bin babel UNSAFE)
vita_create_vpk(babel.vpk ${VITA_TITLEID} eboot.bin
  VERSION ${VITA_VERSION}
  NAME ${VITA_APP_NAME}
  FILE ${CMAKE_SOURCE_DIR}/livearea/icon0.png sce_sys/icon0.png
       ${CMAKE_SOURCE_DIR}/livearea/pic0.png sce_sys/pic0.png
       ${CMAKE_SOURCE_DIR}/livearea/bg0.png sce_sys/livearea/contents/bg0.png
       ${CMAKE_SOURCE_DIR}/livearea/startup.png sce_sys/livearea/contents/startup.png
       ${CMAKE_SOURCE_DIR}/livearea/template.xml sce_sys/livearea/contents/template.xml
)

set_property(SOURCE loader/so_util.c APPEND PROPERTY COMPILE_OPTIONS -Wno-error=return-mismatch -Wno-error=incompatible-pointer-types)
