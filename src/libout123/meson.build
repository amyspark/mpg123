# Component selection:
# If HAVE_MODULES, the library can be built fully independently.
# If not, the library depends on the one builtin module from
# the subdirectory.

subdirectory('modules')

libout123_sources = files(
  'libout123.c',
  'stringlists.h',
  'stringlists.c',
  'out123_int.h',
  'wav.c',
  'wav.h',
  'hextxt.c',
  'hextxt.h',
  'wavhead.h',
)

if conf.get('BUILD_BUFFER', false)
    libout123_sources += files(
        'buffer.c',
        'buffer.h',
        'xfermem.c',
        'xfermem.h',
    )
endif

libmodule_sources = files('module.h')
if conf.get('HAVE_MODULES', false)
    libmodule_sources += files('module.c')
else
    libmodule_sources += files('legacy_module.c')
endif

libmodule = static_library(
    'module',
    libmodule_sources,
    include_directories: includes,
    install: false
)

libout123_deps = [libmodule, libcompat, librt_dep, m_dep]

if conf.get('HAVE_MODULES', false)
    libout123_deps += [libcompat_dl]
else
    libout123_deps += [libdefaultmodule]
endif

if conf.get('NEED_MAINLIB', false) and build_libout123
    # FIXME: module definition file, version info
    libout123 = library(
        'out123',
        libout123_sources,
        dependencies: libout123_deps, # FIXME: libs
        install: true
    )
endif
