headers = files()

if build_libmpg123
    headers += files(
        'mpg123.h',
    )
endif

if build_libsyn123
    headers += files(
        'syn123.h',
    )
endif

if build_libout123
    headers += files(
        'out123.h',
    )
endif

if conf.get('NEED_LIB', false) and conf.get('NEED_MAINLIB', false)
    # All main libraries need fmt123 header installed.
    headers += files(
        'fmt123.h',
    )
endif

install_headers(headers)

includes = [include_directories('.')]
