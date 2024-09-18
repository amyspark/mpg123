config_h = configure_file(
    output: 'config.h',
    configuration: conf
)

subdir('include')

includes += include_directories('.')

subdir('compat')
subdir('common')
if conf.get('NEED_LIB', false)
    subdir('libout123')
    if conf.get('NEED_MAINLIB', false)
        subdir('libmpg123')
        subdir('libsyn123')
    endif
endif

mpg123_sources = files(
    'audio.c',
    'audio.h',
    'common.c',
    'common.h',
    'sysutil.c',
    'sysutil.h',
    'control_generic.c',
    'equalizer.c',
    'getlopt.c',
    'getlopt.h',
    'httpget.c',
    'httpget.h',
    'resolver.c',
    'resolver.h',
    'genre.h',
    'genre.c',
    'mpg123.c',
    'mpg123app.h',
    'metaprint.c',
    'metaprint.h',
    'local.h',
    'local.c',
    'playlist.c',
    'playlist.h',
    'streamdump.h',
    'streamdump.c',
    'term.c',
    'term.h',
    'terms.h',
    'win32_support.h',
)

out123_sources = files(
    'sysutil.c',
    'sysutil.h',
    'common.h',
    'getlopt.c',
    'getlopt.h',
    'local.h',
    'local.c',
    'filters.h',
    'filters.c',
    'out123.c',
    'mpg123app.h',
    'win32_support.h',
)

mpg123_id3dump_sources = files(
  'mpg123-id3dump.c',
  'getlopt.c',
  'getlopt.h',
)

mpg123_strip_sources = files(
  'mpg123-strip.c',
  'getlopt.c',
  'getlopt.h',
)

mpg123_deps = []
mpg123_libs = [libmpg123, libout123, libsyn123]
if conf.get('TERM_POSIX', false)
mpg123_sources += files('term_posix.c')
endif

if conf.get('TERM_WIN32', false)
mpg123_sources += files('term_win32.c')
endif

if conf.get('TERM_NONE', false)
mpg123_sources += files('term_none.c')
endif

if conf.get('NET123', false)
mpg123_sources += files('net123.h')
endif

if conf.get('NET123_EXEC', false)
mpg123_sources += files('net123_exec.c')
endif

if conf.get('NET123_WINHTTP', false)
mpg123_sources += files('net123_winhttp.c')
mpg123_deps += [cc.find_library('winhttp', required: true)]
endif

if conf.get('NET123_WININET', false)
mpg123_sources += files('net123_wininet.c')
mpg123_deps += [cc.find_library('wininet', required: true)]
endif

if conf.get('WIN32_CODES', false)
mpg123_sources += files('win32_support.c')

if conf.get('NETWORK_WINSOCK', false)
mpg123_sources += files('win32_net.c')
mpg123_deps += [cc.find_library('ws2_32', required: true)]
endif

out123_sources += files('win32_support.c')

mpg123_id3dump_sources += files('win32_support.c')
endif

if build_programs
    out123 = executable(
        'out123',
        out123_sources,
        include_directories: includes,
        link_with: [libcompat, libout123, libsyn123],
        install: true,
    )
    if conf.get('HAVE_LFS_WRAP', false)
        mpg123 = executable(
            'mpg123',
            mpg123_sources,
            include_directories: includes,
            link_with: [libcompat, libmpg123],
            dependencies: mpg123_deps,
            install: true,
        )

        mpg123_id3dump = executable(
            'mpg123-id3dump',
            mpg123_id3dump_sources,
            include_directories: includes,
            link_with: [libcompat, libmpg123],
            install: true,
        )

        mpg123_strip = executable(
            'mpg123-strip',
            mpg123_strip_sources,
            include_directories: includes,
            link_with: [libcompat, libmpg123],
            install: true,
        )
    endif
endif
