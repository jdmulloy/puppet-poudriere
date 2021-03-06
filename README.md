# Puppet-poudriere

[![Puppet Forge](https://img.shields.io/puppetforge/v/zleslie/poudriere.svg)]() [![Build Status](https://travis-ci.org/xaque208/puppet-poudriere.svg?branch=master)](https://travis-ci.org/xaque208/puppet-poudriere)

Manage the FreeBSD PkgNG build system with Puppet.

## Simple Implementation

```Puppet
poudriere::env { "90amd64":
  makeopts => [
    "WITH_PKGNG=yes",
    "OPTIONS_SET+= SASL",
    "OPTIONS_SET+= TLS",
    "WITH_IPV6=TRUE",
    "WITH_SSL=YES",
  ]
}

nginx::vhost { "build.${domain}":
  port      => 80,
  vhostroot => '/usr/local/poudriere/data/packages',
  autoindex => true,
}

@@pkgng::repo { "${::fqdn}-90amd64":
  release     => '9.0-RELEASE',
  mirror_type => 'http',
  repopath    => '/90amd64-default/Latest/',
}
```

## Changing default settings

The `poudriere` class has some default parameters for global settings, such as the ZFS pool, root filesystem, and the FreeBSD mirror site to use (see `init.pp` for details). To change these defaults, declare the `poudriere` class with your own parameters.

If you do not want to use ZFS, you can use:

```Puppet
class { 'poudriere':
  zpool => false,
}
```

## Using port-specific make options

You can pass port-specific make options as a hash in `pkg_makeopts`. For instance, if you want to build Apache 2.4 with the PHP 5.5 module, you can do the following:

```Puppet
poudriere::env { "90amd64":
  makeopts => [
    'WITH_PKGNG=yes',
    'APACHE_VERSION=24',
    'APACHE_PORT=www/apache24',
  ],
  pkgs => [
    'www/apache24',
    'lang/php55',
  ],
  pkg_makeopts => {
    'lang/php55' => ['OPTIONS_SET+=APACHE'],
  },
}
```

Alternatively, a file containing make options for both global and port-specific use could be defined by setting `makefile`.

## Using port-specific build options

Ports often allow for enabling or disabling support for certain features. Such options can be manually set by issueing `poudriere options cat/port` but can also be defined in puppet by setting `pkg_optsdir`. This should point to a directory with files such as could be found in `/usr/local/etc/poudriere.d/${jail}-options/`:

```Puppet
poudriere::env { "90amd64":
  pkg_optsdir => 'puppet:///path/to/dir/',
}
```

## Managing seperate portstrees

The `default` portstreee is no longer configuered by this module.  You should create this yourself like so:

```Puppet
  poudriere::portstree { 'default': }
```

```yaml
poudriere::portstrees:
  default:
```

By default environments use a portstree named `default`, which you should create as per the instructions above; however each build environment can be told which portstree to use:

```Puppet
poudriere::portstree { "custom-ports":
  cron_enable  => false,
  fetch_method => 'portsnap',
}

poudriere::env { "custom-build":
  portstree => 'custom-ports',
}
```

```yaml
poudriere::portstrees:
  default:
  custom-ports:
    cron_enabled: false
    fetch_method: portsnap
poudriere::environments:
  custom-build:
    portstree: custom-ports
```
