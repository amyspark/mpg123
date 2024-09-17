# Component selection:
# If HAVE_MODULES, the library can be built fully independently.
# If not, the library depends on the one builtin module from
# the subdirectory.

# For dynamic modules.
if conf.get('HAVE_MODULES', false)
    subdirectory('modules')
endif

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

libout123_deps = [libcompat, librt_dep, m_dep]

if conf.get('HAVE_MODULES', false)
    libout123_deps += [libcompat_dl]
else
    libout123_deps += [libdefaultmodule]
endif

libout123_sources += files('module.h')

if conf.get('HAVE_MODULES', false)
    libout123_sources += files('module.c')
else
    libout123_sources += files('legacy_module.c')
endif

if conf.get('NEED_MAINLIB', false)
    if build_libout123
        # Just for libdefaultmodule.
        if not conf.get('HAVE_MODULES', false)
            subdirectory('modules')
        endif

        # build libout123
        # FIXME: module definition file, version info
        libout123 = library(
            'libout123',
            libout123_sources,
            dependencies: libout123_deps, # FIXME: libs
            install: true
        )
    endif
endif
