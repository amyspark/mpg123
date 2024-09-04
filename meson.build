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


deps = []
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
    deps = [cc.find_library('dl')]
    dlopen = cc.has_function('dlopen', dependencies: deps)
    dlsym = cc.has_function('dlsym', dependencies: deps)
    dlclose = cc.has_function('dlclose', dependencies: deps)

    conf.set('HAVE_DLOPEN', dlopen)
    conf.set('HAVE_DLSYM', dlsym)
    conf.set('HAVE_DLCLOSE', dlclose)
    conf.set('HAVE_DLFCN_H', cc.has_header('dlfcn.h', dependencies: deps))

    modules = modules.enable_auto_if(dlopen and dlclose and dlsym)

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
check_yasm = false
if cpu_type.contains('x86')
    check = '.text\nvaddps %ymm0,%ymm0,%ymm0\n'
    f = configure_file(
        command: [python3, '-c', 'import sys; print(sys.argv[1])', '@0@'.format(check[0])],
        input: 'configure',
        output: '@0@.asm'.format(check[1]),
        capture: true,
    )
    avx_support = cc.compiles(
        f,
        args: ['-x', 'assembler'],
        name: 'assembler supports AVX instructions'
    )

    if not avx_support
        check_yasm = add_languages('nasm', required: false)
        if check_yasm
            x86asm = meson.get_compiler('nasm')

            if x86asm.compiles('.text\nvaddps %ymm0,%ymm0,%ymm0\n',
                args: ['-pgas', '-rcpp'],
                name: 'NASM supports GAS syntax and AVX instructions',
            )
                add_project_arguments('-pgas -rcpp', language: 'nasm')
                avx_support = true
            endif
        endif
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

if unswitched_off_size != sizeof_off_t
    if sizeof_off_t == 8
        message('Detected system with enforced 64 bit offsets, dropping suffixless symbols for uncryptic ABI breakage.')
        conf.set('FORCED_OFF_64', true)
    endif
endif

# Large file support without the magic. Check for the API explicitly.
# The shape-shifting off_t has to be avoided for the library builds.
# The mpg123 application might still use it to check if the API works.
# Or we limit it to portable, off_t-less API at some point.

use_largefile = sizeof_off_t == 8 or get_option('largefile').enabled()

if not use_largefile
    have_lseek64 = cc.has_function('lseek64')
    conf.set('HAVE_LSEEK64', have_lseek64)
    have_off64_t = cc.has_type('off64_t', prefix: '''
        #define _LARGEFILE64_SOURCE
        #include <sys/types.h>
        #include <unistd.h>
    ''')
    if have_lseek64 and have_off64_t
        use_largefile = true
        conf.set('LFS_LARGEFILE_64', true)
        conf.set('HAVE_O_LARGEFILE', cc.compiles('''
            #define _LARGEFILE64_SOURCE
            #include <sys/types.h>
            #include <sys/stat.h>
            #include <unistd.h>
            #include <fcntl.h>

            int foo()
            {
            return open("foo", O_RDONLY|O_LARGEFILE);
            }
        ''', name: 'we have O_LARGEFILE'))
    endif
endif

############## Function Checks

have_mmap = cc.has_function('mmap')
conf.set('HAVE_MMAP', have_mmap)
if not have_mmap
    headers = ['sys/ipc.h', 'sys/shm.h']
    funcs = ['shmget', 'shmat', 'shmdt', 'shmctl']
    buffer = true
    foreach h : headers
        v = cc.has_header(h)
        conf.set('HAVE_@0@'.format(h.to_upper().underscorify()), v)
        if not v
            buffer = false
        endif
    endforeach
    foreach f : funcs
        v = cc.function(h)
        conf.set('HAVE_@0@'.format(f.to_upper().underscorify()), v)
        if not v
            buffer = false
        endif
    endif
endif

# Check if system supports termios
posix_termios = cc.has_header('termios.h')
header_wincon_h = cc.has_header_symbol('ReadConsoleInput', prefix: '''
#include <windows.h>
#include <wincon.h>
''')
conf.set('HAVE_TERMIOS', posix_termios)
conf.set('HAVE_WINCON_H', header_wincon_h)

term_type = 'none'
if posix_termios
    term_type = 'posix'
elif header_wincon_h
    term_type = 'win32'
endif

funcs = [
    'random', 'setenv', 'unsetenv',
    'sched_setscheduler', 'setuid', 'getuid',
    'setpriority',
    'strerror', 'strerror_l', 'uselocale',
    'setlocale', 'nl_langinfo', 'mbstowcs', 'wcstombs', 'wcswidth', 'iswprint',
    'atoll',
    'mkfifo',
    'fork', 'execvp',
    'ctermid',
]
foreach f : funcs
    conf.set('HAVE_@0@'.format(f.to_upper()), cc.has_function(f))
