umdf_include_dirs = include_directories(
  join_paths(wdk_root, 'Include', 'wdf', 'umdf', '2.15'),
  )

umdf_library_dirs = [
  join_paths(wdk_root, 'Lib', 'wdf', 'umdf', 'x64', '2.15'),
  ]

umdf_deps = [
  cpp.find_library('ntdll'),
  cpp.find_library('OneCoreUAP'),
  cpp.find_library('WdfDriverStubUm', dirs : umdf_library_dirs),
  ]

c_args = [
  '-DUMDF_VERSION_MAJOR=2',
  '-DUMDF_VERSION_MINOR=15',
  '-DUMDF_USING_NTSTATUS',
  '-DWIN32_LEAN_AND_MEAN=1',
  # These does not seem actually needed (for kmdf they _were_ needed),
  # but still adding them for MSVC cmdline alignment
  '-D_AMD64_=1',
  '-DAMD64',
  ]

srcs = files(
  'device.c',
  'driver.c',
  'queue.c',
  )

dll = shared_library('echo', srcs,
  c_args : c_args,
  include_directories : [umdf_include_dirs, '../../exe'],
    link_args : [
    '/DYNAMICBASE',
    '/SUBSYSTEM:WINDOWS,10.00',
    '/VERSION:10.0',
    ],
  dependencies : umdf_deps,

  # meson doesn't allow this argument for shared_library: Argument win_subsystem can only be used on executables
  # So, we don don't use it here, but specify in link_args.
  #win_subsystem : 'native,10.00',

  # This does not work at the moment, see https://github.com/mesonbuild/meson/issues/11923
  # We WA this by setting 'cpp_winlibs=' at project's default options.
  #override_options : [ 'c_winlibs=' ],
  )

stampinf = find_program('stampinf')
# inf2cat is located in /c/Program Files (x86)/Windows Kits/10/bin/x86/, no x64 counterpart.
inf2cat = find_program('Inf2Cat', dirs : wdk_root / 'bin' / 'x86')

# Here we actually have circular dependency:
#   echo.inx -> {copy} -> echo.inf -> {stampinf} -> echo.inf
# That's why 'inf' target below depends on echo.inx, not from 'echo.inf' as it actually should.
# NOTE: THIS ALSO BREAKS SEQUENTUAL REBUILD OF THE PROJECT (since next time it tries to
# re-stamp already stamped .inf and all falls apart).
infum = configure_file(
  input : 'echoum.inx',
  output : 'echoum.inf',
  copy : true)

infum_stamped = custom_target('infum',
  input : 'echoum.inx',
  output : 'echoum.inf',
  command : [stampinf,
    # -f <dir> -a <arch> : these should be specified first
    '-f', '@OUTDIR@' / 'echoum.inf', '-a', 'amd64',
    # mind -u vs. -k for kmdf and eventual version difference
    '-d', '*', '-v', '*', '-u', '2.15.0', '-x',
    ],
  )

cat = custom_target('cat',
  input : [infum, dll],
  output : 'echoum.cat',
  # Tool find echo.inf input file automatically, not way to specify explicitly as it seems
  # echo.inf contains a reference to echo.sys which also should be searchable
  command : [inf2cat, '/os:10_x64', '/driver:@OUTDIR@'],
  build_always: true,
  )
