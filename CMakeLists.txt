cmake_minimum_required(VERSION 3.1)
cmake_policy(SET CMP0048 NEW)

project(PBRT
  LANGUAGES C CXX
  DESCRIPTION "PBRT-v2"
  VERSION 0.0.1.0)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

option(WERROR "Option to disable the -Werror flag." ON)
option(LTO "Option to enable Link Time Optimization." OFF)

set(LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)
set(RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)
set(PGO_DIR "${PROJECT_BINARY_DIR}/profiling"
  CACHE STRING
  "Directory where Profile Guided Optimization data is placed.")

if (CMAKE_VERSION VERSION_GREATER_EQUAL 3.9)
  cmake_policy(SET CMP0069 NEW)
  include(CheckIPOSupported)
  check_ipo_supported(RESULT IPO_SUPPORT)
  if (IPO_SUPPORT AND LTO)
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
  endif()
endif()

file(WRITE
  ${PROJECT_BINARY_DIR}/compiler_version.txt
  "${CMAKE_CXX_COMPILER_ID} - ${CMAKE_CXX_COMPILER_VERSION}")

enable_testing()

add_definitions(-w) # Suppresses all warnings. Remove later.

set(Z_SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/inflate.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/compress.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/deflate.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/gzread.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/crc32.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/infback.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/example.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/zutil.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/gzlib.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/minigzip.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/inftrees.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/uncompr.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/gzwrite.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/trees.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/gzclose.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/inffast.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/zlib-1.2.5/adler32.c
)
add_library(z STATIC ${Z_SRCS})
target_compile_definitions(z PUBLIC HAVE_UNISTD_H=1) # TODO: Check if actually present.

set(ILMBASE_SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/IlmThreadPool.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/IlmThreadSemaphorePosixCompat.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/toFloat.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/IlmThread.cpp
  # ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/IlmThreadMutexWin32.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/IlmThreadSemaphore.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/IlmThreadMutexPosix.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/IexThrowErrnoExc.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/ImathFun.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/eLut.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/half.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/ImathRandom.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/ImathBox.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/ImathVec.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/IlmThreadMutex.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/IexBaseExc.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/ImathShear.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/ImathMatrixAlgo.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/IlmThreadSemaphorePosix.cpp
  # ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/IlmThreadSemaphoreWin32.cpp
)
add_library(ilmbase STATIC ${ILMBASE_SRCS})

set(EXR_SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/IlmThreadPosix.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/ImathColorAlgo.cpp
  # ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2/IlmThreadWin32.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfInputFile.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfFloatAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfPreviewImageAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfBoxAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfPxr24Compressor.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfDoubleAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfFramesPerSecond.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfLineOrderAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfConvert.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfB44Compressor.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfRleCompressor.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfCompressionAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfKeyCodeAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfCRgbaFile.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfRgbaFile.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfCompressor.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfKeyCode.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfVecAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfVersion.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfIO.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfRationalAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfTimeCodeAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfEnvmap.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfTiledOutputFile.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/b44ExpLogTable.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfChannelListAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfTimeCode.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfStdIO.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfMatrixAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfAcesFile.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfZipCompressor.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfTiledInputFile.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfChannelList.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfThreading.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfTiledRgbaFile.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfStandardAttributes.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfRgbaYca.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfChromaticitiesAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfTestFile.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfScanLineInputFile.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfTiledMisc.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfStringVectorAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfStringAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfOutputFile.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfMisc.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfChromaticities.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfOpaqueAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfMultiView.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfRational.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfWav.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfEnvmapAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfLut.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfPizCompressor.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfHuf.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfIntAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfHeader.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfTileOffsets.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfTileDescriptionAttribute.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfFrameBuffer.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0/ImfPreviewImage.cpp
)
add_library(exr STATIC ${EXR_SRCS})
target_link_libraries(exr PUBLIC ilmbase z)
target_include_directories(exr SYSTEM PUBLIC $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2>)
target_include_directories(exr SYSTEM PUBLIC $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0>)


set(TIFF_SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_ojpeg.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_dirwrite.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_tile.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_jbig.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_packbits.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/mkg3states.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_warning.c
  # ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_apple.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_thunder.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_swab.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_compress.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_codec.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_luv.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_open.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_dumpmode.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_strip.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_print.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_flush.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_dirinfo.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_fax3.c
  # ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_win3.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_pixarlog.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_getimage.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_dir.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_write.c
  # ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_win32.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_fax3sm.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_predict.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_aux.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_close.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_extension.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_color.c
  # ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_acorn.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_jpeg.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_dirread.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_version.c
  # ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_atari.c
  # ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_unix.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_zip.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_read.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_error.c
  # ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_msdos.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_lzw.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/3rdparty/tiff-3.9.4/tif_next.c
)
add_library(tiff STATIC ${TIFF_SRCS})


