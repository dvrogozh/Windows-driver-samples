kmdf_include_dirs = include_directories(
  join_paths(wdk_root, 'Include', wdk_ver, 'km'),
  join_paths(wdk_root, 'Include', wdk_ver, 'km', 'crt'),
  join_paths(wdk_root, 'Include', 'wdf', 'kmdf', '1.15'),
  )

cpp.has_header('ntddk.h',
  include_directories : kmdf_include_dirs,
  required : true,
  )

kmdf_library_dirs = [
  join_paths(wdk_root, 'Lib', wdk_ver, 'km', 'x64'),
  join_paths(wdk_root, 'Lib', 'wdf', 'kmdf', 'x64', '1.15'),
  ]

kmdf_deps = [
  cpp.find_library('BufferOverflowFastFailK', dirs : kmdf_library_dirs),
  cpp.find_library('WdfLdr', dirs : kmdf_library_dirs),
  cpp.find_library('WdfDriverEntry', dirs : kmdf_library_dirs),
  cpp.find_library('ntoskrnl', dirs : kmdf_library_dirs),
  ]

c_args = [
  # https://learn.microsoft.com/en-us/cpp/build/reference/kernel-create-kernel-mode-binary?view=msvc-170
  '/kernel',
  # Otherwise fails with:
  # C:\Program Files (x86)\Windows Kits\10\include\10.0.22621.0\shared\ntdef.h(197): fatal error C1189: #error:  "No Target Architecture"
  '-D_AMD64_',
  '-DAMD64',
  ]

srcs = files(
  'device.c',
  'driver.c',
  'queue.c',
  )

sys = executable('echo', srcs,
  c_args : c_args,
  include_directories : [kmdf_include_dirs, '../../exe'],
  link_args : [
    # https://learn.microsoft.com/en-us/cpp/build/reference/driver-windows-nt-kernel-mode-driver?view=msvc-170
    # Might be:
    # * /DRIVER
    # * /DRIVER:UPONLY
    # * /DRIVER:WDM
    '/Driver',
    # https://learn.microsoft.com/en-us/cpp/build/reference/entry-entry-point-symbol?view=msvc-170
    # Setting in MSVC: Linker > Advanced > Entry Point
    '/ENTRY:FxDriverEntry',
    ],
  dependencies : kmdf_deps,
  win_subsystem : 'native,10.00',
  # This does not work at the moment, see https://github.com/mesonbuild/meson/issues/11923
  # We WA this by setting 'cpp_winlibs=' at project's default options.
  #override_options : [ 'c_winlibs=' ],
  name_suffix : 'sys',
  )

stampinf = find_program('stampinf')
# inf2cat is located in /c/Program Files (x86)/Windows Kits/10/bin/x86/, no x64 counterpart.
inf2cat = find_program('Inf2Cat', dirs : wdk_root / 'bin' / 'x86')

# Here we actually have circular dependency:
#   echo.inx -> {copy} -> echo.inf -> {stampinf} -> echo.inf
# That's why 'inf' target below depends on echo.inx, not from 'echo.inf' as it actually should.
# NOTE: THIS ALSO BREAKS SEQUENTUAL REBUILD OF THE PROJECT (since next time it tries to
# re-stamp already stamped .inf and all falls apart).
inf = configure_file(
  input : 'echo.inx',
  output : 'echo.inf',
  copy : true)

inf_stamped = custom_target('inf',
  input : 'echo.inx',
  output : 'echo.inf',
  command : [stampinf,
    # -f <dir> -a <arch> : these should be specified first
    '-f', '@OUTDIR@' / 'echo.inf', '-a', 'amd64',
    '-d', '*', '-v', '*', '-k', '1.15', '-x',
    ],
  )

cat = custom_target('cat',
  input : [inf, sys],
  output : 'echo.cat',
  # Tool find echo.inf input file automatically, not way to specify explicitly as it seems
  # echo.inf contains a reference to echo.sys which also should be searchable
  command : [inf2cat, '/os:10_x64', '/driver:@OUTDIR@'],
  build_always: true,
  )
