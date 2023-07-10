project('windows-driver-samples', 'c', 'cpp',
  version : '0.1',
  default_options : [
    'c_winlibs=',
    'cpp_winlibs='
    ]
  )

fs = import('fs')

cpp = meson.get_compiler('cpp')

tracewpp = find_program('tracewpp', required: true)

# We are parsing a path like this:
#  C:\Program Files (x86)\Windows Kits\10\bin\10.0.22621.0\x64\tracewpp.exe
wdk_root = fs.parent(fs.parent(tracewpp.full_path()))
wdk_ver  = fs.name(wdk_root)
wdk_root = fs.parent(fs.parent(wdk_root))

message('Using Windows SDK:')
message('  Path: ' + wdk_root)
message('  Version: ' + wdk_ver)

subdir('general')

