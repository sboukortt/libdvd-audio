project('libdvd-audio', 'c', version: '1.0.1')

version_components = meson.project_version().split('.')
header_config = configuration_data()
header_config.set('LIBDVDAUDIO_MAJOR_VERSION', version_components[0])
header_config.set('LIBDVDAUDIO_MINOR_VERSION', version_components[1])
header_config.set('LIBDVDAUDIO_RELEASE_VERSION', version_components[2])
header = configure_file(
	input: 'include/dvd-audio.h.in',
	output: 'dvd-audio.h',
	configuration: header_config,
	install: true,
	install_dir: 'include',
)

bitstream_table = executable(
	'bitstream-table',
	'src/array.h',
	'src/array.c',
	'src/bitstream-table.c',
	native: true,
)

generated_tables = []
foreach kind_and_flag: [
	['read_bits', '--rb'],
	['read_unary', '--ru'],
	['unread_bit', '--urb'],
]
	kind = kind_and_flag[0]
	foreach endianness: ['be', 'le']
		generated_tables += custom_target(
			f'@kind@_table_@endianness@.h',
			output: f'@kind@_table_@endianness@.h',
			command: [bitstream_table, kind_and_flag[1], f'--@endianness@'],
			capture: true,
		)
	endforeach
endforeach

huffman = executable(
	'huffman',
	'src/huffman.h',
	'src/huffman.c',
	'src/parson.h',
	'src/parson.c',
	c_args: ['-DEXECUTABLE'],
	native: true,
)

generated_codebooks = []
foreach i: [1, 2, 3]
	generated_codebooks += custom_target(
		f'mlp_codebook@i@.h',
		output: f'mlp_codebook@i@.h',
		input: f'src/mlp_codebook@i@.json',
		command: [huffman, '-i', '@INPUT@'],
		capture: true,
	)
endforeach

math_dep = meson.get_compiler('c').find_library('m', required: false)

aob_flags = []
if target_machine.kernel() == 'linux'
	aob_flags += ['-DDVD_STRUCT_IN_LINUX_CDROM_H', '-DHAVE_LINUX_DVD_STRUCT']
endif

libdvda = both_libraries(
	'dvd-audio',
	'src/cppm/cppm.h',
	'src/cppm/cppm.c',
	'src/cppm/dvd_css.h',
	'src/cppm/dvd_css.c',
	'src/cppm/ioctl.h',
	'src/cppm/ioctl.c',
	'src/aob.h',
	'src/aob.c',
	'src/array.h',
	'src/array.c',
	'src/audio_ts.h',
	'src/audio_ts.c',
	'src/bitstream.h',
	'src/bitstream.c',
	'src/dvd-audio.c',
	'src/func_io.h',
	'src/func_io.c',
	'src/huffman.h',
	'src/huffman.c',
	'src/mini-gmp.h',
	'src/mini-gmp.c',
	'src/packet.h',
	'src/packet.c',
	'src/pcm.h',
	'src/pcm.c',
	'src/mlp.h',
	'src/mlp.c',
	header,
	generated_tables,
	generated_codebooks,
	c_args: ['-DHAS_CPPM', aob_flags],
	dependencies: [math_dep],
	version: meson.project_version(),
	install: true,
)

pkgconfig = import('pkgconfig')
pkgconfig.generate(
	libdvda,
	name: meson.project_name(),
	version: meson.project_version(),
	description: 'DVD-Audio extraction library',
)

executable(
	'dvda-debug-info',
	'utils/dvda-debug-info.c',
	link_with: [libdvda],
	install: true,
)

executable(
	'dvda2wav',
	'utils/dvda2wav.c',
	link_with: [libdvda],
	include_directories: ['src'],
	install: true,
)
