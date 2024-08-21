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

conf = configuration_data()

conf.set('HAVE_CLOCK_GETTIME', cc.has_function('clock_gettime'))

modules = get_option('modules').enable_if(
    get_option('default_library') == 'shared'
)

conf.set('DYNAMIC_BUILD', modules.allowed())

conf.set('HAVE_WINDOWS_H', host_machine.system() in ['windows', 'cygwin'] and cc.has_header('windows.h'))

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

android_build = host_machine.system() == 'android'

if host_machine.system() in ['windows', 'cygwin'] and modules
    modules.enable_auto_if(
        cc.has_function('GetThreadErrorMode', prefix: '#include <windows.h>')
    )
endif


libdl = []
if not modules
    message('Modules disabled, not checking for dynamic loading.')
elif host_machine.system() in ['windows', 'cygwin']
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
    conf.set_quoted('LT_MODULE_EXT', host_machine.system() == 'windows' ? '.dll' : '.so')
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
