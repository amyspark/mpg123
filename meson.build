project('mpg123', 'c',
  meson_version: '>= 1.2.0',
  license: 'LGPL2.1+',
  # C11 is needed for Metal support
  # Static libraries by default following upstream
  default_options: [ 'buildtype=debugoptimized', 'c_std=c11', 'default_library=shared'],
  version: '1.32.7',
)

libs = []

cc = meson.get_compiler('c')

os = host_machine.system()
meson_cpu = host_machine.cpu_family()

conf = configuration_data()

conf.set('HAVE_CLOCK_GETTIME', cc.has_function('clock_gettime'))

modules = get_option('modules').enable_if(
    get_option('default_library') == 'shared'
)

conf.set('DYNAMIC_BUILD', modules.allowed())

conf.set('HAVE_WINDOWS_H', os in ['windows', 'cygwin'] and cc.has_header('windows.h'))

uwp_build = cc.compiles('''
    #include <winapifamily.h>
    #if WINAPI_FAMILY_PARTITION (WINAPI_PARTITION_DESKTOP)
    # error Win32 Desktop build
    #endif
    int main() { return 0; }
''',
    name: 'this is a UWP build'
)
conf.set('WINDOWS_UWP', uwp_build)

android_build = os == 'android'

if os in ['windows', 'cygwin'] and modules
    modules.enable_auto_if(
        cc.has_function('GetThreadErrorMode', prefix: '#include <windows.h>')
    )
endif


libdl = []
if not modules
    message('Modules disabled, not checking for dynamic loading.')
elif os in ['windows', 'cygwin']
    modules.enable_auto_if(
        cc.compiles('''
            #include <windows.h>

            int main() {
            LoadLibraryW(0);
            GetProcAddress(0, 0);
            FreeLibrary(0);
            }
        ''',
            name: 'LoadLibrary should be used'
        )
    )
else
    libdl = cc.find_library('dl')
    dlopen = cc.has_function('dlopen', dependencies: libdl)
    dlsym = cc.has_function('dlsym', dependencies: libdl)
    dlclose = cc.has_function('dlclose', dependencies: libdl)

    conf.set('HAVE_DLOPEN', dlopen)
    conf.set('HAVE_DLSYM', dlsym)
    conf.set('HAVE_DLCLOSE', dlclose)
    conf.set('HAVE_DLFCN_H', cc.has_header('dlfcn.h', dependencies: libdl))

    modules.enable_auto_if(dlopen and dlclose and dlsym)

    if modules.allowed()
        message('We found a runtime loader: Modules enabled.')
    else
        warning('We did not find a runtime loader: Modules disabled.')
        modules.disable_auto_if(true)
    endif
    conf.set('USE_MODULES', modules.allowed())
    conf.set_quoted('LT_MODULE_EXT', os == 'windows' ? '.dll' : '.so')
endif

build_programs = get_option('programs')
build_libmpg123 = get_option('libmpg123')
build_libout123 = get_option('libout123')
build_libout123_modules = get_option('libout123-modules')
build_libsyn123 = get_option('libsyn123')

# Set default options (I won't make them configurable)
conf.set('GAPLESS', true)
fifo = get_option('fifo')
conf.set('HAVE_MKFIFO', fifo.allowed())
id3v2 = get_option('id3v2')
conf.set('HAVE_STRING', true)
conf.set('NO_ID3V2', not id3v2.allowed())
conf.set('HAVE_ICY', true)
ntom = get_option('ntom')
conf.set('HAVE_NTOM', ntom.allowed())
if not ntom.allowed()
    conf.set('NO_NTOM', true)
endif
downsample = get_option('downsample')
conf.set('NO_DOWNSAMPLE', not downsample.allowed())
feeder = get_option('feeder')
conf.set('NO_FEEDER', not feeder.allowed())
moreinfo = get_option('moreinfo')
conf.set('NO_MOREINFO', not moreinfo.allowed())
messages = get_option('messages')
if not messages.allowed()
    conf.set('NO_WARNING', true)
    conf.set('NO_ERRORMSG', true)
    conf.set('NO_ERETURN', true)
endif
runtimetables = get_option('runtime-tables')
conf.set('RUNTIME_TABLES', runtimetables.allowed())
new_huffman = get_option('new-huffman')
conf.set('USE_NEW_HUFFTABLE', new_huffman.allowed())
int_quality = get_option('int-quality')
conf.set('ACCURATE_ROUNDING', int_quality.allowed())
# FIXME: depths: int16, int8, int32, real
# FIXME: equalizer
# FIXME: I don't want to enable Yasm
# FIXME: I'm not letting IEEE 754 be disabled
conf.set('IEEE_FLOAT', true)
specialcases = get_option('cases')
conf.set('SYN123_NO_CASES', not specialcases.allowed())

