import os
import SCons.Tool.textfile

locales = []

cflags = '-O2 -g -pipe'
version = '2.0.3'

AddOption('--prefix', dest='prefix', metavar='DIR',
          help='installation prefix')
AddOption('--libdir', dest='libdir', metavar='DIR',
          help='installation libdir')
AddOption('--libexecdir', dest='libexecdir', metavar='DIR',
          help='installation libexecdir')
AddOption('--datadir', dest='datadir', metavar='DIR',
          help='installation datadir')
AddOption('--rpath', dest='rpath', metavar='DIR',
          help='encode rpath in the executables')

opts = Variables('configure.conf')
opts.Add('PREFIX', default='/usr/local')
opts.Add('LIBDIR', default='/usr/local/lib')
opts.Add('LIBEXECDIR', default='/usr/local/lib')
opts.Add('DATADIR', default='/usr/local/share')

def PassVariables(envvar, env):
    for (x, y) in envvar:
        if x in os.environ:
            print 'Warning: you\'ve set %s in the environmental variable!' % x
            env[y] = os.environ[x]

env = Environment(ENV=os.environ,
                  CFLAGS=cflags, CXXFLAGS=cflags, 
                  CPPPATH=['.'], SUBSTFILESUFFIX='.in')
opts.Update(env)

if GetOption('prefix') is not None:
    env['PREFIX'] = GetOption('prefix')
    env['LIBDIR'] = env['PREFIX'] + '/lib'
    env['LIBEXECDIR'] = env['PREFIX'] + '/lib/'
    env['DATADIR'] = env['PREFIX'] + '/share/'

if GetOption('libdir') is not None:
    env['LIBDIR'] = GetOption('libdir')

if GetOption('libexecdir') is not None:
    env['LIBEXECDIR'] = GetOption('libexecdir')

if GetOption('datadir') is not None:
    env['DATADIR'] = GetOption('datadir')

opts.Save('configure.conf', env)

if GetOption('rpath') is not None:
    env.Append(LINKFLAGS='-Wl,-R -Wl,%s' % GetOption('rpath'))

envvar = [('CC', 'CC'),
          ('CXX', 'CXX'),
          ('CFLAGS', 'CFLAGS'),
          ('CXXFLAGS', 'CXXFLAGS'),
          ('LDFLAGS', 'LINKFLAGS')]
PassVariables(envvar, env)

data_dir = env['DATADIR'] + '/scim-sunpinyin'
icons_dir = env['DATADIR'] + '/scim-sunpinyin/icons'
bin_dir = env['LIBEXECDIR'] + '/scim-sunpinyin'
gettext_package = 'scim-sunpinyin'

extra_cflags  = ' -DSCIM_SUNPINYIN_LOCALEDIR=\'"%s"\'' % (env['DATADIR'] + '/locale')
extra_cflags += ' -DSCIM_SUNPINYIN_ICON_DIR=\'"%s"\'' % icons_dir
extra_cflags += ' -DLIBEXECDIR=\'"%s"\'' % bin_dir
extra_cflags += ' -DGETTEXT_PACKAGE=\'"%s"\'' % gettext_package
extra_cflags += ' -Isrc'

env.Append(CFLAGS=extra_cflags)
env.Append(CXXFLAGS=extra_cflags)
env.Replace(SHLIBPREFIX = '')
#
#==============================configure================================
#
def CheckPKGConfig(context, version='0.12.0'):
    context.Message( 'Checking for pkg-config... ' )
    ret = context.TryAction('pkg-config --atleast-pkgconfig-version=%s' % version)[0]
    context.Result(ret)
    return ret

def CheckPKG(context, name):
    context.Message( 'Checking for %s... ' % name )
    ret = context.TryAction('pkg-config --exists \'%s\'' % name)[0]
    context.Result(ret)
    return ret

conf = Configure(env, custom_tests={'CheckPKGConfig' : CheckPKGConfig,
                                    'CheckPKG' : CheckPKG })

def DoConfigure():
    if GetOption('clean'):
        return

    if not conf.CheckPKGConfig():
        Exit(1)

    if not conf.CheckPKG('scim'):
        Exit(1)

    if not conf.CheckPKG('sunpinyin-2.0'):
        Exit(1)

    if not conf.CheckPKG('gtk+-2.0'):
        Exit(1)
    
    env = conf.Finish()
    env.ParseConfig('pkg-config scim sunpinyin-2.0 gtk+-2.0 --libs --cflags')

DoConfigure()

scim_icondir = os.popen('pkg-config scim   --variable=icondir').readlines()[0].rstrip()
scim_moduledir =os.popen('pkg-config scim   --variable=moduledir').readlines()[0].rstrip()

lib_sunpinyin_imengine_setup_target  = "sunpinyin_imengine_setup"
lib_sunpinyin_imengine_setup_sources = ["src/sunpinyin_imengine_setup.cpp"]
env.SharedLibrary(target = lib_sunpinyin_imengine_setup_target, source = lib_sunpinyin_imengine_setup_sources)

lib_sunpinyin_imengine_target  = "sunpinyin_imengine"
lib_sunpinyin_imengine_sources = ['src/imi_scimwin.cpp',
                                  'src/sunpinyin_imengine.cpp',
                                  'src/sunpinyin_lookup_table.cpp',
                                  'src/sunpinyin_utils.cpp']
env.SharedLibrary(target = lib_sunpinyin_imengine_target, source = lib_sunpinyin_imengine_sources)
os.system('chmod 0644 ' + lib_sunpinyin_imengine_setup_target + '.so')
os.system('chmod 0644 ' + lib_sunpinyin_imengine_target + '.so')
for locale in locales:
    mo = 'po/%s.mo' % (locale,)
    env.Command(mo, [], 'msgfmt po/%s.po -o %s' % (locale, mo))

#
#==============================install================================
#
def DoInstall():

    os.system('chmod 0644 ' + lib_sunpinyin_imengine_setup_target + '.so')
    os.system('chmod 0644 ' + lib_sunpinyin_imengine_target + '.so')
        
    icons_target = env.Install(scim_icondir,
                               ['data/sunpinyin_logo.png'])

    imengine_target = env.Install(scim_moduledir + '/IMEngine',
                                   'sunpinyin_imengine.so')

    imsetup_target = env.Install(scim_moduledir + '/SetupUI',
                                   'sunpinyin_imengine_setup.so')

    locale_targets = []
    for locale in locales:
        path = env['DATADIR'] + '/locale/%s/LC_MESSAGES/%s.mo' % \
            (locale, gettext_package)
        locale_targets.append(env.InstallAs(path, 'po/%s.mo' % (locale,)))

    env.Alias('install-libexec', [imengine_target, imsetup_target])
    env.Alias('install-data', [icons_target])
    env.Alias('install-locale', locale_targets)

DoInstall()
env.Alias('install', ['install-libexec', 'install-data', 'install-locale'])
