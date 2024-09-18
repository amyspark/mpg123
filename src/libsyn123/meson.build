# This bulds libsyn123, a basic library for some sound synthesis
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
        dependencies: [m_dep],
        include_directories: includes,
        install: true
    )
endif
