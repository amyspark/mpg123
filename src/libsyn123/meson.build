# This builds libsyn123, a basic library for some sound synthesis
# and format conversion for the mpg123 project.

if build_libsyn123
    libsyn123 = library(
        'syn123',
        files(
            'syn123_int.h',
            'g711_impl.h',
            'pinknoise.c',
            'geiger.c',
            'libsyn123.c',
            'volume.c',
            'resample.c',
            'filter.c',
            'sampleconv.c',
        ),
        include_directories: includes,
        link_with: [libcompat_str],
        dependencies: [m_dep],
        gnu_symbol_visibility: 'hidden',
        version: meson.project_version(),
        install: true
    )

    libsyn123_dep = declare_dependency(
        link_with: libsyn123,
        dependencies: [m_dep],
        include_directories: includes,
    )
else
    libsyn123_dep = disabler()
endif
