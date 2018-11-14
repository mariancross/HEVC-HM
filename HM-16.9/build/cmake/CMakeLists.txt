### THIS CMAKE FILE COULD BE USED FOR COMPILING HEVC REFERENCE SOFTWARE
### distribution is only allowed with the copyright message below

#**************************************************************************
#   Copyright (C) 2017
#   Institut fuer Nachrichtentechnik
#   RWTH Aachen University
#   Johannes Sauer
#   sauer@ient.rwth-aachen.de
#   Jens Schneider
#   schneider@ient.rwth-aachen.de
#
#**************************************************************************

cmake_minimum_required(VERSION 3.3)
project( HM )

include_directories ("${PROJECT_SOURCE_DIR}/../../source/Lib"  )


if(HIGH_BIT_DEPTH)
   message(STATUS "This is a high bit depth build")
   set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O0 -ggdb")
   add_definitions("-DRExt__HIGH_BIT_DEPTH_SUPPORT=1")
   set(CMAKE_DEBUG_POSTFIX "HighBitDepthStaticd" CACHE STRING "Set Debug library postfix" FORCE)
   set(CMAKE_RELEASE_POSTFIX "HighBitDepthStatic" CACHE STRING "Set Release library postfix" FORCE)
else()
   set(CMAKE_DEBUG_POSTFIX "Staticd" CACHE STRING "Set Debug library postfix" FORCE)
   set(CMAKE_RELEASE_POSTFIX "Static" CACHE STRING "Set Release library postfix" FORCE)
endif()


# set debug and release config as in makefiles which are provided
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -Wuninitialized")
set(CMAKE_CXX_FLAGS_DEBUG "-g  -D_DEBUG")

# collect the files belonging to the different libraries
file(GLOB TLibCommon_SRC
    "${PROJECT_SOURCE_DIR}/../../source/Lib/TLibCommon/*.h"
    "${PROJECT_SOURCE_DIR}/../../source/Lib/TLibCommon/*.cpp"
    "${PROJECT_SOURCE_DIR}/../../source/Lib/libmd5/*.h"
    "${PROJECT_SOURCE_DIR}/../../source/Lib/libmd5/*.c"
)

file(GLOB TAppCommon_SRC
    "${PROJECT_SOURCE_DIR}/../../source/Lib/TAppCommon/*.h"
    "${PROJECT_SOURCE_DIR}/../../source/Lib/TAppCommon/*.cpp"
)
file(GLOB TLibEncoder_SRC
    "${PROJECT_SOURCE_DIR}/../../source/Lib/TLibEncoder/*.h"
    "${PROJECT_SOURCE_DIR}/../../source/Lib/TLibEncoder/*.cpp"
)
file(GLOB TLibDecoder_SRC
    "${PROJECT_SOURCE_DIR}/../../source/Lib/TLibDecoder/*.h"
    "${PROJECT_SOURCE_DIR}/../../source/Lib/TLibDecoder/*.cpp"
)
file(GLOB TLibVideoIO_SRC
    "${PROJECT_SOURCE_DIR}/../../source/Lib/TLibVideoIO/*.h"
    "${PROJECT_SOURCE_DIR}/../../source/Lib/TLibVideoIO/*.cpp"
)

# collect the files belonging to the different executables
file(GLOB TAppDecoder_SRC
    "${PROJECT_SOURCE_DIR}/../../source/App/TAppDecoder/*.h"
    "${PROJECT_SOURCE_DIR}/../../source/App/TAppDecoder/*.cpp"
)
file(GLOB TAppEncoder_SRC
    "${PROJECT_SOURCE_DIR}/../../source/App/TAppEncoder/*.h"
    "${PROJECT_SOURCE_DIR}/../../source/App/TAppEncoder/*.cpp"
)


# stating which libaries to build
add_library(TLibVideoIO STATIC ${TLibVideoIO_SRC})
add_library(TLibCommon STATIC ${TLibCommon_SRC})
add_library(TLibDecoder STATIC ${TLibDecoder_SRC})
add_library(TLibDecoderAnalyser STATIC ${TLibDecoder_SRC})
target_compile_definitions(TLibDecoderAnalyser  PUBLIC  RExt__DECODER_DEBUG_BIT_STATISTICS=1)
add_library(TLibEncoder STATIC ${TLibEncoder_SRC})
add_library(TAppCommon STATIC ${TAppCommon_SRC})


# state which binaries to build, you can add additional libraries to link with
add_executable( TAppDecoder ${TAppDecoder_SRC})
target_link_libraries(TAppDecoder TLibDecoder TLibCommon  TLibVideoIO TAppCommon dl pthread )

add_executable( TAppDecoderAnalyser ${TAppDecoder_SRC})
target_link_libraries(TAppDecoderAnalyser TLibDecoderAnalyser TLibCommon  TLibVideoIO TAppCommon dl pthread )
target_compile_definitions(TAppDecoderAnalyser PUBLIC  RExt__DECODER_DEBUG_BIT_STATISTICS=1)

