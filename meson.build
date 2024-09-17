project('mpg123', 'c',
  meson_version: '>= 1.3.0',
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
win32_specific_codes = conf.get('HAVE_WINDOWS_H', false) and os != 'cygwin'

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

if os in ['windows', 'cygwin'] and modules and win32_specific_codes
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

# The full list of supported modules to check, first come, first serve.
check_modules = get_option('with_audio')
output_modules = []
check_forced = false

if check_modules != []
    message('Limiting outputs to build according to your preference')
    check_forced = true
else
    check_modules = [
        'pulse', 'alsa', 'tinyalsa', 'oss', 'coreaudio', 'sndio', 'sun',
        'win32', 'win32_wasapi', # 'os2', 'esd',
        'jack', 'portaudio', 'sdl', # 'nas', 'arts',
        'openal',
    ]
endif

# Dummy output module is always built alongside libout123, but not
# when just building the modules.
if build_libout123
	check_modules += ['dummy']
endif

foreach m : check_modules
    if m == 'dummy'
        output_modules += [m]
        conf.set('HAVE_@0@'.format(m.to_upper()), true)
    elif m == 'oss'
        headers = ['sys/soundcard.h', 'linux/soundcard.h', 'machine/soundcard.h']
        check = false
        foreach h : headers
            v = cc.has_header(h)
            conf.set('HAVE_@0@'.format(h.to_upper().underscorify()), v)
            check = check or v
        endif
        if check
            output_modules += [m]
            conf.set('HAVE_@0@'.format(m.to_upper()), true)
        elif check_forced
            error('A required header for @0@ is missing'.format(m))
        endif
    elif m == 'tinyalsa'
        tinyalsa_dep = dependency('tinyalsa', required: check_forced)
        if cc.has_function('pcm_open',
            prefix: '#include <tinyalsa/asoundlib.h>',
            dependencies: tinyalsa_dep, required: check_forced)
            output_modules += [m]
            conf.set('HAVE_@0@'.format(m.to_upper()), true)
        endif
    elif m == 'alsa'
        if conf.get('HAVE_QSA', false)
            conf.set('HAVE_ALSA', false)
        else
            alsa_dep = cc.find_library('asound', required: check_forced)
            if cc.has_function('snd_pcm_open', prefix: '#include <alsa/asoundlib.h>', dependencies: alsa_dep, required: check_forced)
                output_modules += [m]
                conf.set('HAVE_@0@'.format(m.to_upper()), true)
            endif
        endif
    elif m == 'jack'
        jack_dep = dependency('jack', required: check_forced)
        # JACK module uses semaphores, hence -lpthread.
        # Assuming a good install exposing -lpthread.
        if cc.has_function('sem_post', dependencies: jack_dep, required: check_forced)
            output_modules += [m]
            conf.set('HAVE_@0@'.format(m.to_upper()), true)
        endif
    elif m == 'pulse'
        pulse_dep = dependency('libpulse-simple', required: check_forced)
        if pulse_dep.found()
            output_modules += [m]
            conf.set('HAVE_@0@'.format(m.to_upper()), true)
        endif
    elif m == 'portaudio'
        # Assuming a good install of portaudio, with pkg-config.
        # NOTE: if this doesn't link, add kernel32, ole32, setupapi, winmm
        portaudio_dep = dependency('portaudio-2.0', required: check_forced)
        if cc.has_function('Pa_Initialize', prefix: '#include <portaudio.h>', dependencies: portaudio_dep)
            output_modules += [m]
            conf.set('HAVE_@0@'.format(m.to_upper()), true)
            if not cc.has_function('Pa_GetVersion', dependencies: portaudio_dep)
                # portaudio v18 API is wanted.
                conf.set('PORTAUDIO18', true)
            endif
        endif
    elif m == 'sdl'
        sdl_dep = dependency('sdl2', required: check_forced, fallback: 'sdl')
        if sdl_dep.found()
            output_modules += [m]
            conf.set('HAVE_@0@'.format(m.to_upper()), true)
        endif
    elif m == 'win32'
        win32_dep = cc.find_library('winmm', required: check_forced)
        if cc.has_function('waveOutOpen', prefix: '#include <windows.h>', dependencies: win32_dep, required: check_forced)
            output_modules += [m]
            conf.set('HAVE_@0@'.format(m.to_upper()), true)
        endif
    elif m == 'win32_wasapi'
        win32_wasapi_dep = [
            cc.find_library('ole32', required: check_forced)
            cc.find_library('avrt', required: check_forced)
        ]
        if cc.compiles('''
            #define COBJMACROS 1
            #define _WIN32_WINNT 0x601
            #include <initguid.h>
            #include <audioclient.h>
            #include <mmdeviceapi.h>
            #include <avrt.h>
            int main(){
                /* UUID Checks */
                GUID *IDs[] = {
                    &CLSID_MMDeviceEnumerator,
                    &IID_IMMDeviceEnumerator,
                    &IID_IAudioClient,
                    &IID_IAudioRenderClient
                };
                return 0;
            }
            ''', name: 'we have wasapi headers', dependencies: win32_wasapi_dep, required: check_forced)
            output_modules += [m]
            conf.set('HAVE_@0@'.format(m.to_upper()), true)
        endif
    elif m == 'sndio'
        sndio_dep = dependency('sndio', required: check_forced)
        if sndio_dep.found()
            output_modules += [m]
            conf.set('HAVE_@0@'.format(m.to_upper()), true)
        endif
    elif m == 'sun'
        headers = ['sun/audioio.h', 'sys/audioio.h', 'asm/audioio.h', 'sys/audio.h']
        check = false
        foreach h : headers
            v = cc.has_header(h)
            conf.set('HAVE_@0@'.format(h.to_upper().underscorify()), v)
            check = check or v
        endforeach
        if check
            output_modules += [m]
            conf.set('HAVE_@0@'.format(m.to_upper()), true)
        elif check_forced
            error('A required header for @0@ is missing'.format(m))
        endif
    elif m == 'coreaudio'
        coreaudio_dep = [
            dependency('appleframeworks', modules : 'AudioToolbox', required: true)
        ]
        headers = ['AudioUnit/AudioUnit.h', 'AudioToolbox/AudioToolbox.h']
        check = true
        foreach h : headers
            v = cc.has_header(h, dependencies: coreaudio_dep)
            conf.set('HAVE_@0@'.format(h.to_upper().underscorify()), v)
            check = check and v
        endforeach
        if check
            if not cc.links('''
                #include <AudioUnit/AudioUnit.h>
                int main()
                {
                    AudioUnitInitialize(NULL);
                    return 0;
                }
            ''', name: 'if AudioUnit functions are available within AudioToolbox')
                coreaudio_dep += [
                    dependency('appleframeworks', modules : 'AudioUnit', required: true)
                ]
            endif
            headers = ['CoreServices/CoreServices.h']
            foreach h : headers
                v = cc.has_header(h)
                conf.set('HAVE_@0@'.format(h.to_upper().underscorify()), v)
            endforeach
            buffer = buffer.require(not modules.disabled())
            if modules.disabled()
                warning('Disabling buffer because of directly linked CoreAudio! Use the module if you need the buffer.')
            endif
            output_modules += [m]
            conf.set('HAVE_@0@'.format(m.to_upper()), true)
        elif check_forced
            error('A required header for @0@ is missing'.format(m))
        endif
    elif m == 'openal'
        # Assuming openal-soft and OpenAL.framework for sanity reasons.
        openal_dep = dependency('appleframeworks', modules : 'OpenAL', required: false)
        headers = ['OpenAL/al.h', 'OpenAL/alc.h', 'AL/al.h', 'AL/alc.h', 'al.h', 'alc.h']
        if not openal_dep.found()
            openal_dep = dependency('openal', required: true)
        endif
        foreach h : headers
            v = cc.has_header(h, dependencies: openal_dep)
            conf.set('HAVE_@0@'.format(h.to_upper().underscorify()), v)
        endforeach
        if conf.get('HAVE_OPENAL_AL_H', false) and conf.get('HAVE_OPENAL_ALC_H', false)
            conf.set('OPENAL_SUBDIR_OPENAL', true)
            output_modules += [m]
            conf.set('HAVE_@0@'.format(m.to_upper()), true)
        elif conf.get('HAVE_AL_AL_H', false) and conf.get('HAVE_AL_ALC_H', false)
            conf.set('OPENAL_SUBDIR_OPENAL', true)
            output_modules += [m]
            conf.set('HAVE_@0@'.format(m.to_upper()), true)
        elif conf.get('HAVE_AL_H', false) and conf.get('HAVE_ALC_H', false)
            # Windows?
            output_modules += [m]
            conf.set('HAVE_@0@'.format(m.to_upper()), true)
        elif check_forced
            error('A required header for @0@ is missing'.format(m))
        endif
    else
        error('Unsupported/-known output @0@ demanded!'.format(m))
    endif
endforeach

# Did user choose default audio subsystem ?
# That's (beginning of) the list for mpg123's internal default.
with_default_audio = get_option('with_default_audio')
default_output_modules = []
if with_default_audio.length() != 0
    default_output_modules = with_default_audio
else
    foreach m : output_modules
        if m == 'dummy'
            continue
        else
            default_output_modules += [m]
        endif
    endforeach
endif

default_output_module = ''
if default_output_modules.length() != 0
    default_output_module = default_output_modules[0]
endif

# Without actual modules, there is only the builtin one.
if modules.disabled()
  # Module-less build needs _some_ default module.
  if default_output_module == ''
    default_output_module = 'dummy'
  endif
  default_output_modules = [default_output_module]
endif

# Setup the static build.
conf.set('BUILD_@0@'.format(default_output_module.to_upper()), true)
conf.set('DEFAULT_OUTPUT_MODULE', default_output_modules.join(','))

############## Compiler Optimizations

if get_option('optimization') >= 2
    add_project_arguments(
        cc.get_supported_arguments([
            '-fomit-frame-pointer', '-funroll-all-loops', '-finline-functions', '-ffast-math',
            '/fp:fast',
        ]),
        language: 'c'
    )
endif

############## Seektable size

if seektable > 0
    conf.set('FRAME_INDEX', 1)
endif
# Define that always... also without frame index enabled.
conf.set('INDEX_SIZE', seektable)

############## Win32 function checks

# Check if we want Unicode for Win32. Cygwin and Midipix does not need _wopen
compat_libs = []

win32_unicode = 'unneeded'
win32_sockets = false # disabled, does this have to be a feature
win32_sockets_working = false
win32_wide_working = false
win32_winver_bump = false

### mingw.org may not work properly for newer APIs
if cc.has_define('__MINGW32__') and cc.has_header('_mingw_mac.h')
    warning('You are using the mingw.org distribution\nmpg123 may not build properly\nPlease switch to using mingw-w64 if possible')
endif

if win32_specific_codes
    #### Check for Wide functions
    win32_unicode = cc.has_function('_wopen', prefix: '#include <io.h>')
    if uwp_build
        conf.set('WANT_WIN32_UNICODE', true)
    elif win32_unicode
        shlwapi_dep = cc.find_library('shlwapi')
        if cc.links('''
            #include <windows.h>
            #include <shlwapi.h>
            #include <stdlib.h>
            int main()
            {
                MultiByteToWideChar (0, 0, NULL, 0, NULL, 0);
                WideCharToMultiByte (0, 0, NULL, 0, NULL, 0, NULL, NULL);
                PathCombineW(0,0,0);
                return 0;
            }''', name: 'Unicode functions working', dependencies: shlwapi_dep)
            win32_wide_working = true
        elif not win32_winver_bump
            win32_winver_bump = true
        endif

        if not win32_wide_working
            win32_wide_working = cc.links('''
                #define WINVER 0x501
                #define _WIN32_WINNT 0x501
                #include <windows.h>
                #include <shlwapi.h>
                #include <stdlib.h>
                int main()
                {
                    MultiByteToWideChar (0, 0, NULL, 0, NULL, 0);
                    WideCharToMultiByte (0, 0, NULL, 0, NULL, 0, NULL, NULL);
                    PathCombineW(0,0,0);
                    return 0;
                }''', name: 'Unicode functions working with WINVER bump')
        endif

        if win32_wide_working
            compat_libs += [shlwapi_dep]
            conf.set('WANT_WIN32_UNICODE', true)
        else
            error('Unicode support for Win32 not working')
        endif
    else
        error('Unicode File Open for Win32 not available')
    endif

    #### Check for Network functions
    win32_sockets = cc.has_header('ws2tcpip.h')
    if not win32_sockets
        error('Please update your headers to support winsock 2.2.')
    endif
    conf.set('HAVE_WS2TCPIP_H', win32_sockets)
    if win32_sockets
        ws2_32_dep = cc.find_library('ws2_32')
        if cc.links('''
            #include <winsock2.h>
            #include <ws2tcpip.h>
            #include <stdlib.h>
            int main()
            {
                getaddrinfo(NULL, NULL, NULL, NULL);
                freeaddrinfo(NULL);
                return 0;
            }''', name: 'if winsock2 API is available', dependencies: ws2_32_dep)
            win32_sockets_working = true
        elif not win32_winver_bump
            win32_winver_bump = true
        endif

        if not win32_sockets_working
            win32_sockets_working = cc.links('''
                #define WINVER 0x501
                #define _WIN32_WINNT 0x501
                #include <winsock2.h>
                #include <ws2tcpip.h>
                #include <stdlib.h>
                int main()
                {
                    getaddrinfo(NULL, NULL, NULL, NULL);
                    freeaddrinfo(NULL);
                    return 0;
                }
            ''', name: 'deeper if winsock2 API is available', dependencies: ws2_32_dep)
        endif

        if win32_sockets_working
            network_internal = 'winsock2'
            have_network = true
            have_ipv6 = true
        else
            error('Please update your headers to support winsock 2.2.')
        endif
    endif

    #### Check for Win32 Named Pipe functions
    win32_fifo_working = false
    winver = ""
    if get_option('fifo').allowed()
        if cc.links('''
            #include <windows.h>
            int main(){
                CreateNamedPipeA(NULL,PIPE_ACCESS_DUPLEX|FILE_FLAG_OVERLAPPED,PIPE_TYPE_BYTE,1,255,255,0,NULL);
                CreateNamedPipeW(NULL,PIPE_ACCESS_DUPLEX|FILE_FLAG_OVERLAPPED,PIPE_TYPE_BYTE,1,255,255,0,NULL);
                return 0;
            }''', name: 'we have Named Pipes')
            win32_fifo_working = true
        elif not win32_winver_bump
            win32_winver_bump = true
        endif

        if not win32_fifo_working
            win32_fifo_working = cc.links('''
                #define WINVER 0x501
                #define _WIN32_WINNT 0x501
                #include <windows.h>
                int main(){
                    CreateNamedPipeA(NULL,PIPE_ACCESS_DUPLEX|FILE_FLAG_OVERLAPPED,PIPE_TYPE_BYTE,1,255,255,0,NULL);
                    CreateNamedPipeW(NULL,PIPE_ACCESS_DUPLEX|FILE_FLAG_OVERLAPPED,PIPE_TYPE_BYTE,1,255,255,0,NULL);
                    return 0;
                }''', name: 'we have Named Pipes with WINVER bump')
        endif

        have_mkfifo = win32_fifo_working
        if win32_fifo_working
            conf.set('WANT_WIN32_FIFO', true)
        else
            error('Unicode support for Win32 not working')
        endif
    else
        error('Unicode File Open for Win32 not available')
    endif

    #### WINVER Bump
    if win32_winver_bump
        # Windows Vista and later APIs
        conf.set('WINVER', '0x0600')
        conf.set('_WIN32_WINNT', '0x0600')
        # getaddrinfo and freeaddrinfo are available for Win2K and above
        # Bug: MinGW.org w32api ws2tcpip.h incorrectly wants (_WIN32_WINNT >= 0x0501) for getaddrinfo and freeaddrinfo
        # MultiByteToWideChar and WideCharToMultiByte are available for Win2K and above
        # GetThreadErrorMode only available on Vista and Later
    endif
endif

#### Check mingw.org for EOVERFLOW
eoverflow_present = cc.has_header_symbol('errno.h', 'EOVERFLOW')
if not eoverflow_present
    conf.set('EOVERFLOW', 'EFBIG')
endif

ipv6 = get_option('ipv6')

# make it clear that auto is the default for both enable and with options.
network_type = get_option('network')

if network_type == 'auto'
    
endif
