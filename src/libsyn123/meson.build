# This builds libsyn123, a basic library for some sound synthesis
# and format conversion for the mpg123 project.

if build_libsyn123
    static_lib = static_library('syn123-static',
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
        install: false,
    )

    vs_module_def = custom_target('syn123-def',
        output: 'syn123.def',
        command: [makedef_args, '--regex', '^syn123_', makedef_os_args, '@INPUT0@'],
        input: [
            static_lib
        ],
        capture: true
    )

    libsyn123 = library(
        'syn123',
        link_whole: static_lib,
        link_args: vflag.format(vs_module_def.full_path()),
        vs_module_defs: vs_module_def,
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