set(PBRT_SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/src/filters/box.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/filters/gaussian.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/filters/mitchell.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/filters/triangle.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/filters/sinc.cpp

  ${CMAKE_CURRENT_SOURCE_DIR}/src/renderers/aggregatetest.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/renderers/createprobes.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/renderers/samplerrenderer.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/renderers/surfacepoints.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/renderers/metropolis.cpp

  ${CMAKE_CURRENT_SOURCE_DIR}/src/textures/marble.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/textures/imagemap.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/textures/wrinkled.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/textures/fbm.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/textures/checkerboard.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/textures/bilerp.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/textures/windy.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/textures/dots.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/textures/uv.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/textures/mix.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/textures/scale.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/textures/constant.cpp

  ${CMAKE_CURRENT_SOURCE_DIR}/src/materials/glass.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/materials/subsurface.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/materials/mirror.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/materials/matte.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/materials/uber.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/materials/translucent.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/materials/measured.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/materials/substrate.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/materials/shinymetal.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/materials/mixmat.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/materials/kdsubsurface.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/materials/plastic.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/materials/metal.cpp

  ${CMAKE_CURRENT_SOURCE_DIR}/src/film/image.cpp

  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/memory.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/diffgeom.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/api.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/intersection.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/floatfile.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/geometry.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/scene.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/parallel.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/progressreporter.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/camera.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/error.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/rng.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/spectrum.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/sh.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/timer.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/light.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/film.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/texture.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/pbrtparse.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/sampler.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/probes.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/paramset.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/filter.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/montecarlo.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/imageio.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/transform.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/shrots.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/shape.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/integrator.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/quaternion.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/reflection.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/primitive.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/stdafx.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/fileutil.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/material.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/parser.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/pbrtlex.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/renderer.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/volume.cpp

  ${CMAKE_CURRENT_SOURCE_DIR}/src/accelerators/bvh.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/accelerators/kdtreeaccel.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/accelerators/grid.cpp

  ${CMAKE_CURRENT_SOURCE_DIR}/src/shapes/disk.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/shapes/trianglemesh.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/shapes/heightfield.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/shapes/cylinder.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/shapes/loopsubdiv.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/shapes/hyperboloid.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/shapes/paraboloid.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/shapes/cone.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/shapes/sphere.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/shapes/nurbs.cpp

  ${CMAKE_CURRENT_SOURCE_DIR}/src/cameras/environment.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/cameras/orthographic.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/cameras/perspective.cpp

  ${CMAKE_CURRENT_SOURCE_DIR}/src/integrators/igi.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/integrators/single.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/integrators/emission.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/integrators/photonmap.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/integrators/directlighting.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/integrators/irradiancecache.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/integrators/useprobes.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/integrators/whitted.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/integrators/ambientocclusion.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/integrators/dipolesubsurface.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/integrators/path.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/integrators/diffuseprt.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/integrators/glossyprt.cpp

  ${CMAKE_CURRENT_SOURCE_DIR}/src/samplers/halton.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/samplers/bestcandidate.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/samplers/lowdiscrepancy.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/samplers/random.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/samplers/stratified.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/samplers/adaptive.cpp

  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/ply.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/ply2pbrt.c
  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/targa.c

  ${CMAKE_CURRENT_SOURCE_DIR}/src/volumes/homogeneous.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/volumes/volumegrid.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/volumes/exponential.cpp

  ${CMAKE_CURRENT_SOURCE_DIR}/src/lights/goniometric.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/lights/projection.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/lights/point.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/lights/distant.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/lights/spot.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/lights/infinite.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/lights/diffuse.cpp
)

add_library(pbrt STATIC ${PBRT_SRCS})
target_include_directories(pbrt
  SYSTEM PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src/3rdparty/ilmbase-1.0.2>
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src/3rdparty/openexr-1.7.0>
  PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src>
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src/core>
  $<INSTALL_INTERFACE:include>
  $<INSTALL_INTERFACE:include/pbrt>)
target_link_libraries(pbrt exr)
target_compile_definitions(pbrt PUBLIC PBRT_PROBES_NONE=1 HAVE_EXR=1 PBRT_HAS_OPENEXR=1)

# add_executable(exrdiff   ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/exrdiff.cpp)
# add_executable(exrtotiff ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/exrtotiff.cpp)
# add_executable(tifftoexr ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/tifftoexr.cpp)
# add_executable(bsdftest  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/bsdftest.cpp)
# add_executable(samplepat ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/samplepat.cpp)
# add_executable(obj2pbrt  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/obj2pbrt.cpp)
# add_executable(exravg    ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/exravg.cpp)

add_executable(pbrt_exe ${CMAKE_CURRENT_SOURCE_DIR}/src/main/pbrt.cpp)
set_target_properties(pbrt_exe PROPERTIES OUTPUT_NAME pbrt)
target_link_libraries(pbrt_exe pbrt)

install(TARGETS pbrt_exe DESTINATION bin)
install(TARGETS pbrt DESTINATION lib)