add_executable( TAppEncoder ${TAppEncoder_SRC})
find_package(TensorflowCC COMPONENTS Shared)
if(TensorflowCC_Shared_FOUND)
    target_link_libraries( TAppEncoder LINK_PUBLIC TensorflowCC::Shared TLibEncoder TLibCommon TLibVideoIO TAppCommon dl pthread )
else()
    find_package(TensorflowCC REQUIRED COMPONENTS Static)
    target_link_libraries( TAppEncoder LINK_PUBLIC TensorflowCC::Static TLibEncoder TLibCommon TLibVideoIO TAppCommon dl pthread )
endif()

if(NOT HIGH_BIT_DEPTH)
    add_executable( annexBbytecount  "${PROJECT_SOURCE_DIR}/../../source/App/utils/annexBbytecount.cpp" )
    target_link_libraries( annexBbytecount TLibDecoder TLibCommon TLibVideoIO TAppCommon dl pthread )
    add_executable( convert_NtoMbit_YCbCr  "${PROJECT_SOURCE_DIR}/../../source/App/utils/convert_NtoMbit_YCbCr.cpp" )
    target_link_libraries( convert_NtoMbit_YCbCr TLibDecoder TLibCommon TLibVideoIO TAppCommon dl pthread )
endif()



# set properties for compiling as in makefiles which are provided
set(CommonOptions "-fPIC -DMSYS_LINUX -Wall -Wshadow -Wno-sign-compare")
# warnings as errors
#set(CommonOptions "-fPIC -DMSYS_LINUX -Wall -Wshadow -Wno-sign-compare -Werror")
set(OtherOptions "-D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64 -DMSYS_UNIX_LARGEFILE")

set_target_properties(TLibVideoIO         PROPERTIES COMPILE_FLAGS "${CommonOptions} ${OtherOptions}")
set_target_properties(TLibCommon          PROPERTIES COMPILE_FLAGS "${CommonOptions}")
set_target_properties(TLibDecoder         PROPERTIES COMPILE_FLAGS "${CommonOptions}")
set_target_properties(TLibDecoderAnalyser PROPERTIES COMPILE_FLAGS "${CommonOptions}")
set_target_properties(TLibEncoder         PROPERTIES COMPILE_FLAGS "${CommonOptions}")
set_target_properties(TAppCommon          PROPERTIES COMPILE_FLAGS "${CommonOptions}")
set_target_properties(TAppDecoder         PROPERTIES COMPILE_FLAGS "${CommonOptions} ${OtherOptions}")
set_target_properties(TAppDecoderAnalyser PROPERTIES COMPILE_FLAGS "${CommonOptions} ${OtherOptions}")
set_target_properties(TAppEncoder         PROPERTIES COMPILE_FLAGS "${CommonOptions} ${OtherOptions}")



# add postfixes for executables too
if(HIGH_BIT_DEPTH)
    set_target_properties( TAppDecoder TAppDecoderAnalyser TAppEncoder  PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})
    set_target_properties( TAppDecoder TAppDecoderAnalyser TAppEncoder  PROPERTIES RELEASE_POSTFIX ${CMAKE_RELEASE_POSTFIX})
else()
    set_target_properties( TAppDecoder TAppDecoderAnalyser TAppEncoder annexBbytecount convert_NtoMbit_YCbCr PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})
    set_target_properties( TAppDecoder TAppDecoderAnalyser TAppEncoder annexBbytecount convert_NtoMbit_YCbCr PROPERTIES RELEASE_POSTFIX ${CMAKE_RELEASE_POSTFIX})
endif()


# specify where to put the produced binaries and libraries
if(HIGH_BIT_DEPTH)
   INSTALL(TARGETS
   #libaries
     TLibCommon
     TAppCommon
     TLibEncoder
     TLibDecoder
     TLibDecoderAnalyser
     TLibVideoIO
   #binaries
     TAppDecoder
     TAppDecoderAnalyser
     TAppEncoder
   RUNTIME DESTINATION "${PROJECT_SOURCE_DIR}/../../bin"
   LIBRARY DESTINATION "${PROJECT_SOURCE_DIR}/../../lib"
   ARCHIVE DESTINATION "${PROJECT_SOURCE_DIR}/../../lib"
   )
else()
   INSTALL(TARGETS
   #libaries
     TLibCommon
     TAppCommon
     TLibEncoder
     TLibDecoder
     TLibDecoderAnalyser
     TLibVideoIO
   #binaries
     TAppDecoder
     TAppDecoderAnalyser
     TAppEncoder
     annexBbytecount
     convert_NtoMbit_YCbCr
   RUNTIME DESTINATION "${PROJECT_SOURCE_DIR}/../../bin"
   LIBRARY DESTINATION "${PROJECT_SOURCE_DIR}/../../lib"
   ARCHIVE DESTINATION "${PROJECT_SOURCE_DIR}/../../lib"
   )
endif()

