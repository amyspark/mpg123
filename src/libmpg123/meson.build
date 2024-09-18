if conf.get('HAVE_GETCPUFLAGS', false)
    testcpu = executable(
        'testcpu',
        files('testcpu.c', 'getcpuflags.S'),
        link_with: [libcompat],
        install: false,
    )
endif

# Uses VLA with specified dimensions in Y
build_options = []
if cc.compiles('''#include <stdint.h>
    #include <stdio.h>
    extern void print_array( int statick, int fixed, double fixed_scale
    ,	const char *indent, const char *name
    ,	size_t count, double tab[], int calc );
    void print_array2d( int fixed, double fixed_scale
    ,	const char *name, size_t x, size_t y
    , double tab[][y], int calc )
    {
        if(calc)
        {
            if(!fixed)
                printf("static ALIGNED(16) real %s[%zu][%zu];\n", name, x, y);
            return;
        }
        printf( "static const%s real %s[%zu][%zu] = \n{\n", fixed ? "" : " ALIGNED(16)"
        ,	name, x, y );
        for(size_t i=0; i<x; ++i)
        {
            if(i)
                printf(",");
            print_array(1, fixed, fixed_scale, "\t", NULL, y, tab[i], calc);
        }
        printf("};\n");
    }
    int main()
    {
        return 0;
    }''', name: 'tolerates VLA')
    calctables = executable(
        'calctables',
        files('calctables.c', 'init_costabs.h', 'init_layer12.h', 'init_layer3.h'),
        include_directories: includes,
        dependencies: m_dep,
        install: false
    )
endif

