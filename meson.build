project('libpng', 'c', version: '1.6.40', license: 'libpng', meson_version: '>=0.50.0')

pkg = import('pkgconfig')

png_versions = meson.project_version().split('.')
png_major_version = png_versions[0]
png_minor_version = png_versions[1]
png_micro_version = png_versions[2]
png_libname = 'png@0@@1@'.format(png_major_version, png_minor_version)
png_libversion = '@0@@1@.@2@.0'.format(png_major_version, png_minor_version, png_micro_version)

cc = meson.get_compiler('c')

include = include_directories('.')

c_args = []

if host_machine.system() == 'windows'
  add_project_arguments(
    '-D_CRT_NONSTDC_NO_DEPRECATE',
    '-D_CRT_SECURE_NO_DEPRECATE',
    language : 'c'
  )
  if get_option('default_library') != 'static'
    c_args += '-DPNG_BUILD_DLL'
  endif
endif

zlib = dependency('zlib')
libm = cc.find_library('m', required: false)

libpng_deps = [ zlib, libm ]

png_src = files(
  'png.c',
  'pngerror.c',
  'pngget.c',
  'pngmem.c',
  'pngpread.c',
  'pngread.c',
  'pngrio.c',
  'pngrtran.c',
  'pngrutil.c',
  'pngset.c',
  'pngtrans.c',
  'pngwio.c',
  'pngwrite.c',
  'pngwtran.c',
  'pngwutil.c',
)


if host_machine.cpu_family() == 'aarch64' or cc.get_define('__ARM_NEON').strip() != ''
  png_src += files(
    'arm/arm_init.c',
    'arm/filter_neon_intrinsics.c',
    'arm/palette_neon_intrinsics.c',
  )
  if cc.get_id() == 'msvc'
    cl = find_program('cl')
    msvc_preprocessed_asm_files = custom_target(
      'MSVC assembly preprocessing',
      output: '@BASENAME@.i',
      input: 'arm/filter_neon.S',
      command: [cl, '/Fi@OUTPUT@', '/P', '@INPUT@', '-I@OUTDIR@', c_args],
    )
    msvc_armasm = find_program('armasm64')
    png_src += custom_target(
      'MSVC assembly compilation',
      output: '@BASENAME@.obj',
      input: msvc_preprocessed_asm_files,
      command: [msvc_armasm, '@INPUT@', '/Fo@OUTPUT@', '-I@OUTDIR@'],
    )
  else
    png_src += files('arm/filter_neon.S')
  endif

  c_args += '-DPNG_ARM_NEON_OPT=2'
elif host_machine.cpu_family() in ['x86', 'x86_64']
  png_src += files(
    'intel/filter_sse2_intrinsics.c',
    'intel/intel_init.c',
  )
  c_args += '-DPNG_INTEL_SSE_OPT=1'
endif

libpng = library(
  png_libname,
  png_src,
  version: png_libversion,
  dependencies: libpng_deps,
  c_args: c_args,
  install: true,
)

pngincsubdir = 'lib' + png_libname
pngincludedir = get_option('includedir') / pngincsubdir

pkglibconf = configure_file(
  input: 'scripts/pnglibconf.h.prebuilt',
  output: 'pnglibconf.h',
  copy: true,
)

install_headers(
  'png.h',
  'pngconf.h',
  pkglibconf,
  subdir: pngincsubdir,
)

libpng_dep = declare_dependency(
  include_directories: include,
  link_with: libpng,
  dependencies: libpng_deps,
)

libpng_pkg_requires = [ ]
if get_option('default_library') == 'static'
  libpng_pkg_requires += zlib
endif

pkg.generate(libpng,
    filebase : 'libpng',
    name : 'libpng',
    subdirs : 'lib' + png_libname,
    requires : libpng_pkg_requires,
    libraries : libpng,
    libraries_private : libpng_deps,
    version : meson.project_version(),
    description : 'Loads and saves PNG files',
    url : 'http://www.libpng.org/pub/png/libpng.html',
)

pkg.generate(libpng,
    filebase : 'libpng16',
    name : 'libpng',
    subdirs : 'lib' + png_libname,
    requires : libpng_pkg_requires,
    libraries : libpng,
    libraries_private : libpng_deps,
    version : meson.project_version(),
    description : 'Loads and saves PNG files',
    url : 'http://www.libpng.org/pub/png/libpng.html',
)

if get_option('tests')
  png_test = executable('pngtest', 'pngtest.c', dependencies: libpng_dep)
  test('pngtest', png_test, args: files('pngtest.png'))
endif
