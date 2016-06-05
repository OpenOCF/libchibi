##
# CA build script
##

import os.path

Import('env')

ca_os = env.get('TARGET_OS')
ca_transport = env.get('TARGET_TRANSPORT')
secured = env.get('SECURED')
with_ra = env.get ('WITH_RA')
with_tcp = env.get('WITH_TCP')
src_dir = env.get('SRC_DIR')
root_dir = os.pardir
ca_path = os.curdir

print "ROOT DIR: ", Dir(root_dir).abspath
print "CA_PATH: ",  Dir(ca_path).abspath

#####################################################################
# Source files and Target(s)
######################################################################

print"Reading ca script %s"%ca_transport

env.PrependUnique(CPPPATH = [ os.path.join(root_dir, '../include/connectivity') ])
env.AppendUnique(CPPPATH = [ os.path.join(root_dir),
                             os.path.join(ca_path, 'include'),
                             os.path.join(ca_path, 'lib/libcoap-4.1.1'),
                             os.path.join(root_dir, 'logger/include/'),
                             os.path.join(ca_path, 'common/include'),
                             os.path.join(ca_path, 'util/include') ])

if ca_os not in ['arduino', 'windows', 'winrt']:
	env.AppendUnique(CPPDEFINES = ['WITH_POSIX'])

if ca_os not in ['darwin', 'ios', 'windows', 'winrt']:
	env.AppendUnique(LINKFLAGS = ['-Wl,--no-undefined'])

if ca_os in ['darwin','ios']:
	env.AppendUnique(CPPDEFINES = ['_DARWIN_C_SOURCE'])

# Getting common source files
env.SConscript('common/SConscript')

# Getting util source files
env.SConscript(Dir(ca_path).abspath + '/util/SConscript')

# The tinydtls library is found in '#extlibs/tinydtls', where the '#'
# is interpreted by SCons as the top-level iotivity directory where
# the SConscruct file is found.
if env.get('SECURED') == '1':
	if ca_os == 'tizen' and os.path.exists(root_dir + '/extlibs/tinydtls'):
		env.SConscript(os.path.join(root_dir, 'extlibs/tinydtls/SConscript'))
	else:
		env.SConscript('#extlibs/tinydtls/SConscript')
	if ca_os == 'tizen' and os.path.exists(root_dir + '/extlibs/timer'):
		env.SConscript(os.path.join(root_dir, 'extlibs/timer/SConscript'))
		env.AppendUnique(CPPPATH = [os.path.join(root_dir, 'extlibs/timer')])
	else:
		env.SConscript('#extlibs/timer/SConscript')
		env.AppendUnique(CPPPATH = ['#extlibs/timer'])

env.AppendUnique(CA_SRC = [os.path.join(ca_path,
                                        'adapter_util/caadapterutils.c')])

if env.get('SECURED') == '1':
	env.AppendUnique(CA_SRC = [os.path.join(ca_path,
                                                'adapter_util/caadapternetdtls.c')])

	# 'external' dir missing in official sources
	# env.AppendUnique(CPPPATH = [os.path.join(root_dir,
        #                                          'external/inc')])

if env.get('DTLS_WITH_X509') == '1':
	env.AppendUnique(CPPPATH = [src_dir + 'src/connectivity/include/pkix'])
	env.AppendUnique(CPPPATH = [src_dir + 'src/tinydtls/ecc/'])
	env.AppendUnique(CPPPATH = [src_dir + 'src/tinydtls/sha2/'])
	env.AppendUnique(CPPDEFINES = ['__WITH_X509__'])
	if not env.get('RELEASE'):
		env.AppendUnique(CPPDEFINES = ['X509_DEBUG'])
	pkix_src = Glob('adapter_util/pkix/*.c');
	env.AppendUnique(CA_SRC = pkix_src)

ca_common_src = None

if env.get('ROUTING') == 'GW':
	env.AppendUnique(CPPDEFINES = ['ROUTING_GATEWAY'])
elif env.get('ROUTING') == 'EP':
	env.AppendUnique(CPPDEFINES = ['ROUTING_EP'])