if build_libmpg123
    libmpg123_sources = files(
        'mpeghead.h',
        'parse.c',
        'parse.h',
        'frame.c',
        'format.c',
        'frame.h',
        'reader.h',
        'decode.h',
        'dct64.c',
        'synth.h',
        'synth_mono.h',
        'synth_ntom.h',
        'synth_8bit.h',
        'synths.h',
        'huffman.h',
        'newhuffman.h',
        'icy.h',
        'icy2utf8.h',
        'id3.h',
        'id3.c',
        'getbits.h',
        'optimize.h',
        'optimize.c',
        'readers.c',
        'lfs_wrap.h',
        'costabs.h',
        'tabinit.c',
        'libmpg123.c',
        'gapless.h',
        'mpg123lib_intern.h',
        'mangle.h',
        'getcpuflags.h',
        'index.h',
        'index.c',
    )

    if conf.get('HAVE_LAYER1', false)
        libmpg123_sources += files('layer1.c')
    endif

    if conf.get('HAVE_LAYER2', false)
        libmpg123_sources += files('layer2.c')
    endif

    if conf.get('HAVE_LAYER3', false)
        libmpg123_sources += files('layer3.c')
    endif

    if conf.get('HAVE_EQUALIZER', false)
        libmpg123_sources += files('equalizer.c')
    endif

    if conf.get('HAVE_DITHER', false)
        libmpg123_sources += files('dither.c')
    endif

    if conf.get('HAVE_SYNTH8', false)
        libmpg123_sources += files('synth_8bit.c')
    endif

    if conf.get('HAVE_SYNTH16', false)
        libmpg123_sources += files('synth.c')
    endif

    if conf.get('HAVE_SYNTH32', false)
        libmpg123_sources += files('synth_s32.c')
    endif

    if conf.get('HAVE_SYNTHREAL', false)
        libmpg123_sources += files('synth_real.c')
    endif

    if conf.get('HAVE_LFS_WRAP', false)
        libmpg123_sources += files('lfs_wrap.c')
    endif

    if conf.get('HAVE_ICY', false)
        libmpg123_sources += files('icy.c', 'icy2utf8.c')
    endif

    if conf.get('HAVE_FEATURE', false)
        libmpg123_sources += files('feature.c')
    endif

    if conf.get('HAVE_NTOM', false)
        libmpg123_sources += files('ntom.c')
    endif

    if conf.get('HAVE_STRING', false)
        libmpg123_sources += files('stringbuf.c')
    endif

    if conf.get('HAVE_GETCPUFLAGS', false)
        libmpg123_sources += files('getcpuflags.S')
    endif

    if conf.get('HAVE_GETCPUFLAGS_X86_64', false)
        libmpg123_sources += files('getcpuflags_x86_64.S')
    endif

    if conf.get('HAVE_GETCPUFLAGS_ARM', false)
        libmpg123_sources += files('getcpuflags_arm.c', 'check_neon.S')
    endif

    if conf.get('HAVE_ALTIVEC', false)
        libmpg123_sources += files('synth_altivec.c', 'dct64_altivec.c')
    endif

    if conf.get('HAVE_I386', false)
        libmpg123_sources += files('dct64_i386.c')
    endif

    if conf.get('HAVE_I486', false)
        libmpg123_sources += files('synth_i486.c', 'dct64_i486.c')
    endif

    if conf.get('HAVE_I586', false)
        libmpg123_sources += files('synth_i586.S')
    endif

    if conf.get('HAVE_I586_DITHER', false)
        libmpg123_sources += files('synth_i586_dither.S')
    endif

    if conf.get('HAVE_3DNOW', false)
        libmpg123_sources += files('synth_3dnow.S', 'dct64_3dnow.S')
    endif

    if conf.get('HAVE_3DNOW', false)
    if conf.get('HAVE_EQUALIZER', false)
        libmpg123_sources += files('equalizer_3dnow.S')
    endif
    endif

    if conf.get('HAVE_3DNOW_VINTAGE', false)
    if conf.get('HAVE_LAYER3', false)
        libmpg123_sources += files('dct36_3dnow.S')
    endif
    endif

    if conf.get('HAVE_3DNOWEXT', false)
        libmpg123_sources += files('dct64_3dnowext.S', 'synth_3dnowext.S')
    endif

    if conf.get('HAVE_3DNOWEXT_VINTAGE', false)
    if conf.get('HAVE_LAYER3', false)
        libmpg123_sources += files('dct36_3dnowext.S')
    endif
    endif

    if conf.get('HAVE_MMXTAB', false)
        libmpg123_sources += files('tabinit_mmx.S')
    endif

    if conf.get('HAVE_MMX', false)
        libmpg123_sources += files('dct64_mmx.S', 'synth_mmx.S')
    endif

    if conf.get('HAVE_SSE_VINTAGE', false)
    if conf.get('HAVE_FLOATDCT', false)
        libmpg123_sources += files('dct64_sse_float.S')
    endif
    endif

    if conf.get('HAVE_SSE_VINTAGE', false)
    if conf.get('HAVE_SYNTHREAL', false)
        libmpg123_sources += files('synth_sse_float.S synth_stereo_sse_float.S')
    endif
    endif

    if conf.get('HAVE_SSE_VINTAGE', false)
    if conf.get('HAVE_SYNTH32', false)
        libmpg123_sources += files('synth_sse_s32.S', 'synth_stereo_sse_s32.S')
    endif
    endif

    if conf.get('HAVE_SSE', false)
    if conf.get('HAVE_LAYER3', false)
        libmpg123_sources += files('dct36_sse.S')
    endif
    endif

    if conf.get('HAVE_SSE_VINTAGE', false)
    if conf.get('HAVE_ACCURATE', false)
        libmpg123_sources += files('synth_sse_accurate.S', 'synth_stereo_sse_accurate.S')
    endif
    endif

    if conf.get('HAVE_SSE_VINTAGE', false)
    if not conf.get('HAVE_ACCURATE', false)
        libmpg123_sources += files('dct64_sse.S', 'synth_sse.S')
    endif
    endif

    if conf.get('HAVE_X86_64', false)
    if conf.get('HAVE_LAYER3', false)
        libmpg123_sources += files('dct36_x86_64.S')
    endif
    endif

    if conf.get('HAVE_X86_64', false)
    if conf.get('HAVE_SYNTHREAL', false)
        libmpg123_sources += files('synth_stereo_x86_64_float.S')
    endif
    endif

    if conf.get('HAVE_X86_64', false)
    if conf.get('HAVE_SYNTH32', false)
        libmpg123_sources += files('synth_stereo_x86_64_s32.S')
    endif
    endif

    if conf.get('HAVE_X86_64', false)
    if conf.get('HAVE_ACCURATE', false)
        libmpg123_sources += files('synth_stereo_x86_64_accurate.S')
    endif
    endif

    if conf.get('HAVE_X86_64', false)
    if not conf.get('HAVE_ACCURATE', false)
        libmpg123_sources += files('dct64_x86_64.S', 'synth_stereo_x86_64.S')
    endif
    endif

    if conf.get('HAVE_X86_64', false)
    if conf.get('HAVE_FLOATDCT', false)
        libmpg123_sources += files('dct64_x86_64_float.S')
    endif
    endif

    if conf.get('HAVE_X86_64_MONO', false)
    if conf.get('HAVE_SYNTHREAL', false)
        libmpg123_sources += files('synth_x86_64_float.S')
    endif
    endif

    if conf.get('HAVE_X86_64_MONO', false)
    if conf.get('HAVE_SYNTH32', false)
        libmpg123_sources += files('synth_x86_64_s32.S')
    endif
    endif

    if conf.get('HAVE_X86_64_MONO', false)
    if conf.get('HAVE_ACCURATE', false)
        libmpg123_sources += files('synth_x86_64_accurate.S')
    endif
    endif

    if conf.get('HAVE_X86_64_MONO', false)
    if not conf.get('HAVE_ACCURATE', false)
        libmpg123_sources += files('synth_x86_64.S')
    endif
    endif

    if conf.get('HAVE_AVX', false)
    if conf.get('HAVE_LAYER3', false)
        libmpg123_sources += files('dct36_avx.S')
    endif
    endif

    if conf.get('HAVE_AVX', false)
    if conf.get('HAVE_FLOATDCT', false)
        libmpg123_sources += files('dct64_avx_float.S')
    endif
    endif

    if conf.get('HAVE_AVX', false)
    if conf.get('HAVE_SYNTHREAL', false)
        libmpg123_sources += files('synth_stereo_avx_float.S')
    endif
    endif

    if conf.get('HAVE_AVX', false)
    if conf.get('HAVE_SYNTH32', false)
        libmpg123_sources += files('synth_stereo_avx_s32.S')
    endif
    endif

    if conf.get('HAVE_AVX', false)
    if conf.get('HAVE_ACCURATE', false)
        libmpg123_sources += files('synth_stereo_avx_accurate.S')
    endif
    endif

    if conf.get('HAVE_AVX', false)
    if not conf.get('HAVE_ACCURATE', false)
        libmpg123_sources += files('dct64_avx.S synth_stereo_avx.S')
    endif
    endif

    if conf.get('HAVE_ARM', false)
    if conf.get('HAVE_ACCURATE', false)
        libmpg123_sources += files('synth_arm_accurate.S')
    endif
    endif

    if conf.get('HAVE_ARM', false)
    if not conf.get('HAVE_ACCURATE', false)
        libmpg123_sources += files('synth_arm.S')
    endif
    endif

    if conf.get('HAVE_NEON', false)
    if conf.get('HAVE_LAYER3', false)
        libmpg123_sources += files('dct36_neon.S')
    endif
    endif

    if conf.get('HAVE_NEON', false)
    if conf.get('HAVE_FLOATDCT', false)
        libmpg123_sources += files('dct64_neon_float.S')
    endif
    endif

    if conf.get('HAVE_NEON', false)
    if conf.get('HAVE_SYNTHREAL', false)
        libmpg123_sources += files('synth_neon_float.S synth_stereo_neon_float.S')
    endif
    endif

    if conf.get('HAVE_NEON', false)
    if conf.get('HAVE_SYNTH32', false)
        libmpg123_sources += files('synth_neon_s32.S', 'synth_stereo_neon_s32.S')
    endif
    endif

    if conf.get('HAVE_NEON', false)
    if conf.get('HAVE_ACCURATE', false)
        libmpg123_sources += files('synth_neon_accurate.S', 'synth_stereo_neon_accurate.S')
    endif
    endif

    if conf.get('HAVE_NEON', false)
    if not conf.get('HAVE_ACCURATE', false)
        libmpg123_sources += files('dct64_neon.S', 'synth_neon.S', 'synth_stereo_neon.S')
    endif
    endif

    if conf.get('HAVE_NEON64', false)
    if conf.get('HAVE_LAYER3', false)
        libmpg123_sources += files('dct36_neon64.S')
    endif
    endif

    if conf.get('HAVE_NEON64', false)
    if conf.get('HAVE_FLOATDCT', false)
        libmpg123_sources += files('dct64_neon64_float.S')
    endif
    endif

    if conf.get('HAVE_NEON64', false)
    if conf.get('HAVE_SYNTHREAL', false)
        libmpg123_sources += files('synth_neon64_float.S', 'synth_stereo_neon64_float.S')
    endif
    endif

    if conf.get('HAVE_NEON64', false)
    if conf.get('HAVE_SYNTH32', false)
        libmpg123_sources += files('synth_neon64_s32.S', 'synth_stereo_neon64_s32.S')
    endif
    endif

    if conf.get('HAVE_NEON64', false)
    if conf.get('HAVE_ACCURATE', false)
        libmpg123_sources += files('synth_neon64_accurate.S', 'synth_stereo_neon64_accurate.S')
    endif
    endif

    if conf.get('HAVE_NEON64', false)
    if not conf.get('HAVE_ACCURATE', false)
        libmpg123_sources += files('dct64_neon64.S', 'synth_neon64.S', 'synth_stereo_neon64.S')
    endif
    endif

    libmpg123_sources += files(
        'l2tables.h',
        'l12tabs.h',
        'l3bandgain.h',
        'l3tabs.h',
        'dither.h',
        'dither_impl.h',
        'synth_sse3d.h',
    )

    if avx_support
        libmpg123_sources += files(
            'dct36_avx.S',
            'dct64_avx.S',
            'dct64_avx_float.S',
            'synth_stereo_avx.S',
            'synth_stereo_avx_float.S',
            'synth_stereo_avx_s32.S',
            'synth_stereo_avx_accurate.S',
            'mangle.h',
        ) + config_h
    endif

    static_lib = static_library('mpg123-static',
        libmpg123_sources,
        include_directories: includes,
        dependencies: [m_dep, network_deps, compat_libs],
        gnu_symbol_visibility: 'hidden',
        link_with: [libcompat],
        install: false,
    )

    vs_module_def = custom_target('mpg123-def',
        output: 'mpg123.def',
        command: [makedef_args, '--regex', '^mpg123_', makedef_os_args, '@INPUT0@'],
        input: [
            static_lib
        ],
        capture: true
    )

    libmpg123 = library('mpg123',
        link_whole: static_lib,
        include_directories: includes,
        vs_module_defs: vs_module_def,
        gnu_symbol_visibility: 'hidden',
        version: meson.project_version(),
        install: true,
    )

    libmpg123_dep = declare_dependency(
        link_with: libmpg123,
        dependencies: [m_dep, network_deps, compat_libs],
        include_directories: includes,
    )
endif
