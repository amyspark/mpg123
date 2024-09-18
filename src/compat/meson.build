src_compat_libcompat_sources = files(
    'compat_str.c',
    'compat.c',
    'wpathconv.h',
    'compat.h'
)

libcompat = static_library(
    'libcompat',
    src_compat_libcompat_sources,
    include_directories: includes,
    install: false,
)

# A smaller one for the output modules. Do not want to grow them
# unnecessarily with unused code.
src_compat_libcompat_str_sources = files(
  'compat_str.c',
  'compat.h',
)

libcompat_str = static_library(
    'libcompat_str',
    src_compat_libcompat_str_sources,
    include_directories: includes,
    install: false,
)

# A separate lib for the dlopen stuff, less linker noise.
src_compat_libcompat_dl_sources = files(
  'compat_dl.c',
  'wpathconv.h',
  'compat.h',
)

libcompat_dl = static_library(
    'libcompat_dl',
    src_compat_libcompat_dl_sources,
    link_with: libdl,
    include_directories: includes,
    install: false,
)
