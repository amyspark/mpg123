# Do not include uneeded headers from mpg123app.h .
libout123_mod_cppflags = ['-DBUILDING_OUTPUT_MODULES=1']

# __modules = [
#     ['output_dummy', 'HAVE_DUMMY', files('dummy.c'), []],
#     ['output_tinyalsa', 'HAVE_TINYALSA', files('tinyalsa.c'), tinyalsa_dep],
#     ['output_alsa', 'HAVE_ALSA', files('alsa.c'), alsa_dep],
#     ['output_coreaudio', 'HAVE_COREAUDIO', files('coreaudio.c'), coreaudio_dep],
#     ['output_jack', 'HAVE_JACK', files('jack.c'), jack_dep],
#     ['output_portaudio', 'HAVE_PORTAUDIO', files('portaudio.c'), portaudio_dep],
#     ['output_pulse', 'HAVE_PULSE', files('pulse.c'), pulse_dep],
#     ['output_sdl', 'HAVE_SDL', files('sdl.c'), sdl_dep],
#     ['output_sndio', 'HAVE_SNDIO', files('sndio.c'), sndio_dep],
#     ['output_win32', 'HAVE_WIN32', files('win32.c'), win32_dep],
#     ['output_win32_wasapi', 'HAVE_WIN32_WASAPI', files('win32_wasapi.c'), win32_wasapi_dep],
#     ['output_openal', 'HAVE_OPENAL', files('openal.c'), openal_dep],
# ]

foreach mod : output_modules
    mod_f = files(
        '@0@.c'.format(mod)
    )
    mod_dep = get_variable('@0@_dep'.format(mod), [])
    if conf.get('HAVE_MODULES', false)
        have = 'HAVE_@0@'.format(mod.to_upper())
        if conf.get('BUILD_LIBOUT123_MODULES', false) and conf.get(have, false)
            output_module = shared_module(
                'output_@0@'.format(mod),
                mod_f,
                c_args: libout123_mod_cppflags,
                include_directories: includes,
                link_with: libcompat_str,
                dependencies: mod_dep,
                name_prefix: '',
                gnu_symbol_visibility: 'default',
                install: true,
                install_dir: conf.get('PKGLIBDIR')
            )
        endif
    else
        have = 'BUILD_@0@'.format(mod.to_upper())
        if conf.get(have, false)
            libdefaultmodule_sources = mod_f
            libdefaultmodule_deps = mod_dep
        endif
    endif
endforeach

if conf.get('HAVE_MODULES', false)
    libdefaultmodule = []
else
    libdefaultmodule = static_library(
        'defaultmodule',
        libdefaultmodule_sources,
        include_directories: includes,
        dependencies: libdefaultmodule_deps,
        install: false
    )
endif