cpu = get_option('cpu')
newoldwritesample = get_option('newoldwritesample')
if cpu == 'generic_fpu'
    if meson_cpu == 'aarch64'
        cpu = 'aarch64'
        newoldwritesample = newoldwritesample.enable_auto_if(
            os in ['linux', 'darwin', 'ios'] or os.contains('bsd')
        )
    elif meson_cpu == 'arm'
        cpu = 'arm_fpu'
        newoldwritesample = newoldwritesample.disable_auto_if(
            os in ['linux', 'darwin', 'ios']
        )
    elif meson_cpu == 'x86_64'
        cpu = 'x86-64'
    elif meson_cpu == 'x86'
        cpu = 'x86'
        newoldwritesample = newoldwritesample.disable_auto_if(
            os in ['linux', 'darwin', 'ios'] or os.contains('bsd')
        )
    elif meson_cpu == 'ppc64'
        cpu = 'altivec'
    else
        add_project_arguments('-DGENERIC', language: 'c')
    endif
    newoldwritesample = newoldwritesample.disable_auto_if(cpu == 'generic_fpu')
endif

buffer = get_option('buffer')
buffer = buffer.disable_auto_if(
    cpu == 'generic_fpu' or cc.has_define('__MINGW32__') or cc.has_define('__MINGW64__')
)

# FIXME: all my supported CPUs require float
# FIXME: int16 etc.
conf.set('HAVE_SYNTH16', true)
conf.set('HAVE_SYNTH32', true)
conf.set('HAVE_SYNTHREAL', true)

equalizer = get_option('equalizer')
conf.set('HAVE_EQUALIZER', equalizer.allowed())
conf.set('NO_EQUALIZER', not equalizer.allowed())

layer1 = get_option('layer1')
conf.set('HAVE_LAYER1', layer1.allowed())
layer2 = get_option('layer2')
conf.set('HAVE_LAYER2', layer2.allowed())
layer3 = get_option('layer3')
conf.set('HAVE_LAYER3', layer3.allowed())

seektable = get_option('seektable')

if cc.compiles('int main() { __asm__(\".balign 4\"); return 0; }', name: '.balign is present')
    conf.set('ASMALIGN_BALIGN', true)
elif cc.compiles('int main() { __asm__(\".align 3\"); return 0; }', name: 'Define if .align takes 3 for alignment of 2^3=8 bytes instead of 8.')
    conf.set('ASMALIGN_EXP', true)
else
    conf.set('ASMALIGN_BYTE', true)
endif

conf.set('CCALIGN', cc.compiles('int main() { __attribute__((aligned(16))) float var; return 0; }', name: 'align(16) is supported'))

avx_support = false
if cpu_type.contains('x86')
    x86asm = find_program('nasm', version: '>=2.12', required: false)
    if not x86asm.found()
        x86asm = find_program('yasm', required: false)
    endif

    if x86asm.found()
        add_languages('nasm')
        x86asm = meson.get_compiler('nasm')

        check = '.text\nvaddps %ymm0,%ymm0,%ymm0\n'
        f = configure_file(
            command: [python3, '-c', 'import sys; print(sys.argv[1])', '@0@'.format(check[0])],
            input: 'configure',
            output: '@0@.asm'.format(check[1]),
            capture: true,
        )
        avx_support = x86asm.compiles(
            f,
            args: ['-x', 'assembler'],
            name: 'supports GAS syntax and AVX instructions'
        )
    endif

    if cpu_type.contains('x86') and not avx_support
        error('Assembler doesn\'t understand AVX instructions.')
    endif
endif

############### Really basic headers, needed for other checks.

headers = [
    'unistd.h',
    'sched.h',
    'sys/ioctl.h',
    'sys/types.h',
    'sys/time.h',
    'sys/wait.h',
    'sys/resource.h',
    'sys/signal.h',
    'signal.h',
    'sys/select.h',
    'dirent.h',
    'sys/stat.h',
]

foreach h : headers
    id = 'HAVE_@0@'.format(h.to_upper().underscorify())
    conf.set(id, cc.has_header(h))
endforeach

############## LFS stuff

portable_api = get_option('portable')
conf.set('PORTABLE_API', portable_api.allowed())
conf.set('HAVE_LFS_WRAP', not portable_api.allowed())

# Check for shape-shifting off_t just for the mpg123 client application.
switched_off_size = cc.sizeof('off_t',
    prefix: '''
        #undef  _FILE_OFFSET_BITS
        #define _FILE_OFFSET_BITS 64
        #include <sys/types.h>
    ''',
    name: 'switched off_t size'
)
unswitched_off_size = cc.sizeof('off_t',
    prefix: '''
        #undef  _FILE_OFFSET_BITS
        #include <sys/types.h>
    ''',
    name: 'unswitched off_t size'
)
# Now the size without meddling, possibly enforced to 64 bits by system settings.
sizeof_off_t = cc.sizeof('off_t',
    prefix: '''
        #include <sys/types.h>
    ''',
    name: 'off_t size'
)
conf.set('SIZEOF_OFF_T', sizeof_off_t)
# Some paranoia about the limited choices we accept.
# 32 or 64 bit offsets, an the switched one better not be smaller.
if sizeof_off_t == -1
    error('Not even off_t found. I need some POSIX.')
elif sizeof_off_t != 4 and sizeof_off_t != 8
    error('unexpected size of off_t')
elif sizeof_off_t != switched_off_size
    if switched_off_size != 8
        error('bad switched off_t size')
    endif
    conf.set('LFS_INSENSITIVE', true)
endif
