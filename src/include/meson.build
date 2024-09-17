includes = files()

if build_libmpg123
    includes += files(
        'mpg123.h',
    )
endif

if build_libsyn123
    includes += files(
        'syn123.h',
    )
endif

if build_libout123
    includes += files(
        'out123.h',
    )
endif

if conf.get('NEED_LIB', false) and conf.get('NEED_MAINLIB', false)
    # All main libraries need fmt123 header installed.
    includes += files(
        'fmt123.h',
    )
endif

includes = include_directories('.')