endif

have_mkfifo = conf.get('HAVE_MKFIFO')
have_fork = conf.get('HAVE_FORK') and conf.get('HAVE_EXECVP')

conf.set('HAVE__SETMODE', cc.compiles('''
#include <io.h>
#include <fcntl.h>
#include <stdio.h>

int main()
{
  _setmode(STDIN_FILENO, _O_BINARY);
  _setmode(STDOUT_FILENO, _O_TEXT);
  return 0;
}
''', name: 'Win/DOS system with _setmode()'))

conf.set('HAVE_SETMODE', cc.compiles('''
#include <io.h>
#include <fcntl.h>
#include <stdio.h>

int main()
{
  setmode(STDIN_FILENO, _O_BINARY);
  setmode(STDOUT_FILENO, _O_TEXT);
  return 0;
}
''', name: 'Win/DOS system with setmode()'))

############## Header and Library Checks

headers = [
    'locale.h', 'langinfo.h', 'wchar.h', 'wctype.h',
    'netdb.h', 'sys/param.h', 'sys/socket.h', 'netinet/in.h', 'arpa/inet.h'
]
foreach h : headers
    conf.set('HAVE_@0@'.format(f.to_upper().underscorify()), cc.has_header(f))
endif

network_internal = 'unknown'
have_network = false
if conf.get('HAVE_NETDB_H') and conf.get('HAVE_SYS_PARAM_H') and conf.get('HAVE_SYS_SOCKET_H') and conf.get('HAVE_NETINET_IN_H') and conf.get('HAVE_ARPA_INET_H')
    network_internal = 'posix'
    have_network = true
endif


if not cc.has_function('gethostbyname', dependencies: [])
    network_deps = [
        cc.find_library('nsl'),
        cc.find_library('socket'),
        cc.find_library('network')
    ]
    foreach dep : network_deps
        if cc.has_function('gethostbyname', dependencies: dep)
            deps += [dep]
            break
        endif
    endif
endif

if not cc.has_function('socket', dependencies: [])
    dep = cc.find_library('socket')
    if cc.has_function('gethostbyname', dependencies: dep)
        deps += [dep]
    endif
endif

conf.set('HAVE_GETADDRINFO', cc.has_function('getaddrinfo', dependencies: deps))
have_ipv6 = conf.get('HAVE_GETADDRINFO')

deps += [cc.find_library('m'), cc.find_library('mx')]
cc.has_function('sqrt', deps: deps, required: true)
cc.has_function('powf', deps: deps, required: true)

conf.set('HAVE_BYTESWAP_H', cc.has_header('byteswap.h'))

############## Choose compiler flags and CPU

conf.set('HAVE_OS2_H', cc.has_header('os2.h'))
conf.set('ABI_ALIGN_FUN', cpu == 'x86')

# Choice of extra libmpg12 sources via lots of AM_CONDITIONALS.
# Those are defined after this block of CPU selection. Make sure
# that any new conditional gets added down there before using it
# here in definitions.

s_altivec = ['HAVE_ALTIVEC']
s_i386 = ['HAVE_I386']
s_i486 = s_i386 + ['HAVE_I486']
s_i586 = s_i386 + ['HAVE_I586']
s_i586d = s_i386 + ['HAVE_I586_DITHER']
s_3dnow = s_i386 + ['HAVE_3DNOW']
s_3dnowext = s_i386 + ['HAVE_3DNOWEXT', 'HAVE_MMXTAB']
s_3dnow_vintage = s_3dnow + ['HAVE_3DNOW_VINTAGE']
s_3dnowext_vintage = s_3dnowext + ['HAVE_3DNOWEXT_VINTAGE']

s_mmx = s_i386 + ['HAVE_MMX', 'HAVE_MMXTAB']
s_sse_vintage = s_i386 + ['HAVE_MMXTAB', 'HAVE_SSE_VINTAGE']
s_sse = s_sse_vintage + ['HAVE_SSE']
s_x86_64 = ['HAVE_X86_64', 'HAVE_X86_64_MONO']
s_x86_64_mono_synths = ['HAVE_X86_64_MONO']
s_x86_64_avx = ['HAVE_AVX']
s_x86multi = ['HAVE_GETCPUFLAGS']
s_x86_64_multi = ['HAVE_GETCPUFLAGS_X86_64']
s_dither = ['HAVE_DITHER']
s_neon = ['HAVE_NEON']
s_neon64 = ['HAVE_NEON64']
s_arm_multi = ['HAVE_GETCPUFLAGS_ARM']
s_arm = ['HAVE_ARM']

# See earlier fixme -- int32 and real are forcibly allowed
conf.set('HAVE_FLOATDCT', true)

