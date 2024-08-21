# Do not include unneeded headers from mpg123app.h .
libout123_mod_cppflags = ['-DBUILDING_OUTPUT_MODULES=1']

# On macOS, the default suffix for loadable modules is .so, but meson uses
# .dylib by default. Since the code expects .so and autotools was using .so,
# make meson do the same.
# See https://github.com/p11-glue/p11-kit/commit/8a4348e8468d08d992cb7d22a4d887d0191383f6
module_suffix = []
if os in ['darwin', 'ios']
  module_suffix = 'so'
endif

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
                name_suffix: module_suffix,
                gnu_symbol_visibility: 'default',
                install: build_libout123,
                install_dir: get_option('libdir') / meson.project_name()
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
