cmake_minimum_required(VERSION 3.3)
project(jxrlib)
include(flags OPTIONAL)
set(lib_name ${PROJECT_NAME})
#######################################
set(JXRCommon_srcs
  image/sys/adapthuff.c
  image/sys/ansi.h
  image/sys/common.h
  image/sys/image.c
  image/sys/perfTimer.h
  image/sys/perfTimerANSI.c
  image/sys/strcodec.c
  image/sys/strcodec.h
  image/sys/strPredQuant.c
  image/sys/strTransform.c
  image/sys/strTransform.h
  image/sys/windowsmediaphoto.h
  image/sys/xplatform_image.h
  )
source_group(JXRCommon FILES ${JXRCommon_srcs})
list(APPEND ${lib_name}_libsrcs ${JXRCommon_srcs})
###################
set(JXRDecode_srcs
  image/decode/decode.c
  image/decode/decode.h
  image/decode/JXRTranscode.c
  image/decode/postprocess.c
  image/decode/segdec.c
  image/decode/strdec.c
  image/decode/strdec_x86.c
  image/decode/strInvTransform.c
  image/decode/strPredQuantDec.c
  )
source_group(JXRDecode FILES ${JXRDecode_srcs})
list(APPEND ${lib_name}_libsrcs ${JXRDecode_srcs})
###################
set(JXREncode_srcs
  image/encode/encode.c
  image/encode/encode.h
  image/encode/segenc.c
  image/encode/strenc.c
  image/encode/strenc_x86.c
  image/encode/strFwdTransform.c
  image/encode/strPredQuantEnc.c
  )
source_group(JXREncode FILES ${JXREncode_srcs})
list(APPEND ${lib_name}_libsrcs ${JXREncode_srcs})
###################
set(Include_srcs
  common/include/guiddef.h
  common/include/wmsal.h
  common/include/wmspecstring.h
  common/include/wmspecstrings_adt.h
  common/include/wmspecstrings_strict.h
  common/include/wmspecstrings_undef.h
  )
source_group(Include FILES ${Include_srcs})
list(APPEND ${lib_name}_libsrcs ${Include_srcs})
###################
set(JXRGlue_srcs
  jxrgluelib/JXRGlue.c
  jxrgluelib/JXRGlue.h
  jxrgluelib/JXRGlueJxr.c
  jxrgluelib/JXRGluePFC.c
  jxrgluelib/JXRMeta.c
  jxrgluelib/JXRMeta.h
  )
source_group(JXRGlue FILES ${JXRGlue_srcs})
list(APPEND ${lib_name}_libsrcs ${JXRGlue_srcs})
###################
set(JXRTest_srcs
  jxrtestlib/JXRTest.c
  jxrtestlib/JXRTest.h
  jxrtestlib/JXRTestBmp.c
  jxrtestlib/JXRTestHdr.c
  jxrtestlib/JXRTestPnm.c
  jxrtestlib/JXRTestTif.c
  jxrtestlib/JXRTestYUV.c
  )
source_group(JXRTest FILES ${JXRTest_srcs})
list(APPEND ${lib_name}_libsrcs ${JXRTest_srcs})
#######################################
set(verString "^JXR_VERSION=+([0-9]+)\\.([0-9]+)?")
set(verFile Makefile)
file(STRINGS ${verFile} VER REGEX "${verString}")
string(REGEX REPLACE "${verString}" "\\1.\\2" VER ${VER})
if(NOT DEFINED JXR_VER)
  set(JXR_VER ${VER})
elseif(NOT ${JXR_VER} STREQUAL ${VER})
  message(AUTHOR_WARNING "version passed in (${JXR_VER}) doesn't match version from ${verFile} (${VER})")
endif()
set(ver _${JXR_VER})
set(verDir /${PROJECT_NAME}${ver})
#######################################
add_library(${lib_name} STATIC ${${lib_name}_libsrcs})
target_include_directories(${lib_name} PUBLIC $<INSTALL_INTERFACE:$<INSTALL_PREFIX>/include${verDir}>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/jxrtestlib>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/jxrgluelib>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/image/sys>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/common/include>
  )
target_compile_definitions(${lib_name} PRIVATE -DDISABLE_PERF_MEASUREMENT)
if(CMAKE_COMPILER_IS_GNUCXX OR "${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
  target_compile_definitions(${lib_name} PRIVATE -D__ANSI__)
endif()
set_target_properties(${lib_name} PROPERTIES OUTPUT_NAME ${lib_name}${ver})
include(CheckLibraryExists)
check_library_exists(m pow "${CMAKE_LIBRARY_PATH}" HAVE_LIBM)
if(HAVE_LIBM)
  target_link_libraries(${lib_name} PRIVATE m)
endif()
#######################################
set(exes JXRDec420 JxrDecApp JxrEnc420 JxrEncApp)
foreach(exe ${exes})
  add_executable(${exe} jxrencoderdecoder/${exe}.c)
  target_link_libraries(${exe} PRIVATE ${lib_name})
endforeach()
#######################################
set(lib_hdrs
  "${Include_srcs}"
  image/sys/windowsmediaphoto.h
  jxrgluelib/JXRGlue.h
  jxrgluelib/JXRMeta.h
  )
set(targetsFile ${PROJECT_NAME}${ver}-targets)
install(TARGETS ${lib_name} EXPORT ${targetsFile}
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  )
install(FILES ${lib_hdrs} DESTINATION include${verDir}/${PROJECT_NAME})
if(DEFINED XP_NAMESPACE)
  set(nameSpace NAMESPACE ${XP_NAMESPACE}::)
endif()
install(EXPORT ${targetsFile} DESTINATION lib/cmake ${nameSpace})