if ca_os == 'arduino':
	env.AppendUnique(CPPDEFINES = ['SINGLE_THREAD'])
	env.AppendUnique(CPPDEFINES = ['WITH_ARDUINO'])
	print "setting WITH_ARDUINO"
	ca_common_src = [
		'caconnectivitymanager.c',
		'cainterfacecontroller.c',
		'camessagehandler.c',
		'canetworkconfigurator.c',
		'caprotocolmessage.c',
		'caretransmission.c',
		]
else:
	ca_common_src = [
		'caconnectivitymanager.c',
		'cainterfacecontroller.c',
		'camessagehandler.c',
		'canetworkconfigurator.c',
		'caprotocolmessage.c',
		'caqueueingthread.c',
		'caretransmission.c',
		]
	if (('IP' in ca_transport) or ('ALL' in ca_transport)):
		env.AppendUnique(CA_SRC = [os.path.join(ca_path, 'cablockwisetransfer.c') ])
		env.AppendUnique(CPPDEFINES = ['WITH_BWT'])
	if secured == '1':
		env.AppendUnique(CPPDEFINES = ['__WITH_DTLS__'])
		if ca_os == 'tizen' and os.path.exists(root_dir + '/extlibs/tinydtls'):
			env.AppendUnique(CPPPATH = [os.path.join(root_dir, 'extlibs/tinydtls')])
		else:
			env.AppendUnique(CPPPATH = ['#extlibs/tinydtls'])

ca_common_src = [
        os.path.join(ca_path, d) for d in ca_common_src ]

env.AppendUnique(CA_SRC = ca_common_src)

if 'ALL' in ca_transport:
		transports = [ 'ip_adapter', 'bt_edr_adapter', 'bt_le_adapter']
		if with_ra:
				transports.append ('ra_adapter')
		if ca_os in ['android']:
				transports.append ('nfc_adapter')
		env.SConscript(dirs = [
				os.path.join(ca_path, d) for d in transports ])

if 'IP' in ca_transport:
	env.SConscript(os.path.join(ca_path, 'ip_adapter/SConscript'))
	if ca_os == 'arduino':
		if with_tcp:
			transports = [ 'ip_adapter', 'tcp_adapter']
			env.SConscript(dirs = [
				os.path.join(ca_path, d) for d in transports ])

if 'BT' in ca_transport:
	env.SConscript(os.path.join(ca_path, 'bt_edr_adapter/SConscript'))

if 'BLE' in ca_transport:
	env.SConscript(os.path.join(ca_path, 'bt_le_adapter/SConscript'))

if 'NFC' in ca_transport:
	env.SConscript(os.path.join(ca_path, 'nfc_adapter/SConscript'))

if ca_os in ['linux', 'tizen', 'android']:
	if with_tcp == True:
		env.SConscript(os.path.join(ca_path, 'tcp_adapter/SConscript'))
		env.AppendUnique(CPPDEFINES = ['WITH_TCP'])

if ca_os in ['linux', 'tizen', 'android', 'arduino', 'ios']:
	if (('BLE' in ca_transport) or ('BT' in ca_transport) or ('ALL' in ca_transport)):
		env.AppendUnique(CPPDEFINES = ['WITH_TCP'])

print "Include path is %s" % env.get('CPPPATH')
print "Files path is %s" % env.get('CA_SRC')

lib_env = env.Clone()

if ca_os == 'android':
    lib_env.AppendUnique(LINKFLAGS = ['-Wl,-soname,libconnectivity_abstraction.so'])

if ca_os in ['android', 'tizen']:
	lib_env.AppendUnique(LIBS = ['coap'])
	if lib_env.get('SECURED') == '1':
		lib_env.AppendUnique(LIBS = ['tinydtls'])
		lib_env.AppendUnique(LIBS = ['timer'])
	if ca_os != 'android':
		lib_env.AppendUnique(LIBS = ['rt'])
	calib = lib_env.SharedLibrary('connectivity_abstraction', lib_env.get('CA_SRC'))
else:
	calib = lib_env.StaticLibrary('connectivity_abstraction', lib_env.get('CA_SRC'))
lib_env.InstallTarget(calib, 'libconnectivity_abstraction')
lib_env.UserInstallTargetLib(calib, 'libconnectivity_abstraction')