more_sources=""
if cpu == 'generic_dither'
    add_project_arguments('-DOPT_GENERIC_DITHER -DREAL_IS_FLOAT', language: 'c')
    ccalign = false
elif cpu == 'altivec'
    add_project_arguments('-maltivec -DREAL_IS_FLOAT -DOPT_ALTIVEC', language: 'c')
    add_project_arguments(
        cc.get_supported_arguments('-faltivec'),
        language: 'c'
    )
    more_sources = s_altivec
elif cpu == 'arm_fpu'
    add_project_arguments('-DOPT_MULTI -DOPT_GENERIC -DOPT_GENERIC_DITHER -DOPT_NEON -DREAL_IS_FLOAT', language: 'c')
    more_sources = s_neon + s_dither + s_arm_multi
elif cpu == 'aarch64'
    add_project_arguments('-DOPT_MULTI -DOPT_GENERIC -DOPT_GENERIC_DITHER -DOPT_NEON64 -DREAL_IS_FLOAT', language: 'c')
    more_sources = s_neon64 + s_dither + s_arm_multi
elif cpu == 'x86'
    add_project_arguments('-DOPT_MULTI -DOPT_GENERIC -DOPT_GENERIC_DITHER -DOPT_I386 -DOPT_I586 -DOPT_I586_DITHER -DOPT_MMX -DOPT_3DNOW -DOPT_3DNOW_VINTAGE -DOPT_3DNOWEXT -DOPT_3DNOWEXT_VINTAGE -DOPT_SSE -DOPT_SSE_VINTAGE -DREAL_IS_FLOAT', language: 'c')
    more_sources = s_i386 + s_i586 + s_i586d + s_mmx + s_3dnow_vintage + s_3dnowext_vintage + s_sse + s_x86multi + s_dither
elif cpu == 'x86-64'
    add_project_arguments('-DOPT_MULTI -DOPT_X86_64 -DOPT_GENERIC -DOPT_GENERIC_DITHER -DREAL_IS_FLOAT', language: 'c')
    more_sources = '@0@ @1@ @2@'.format(s_x86_64, s_dither, s_arm_multi)
    if avx_support
        add_project_arguments('-DOPT_AVX')
    endif
else
    error('Unknown CPU type @0@'.format(cpu))
endif

conf.set('HAVE_ALTIVEC', more_sources.contains('HAVE_ALTIVEC'))
conf.set('HAVE_I386', more_sources.contains('HAVE_I386'))
conf.set('HAVE_I486', more_sources.contains('HAVE_I486'))
conf.set('HAVE_I586', more_sources.contains('HAVE_I586'))
conf.set('HAVE_I586_DITHER', more_sources.contains('HAVE_I586_DITHER'))
conf.set('HAVE_3DNOW', more_sources.contains('HAVE_3DNOW'))
conf.set('HAVE_3DNOWEXT', more_sources.contains('HAVE_3DNOWEXT'))
conf.set('HAVE_MMXTAB', more_sources.contains('HAVE_MMXTAB'))
conf.set('HAVE_3DNOW_VINTAGE', more_sources.contains('HAVE_3DNOW_VINTAGE'))
conf.set('HAVE_3DNOWEXT_VINTAGE', more_sources.contains('HAVE_3DNOWEXT_VINTAGE'))
conf.set('HAVE_MMX', more_sources.contains('HAVE_MMX'))
conf.set('HAVE_SSE_VINTAGE', more_sources.contains('HAVE_SSE_VINTAGE'))
conf.set('HAVE_SSE', more_sources.contains('HAVE_SSE'))
conf.set('HAVE_X86_64', more_sources.contains('HAVE_X86_64'))
conf.set('HAVE_X86_64_MONO', more_sources.contains('HAVE_X86_64_MONO'))
conf.set('HAVE_AVX', more_sources.contains('HAVE_AVX'))
conf.set('HAVE_GETCPUFLAGS', more_sources.contains('HAVE_GETCPUFLAGS'))
conf.set('HAVE_GETCPUFLAGS_X86_64', more_sources.contains('HAVE_GETCPUFLAGS_X86_64'))
conf.set('HAVE_DITHER', more_sources.contains('HAVE_DITHER'))
conf.set('HAVE_NEON', more_sources.contains('HAVE_NEON'))
conf.set('HAVE_NEON64', more_sources.contains('HAVE_NEON64'))
conf.set('HAVE_GETCPUFLAGS_ARM', more_sources.contains('HAVE_GETCPUFLAGS_ARM'))
conf.set('HAVE_ARM', more_sources.contains('HAVE_ARM'))

conf.set('HAVE_FEATURE', true)

# Finally insert the code switch for alignment, above cpu selection having possibly changed the setting.
conf.set('CCALIGN', ccalign)

############## Output module choice
