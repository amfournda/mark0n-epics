# Reference
<!-- DO NOT EDIT: This document was generated by Puppet Strings -->

## Table of Contents

**Classes**

_Public Classes_

* [`epics`](#epics): Global configuration for IOCs
* [`epics::carepeater`](#epicscarepeater): Install and run the EPICS Channel Access Repeater
* [`epics::catools`](#epicscatools): Install Channel Access command line tools.
* [`epics::ioc::software`](#epicsiocsoftware): Install software needed to build and run EPICS IOCs.

_Private Classes_

* `epics::ioc::telnet`: Install tools to connect to procServ using TCP.
* `epics::ioc::unix_domain_socket`: Install tools to connect to procServ using Unix domain sockets.

**Defined types**

* [`epics::ioc`](#epicsioc): Manage an IOC instance.

## Classes

### epics

This class takes care of all system-wide tasks which are needed in order to
run a soft IOC. It installs required software and prepares machine-global
directories and configuration files.

#### Parameters

The following parameters are available in the `epics` class.

##### `iocbase`

Data type: `Stdlib::Absolutepath`

All IOC directories are expected to be located in a central directory. This
parameter specifies the path to this base directory. Defaults to
'/usr/local/lib/iocapps'.

Note: Keeping all IOC directories in a central place is required to maintain
compatibility with [sysv-rc-softioc](https://github.com/epicsdeb/sysv-rc-softioc).
This restriction might be dropped in the future.

##### `owner`

Data type: `String`

Owner of files/directories shared by all IOC instances (like the log
directory). Defaults to 'root'.

##### `group`

Data type: `String`

Group of files/directories shared by all IOC instances (like the log
directory). IOCs are also running under this group. Defaults to 'softioc'.

##### `gid`

Data type: `Optional[Integer]`

Define the group id of the group the IOCs are run as. The gid will be picked
automatically if this option is not specified.

Default value: `undef`

### epics::carepeater

Running the Channel Access Repeater is often useful on workstations and other
computers running multiple Channel Access clients. It forwards beacons to
multiple clients which allows all clients running on the machine to be
notified when IOCs are started or stopped. This often leads to faster
reconnects after IOC restarts. See the
[Channel Access Reference Manual](https://epics.anl.gov/base/R7-0/3-docs/CAref.html#Repeater)
for details.

In some cases installing the package containing the Channel Access Repeater
executable might be sufficient to start the Channel Access Repeater service.
However, this class provides more fine-grained control allowing users to run
the service on a custom port or under a different user. It can also ensure the
service is actually running (e.g. if a sysadmin stops it and forgets to start
it after the maintenance work is finished).

#### Examples

##### Ensure Channel Access Repeater is running

```puppet
include epics::carepeater
```

##### Ensure Channel Access Repeater is not running

```puppet
class { 'epics::carepeater':
  ensure => 'stopped',
  enable => false,
}
```

##### Ensure Channel Access Repeater is running with custom port and user

```puppet
class { 'epics::carepeater':
  port => 5077,
  user => 'epics',
}
```

#### Parameters

The following parameters are available in the `epics::carepeater` class.

##### `ensure`

Data type: `Stdlib::Ensure::Service`

Specifies whether the Channel Access Repeater service should be running.
Valid values are 'running', 'stopped'. Defaults to 'running'.

##### `enable`

Data type: `Boolean`

Whether the Channel Access Repeater service should be enabled. This
determines if the service is started on system boot. Valid values are true,
false. Defaults to true.

##### `executable`

Data type: `String`

Channel Access Repeater executable. Defaults to '/usr/bin/caRepeater'.

##### `port`

Data type: `Stdlib::Port`

Port that the Channel Access Repeater will listen on. This is setting the
value of the `EPICS_CA_REPEATER_PORT` environment variable. Defaults to
5065.

##### `dropin_file_ensure`

Data type: `Enum['present', 'absent', 'file']`

EPICS Base comes with a systemd service file that allows Channel Access
Repeater to be started. However, by itself it doesn't allow its
configuration to be tweaked (e.g. custom port, user name etc.). This class
thus augments the systemd service file that comes with EPICS with a drop-in
file allowing for additional configuration. This parameter controls whether
this drop-in file should exist or not. Please refer to the
[camptocamp/systemd documentation](https://forge.puppet.com/camptocamp/systemd#drop-in-files)
for details. Defaults to 'present'.

##### `user`

Data type: `String`

User that the Channel Access Repeater service will run as. Defaults to
'nobody'.

### epics::catools

Installs the Channel Access command line tools provided by EPICS Base
(caget, cainfo, camonitor, caput, caRepeater, casw).

#### Examples

##### 

```puppet
include epics::catools
```

#### Parameters

The following parameters are available in the `epics::catools` class.

##### `ensure`

Data type: `String`

What state the package should be in. Valid values include 'installed',
'latest' as well as a version number of the package. See the
[documentation of resource type 'package'](https://puppet.com/docs/puppet/latest/types/package.html#package-attribute-ensure)
for details.

### epics::ioc::software

This class installs software needed to build and run an EPICS IOC. If IOCs
are managed by epics::ioc this class is instantiated automatically. You might
want to include this class directly if your IOCs are managed by other means.

#### Parameters

The following parameters are available in the `epics::ioc::software` class.

##### `ensure_build_essential`

Data type: `String`

What state the 'build-essential' package should be in. Valid values include
'installed', 'latest' as well as a version number of the package. See the
[documentation of resource type 'package'](https://puppet.com/docs/puppet/latest/types/package.html#package-attribute-ensure)
for details.

##### `ensure_epics_dev`

Data type: `String`

What state the 'epics-dev' package should be in. Valid values include
'installed', 'latest' as well as a version number of the package. See the
[documentation of resource type 'package'](https://puppet.com/docs/puppet/latest/types/package.html#package-attribute-ensure)
for details.

##### `ensure_procserv`

Data type: `String`

What state the 'procserv' package should be in. Valid values include
'installed', 'latest' as well as a version number of the package. See the
[documentation of resource type 'package'](https://puppet.com/docs/puppet/latest/types/package.html#package-attribute-ensure)
for details.

##### `ensure_sysv_rc_softioc`

Data type: `String`

What state the 'sysv-rc-softioc' package should be in. Valid values include
'installed', 'latest' as well as a version number of the package. See the
[documentation of resource type 'package'](https://puppet.com/docs/puppet/latest/types/package.html#package-attribute-ensure)
for details. On machines using other service providers like systemd this
parameter is ignored.

## Defined types

### epics::ioc

This type configures an IOC instance. It creates configuration files,
populates them with the correct values, builds the IOC code (if desired) and
ensures the IOC instance can be started as a service. The top-level IOC
directory of the IOC is expected to be $iocbase/<ioc_name> where <ioc_name>
is the title specified when instantiating the 'epics::ioc' resource.

IOCs are run in [procServ](https://github.com/ralphlange/procServ) to ease
maintenance. They are started as a system service. On systems that use systemd
as init system a systemd service file will be created for each IOC process. On
systems using System-V-style init scripts this module relies on
sysv-rc-softioc for creating the init scripts. It is possible to transition
from one init system to another without modifying any Puppet code (just make
sure to run Puppet after rebooting the machine to give the module the
opportunity to make the required adjustments).

In contrast to libraries and regular applications which are installed from
packages, IOCs applications are build on the target machine. This allows IOC
engineers to fix problems in the EPICS run-time database quickly in the field
without waiting for a potentially long-running CI pipeline. Of course this
power comes with the responsibility to push these changes to the version
control system before they are lost due to a broken drive.

### Environment Variables
Some parameters of this class result in environment variables being set.
Please refer to the following table for a list:

<table>
<thead>
<tr>
<th>Parameter</th>
<th>Sets Environment Variable</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>ca_addr_list</code></td>
<td><code>EPICS_CA_ADDR_LIST</code></td>
</tr>
<tr>
<td><code>ca_auto_addr_list</code></td>
<td><code>EPICS_CA_AUTO_ADDR_LIST</code></td>
</tr>
<tr>
<td><code>ca_max_array_bytes</code></td>
<td><code>EPICS_CA_MAX_ARRAY_BYTES</code></td>
</tr>
<tr>
<td><code>log_port</code></td>
<td><code>EPICS_IOC_LOG_PORT</code></td>
</tr>
<tr>
<td><code>log_server</code></td>
<td><code>EPICS_IOC_LOG_INET</code></td>
</tr>
<tr>
<td><code>ca_sec_file</code></td>
<td><code>EPICS_CA_SEC_FILE</code></td>
</tr>
</tbody>
</table>
Environment variables that are not on this list can be set using the
'env_vars' parameter.

#### Examples

##### Simple

```puppet
file { '/usr/local/lib/iocapps/vacuumioc':
  source  => 'puppet:///vacuumioc',
  recurse => true,
}

epics::ioc { 'vacuumioc':
  subscribe => File['/usr/local/lib/iocapps/vacuumioc'],
}
```

##### Two IOC instances on one machine

```puppet
package { 'epics-autosave-dev':
  ensure => 'latest',
  tag    => 'epics_ioc_pkg',
}

# Stick with v4.38 until we found time to test the latest version:
package { 'epics-asyn-dev':
  ensure => '4.38',
  tag    => 'epics_ioc_pkg',
}

package { 'epics-stream-dev':
  ensure => 'latest',
  tag    => 'epics_ioc_pkg',
}

vcsrepo { '/usr/local/lib/iocapps/llrf':
  ensure   => 'latest',
  source   => 'git://example.com/llrfioc.git',
  provider => 'git',
  owner    => 'softioc-llrf',
  group    => 'softioc',
}

vcsrepo { '/usr/local/lib/iocapps/cryo':
  ensure   => 'latest',
  source   => 'git://example.com/cryo.git',
  provider => 'git',
  owner    => 'softioc-cryo',
  group    => 'softioc',
}

# Settings for all IOC instances (consider putting this into a facility-wide
# profile that is applied to all IOC machines):
Epics::Ioc {
  manage_autosave_dir => true,
  autosave_base_dir   => '/mnt/autosave',
  log_server          => 'log.example.com',
}

# For this IOC we always want the latest and greatest so we let Puppet
# rebuild and restart it whenever new IOC code is pulled from the Git repo
# or when a new version of a support package is installed:
epics::ioc { 'llrf':
  console_port => 4051,
  subscribe    => [
    Package['epics-autosave-dev'],          # rebuild and restart when package is updated
    Package['epics-asyn-dev'],              # rebuild and restart when package is updated
    Vcsrepo['/usr/local/lib/iocapps/llrf'], # rebuild and restart when package is updated
  ],
}

# For this IOC we can't afford any unplanned downtime so we rebuild but
# do not automatically restart this IOC. Rebuilding the IOC ensures that
# even in case the IOC crashes we always have a binary that is ready to run
# (we don't want to end up starting an IOC executable that has been linked
# against an old version of a library which has been removed from the system).
epics::ioc { 'cryo':
  console_port     => 4052,
  auto_restart_ioc => false,
  subscribe        => [
    Package['epics-asyn-dev'],
    Package['epics-stream-dev'],
    Vcsrepo['/usr/local/lib/iocapps/cryo'],
  ],
}
```

#### Parameters

The following parameters are available in the `epics::ioc` defined type.

##### `ensure`

Data type: `Optional[Stdlib::Ensure::Service]`

Ensures the IOC service is running/stopped. Valid values are 'running',
'stopped', and undef. If not specified Puppet will not start/stop the IOC
service.

Default value: `undef`

##### `enable`

Data type: `Optional[Boolean]`

Whether the IOC service should be enabled to start at boot. Valid values are
true, false, and undef. If not specified (undefined) Puppet will not
enable/disable the IOC service.

Default value: `undef`

##### `manage_autosave_dir`

Data type: `Boolean`

Whether to automatically populate the `AUTOSAVE_DIR` environment variable.
Valid values are true and false. If true the specified directory will be
created (users need to ensure the parent directory exists) and permissions
will be set appropriately. The `AUTOSAVE_DIR` environment variable will be
set to <autosave_base_dir>/softioc-<ioc_name>. Also see the
'autosave_base_dir' parameter.

Default value: lookup('epics::ioc::manage_autosave_dir', Boolean)

##### `auto_restart_ioc`

Data type: `Boolean`

Whether to restart the IOC after recompiling. If set to true the IOC will
be restarted automatically after recompiling the source code (see
`run_make`). This ensures the latest code is being used. Defaults to true.

Default value: lookup('epics::ioc::auto_restart_ioc', Boolean)

##### `autosave_base_dir`

Data type: `String`

The path to the base directory for the EPICS 'autosave' module. Defaults to
'/var/lib'.

Default value: lookup('epics::ioc::autosave_base_dir', String)

##### `bootdir`

Data type: `String`

Path to the directory containing the IOC start script. This path is
relative to the IOC's top directory (<iocbase>/<ioc_name>). Defaults to
'iocBoot/ioc${{HOST_ARCH}}'.

Default value: lookup('epics::ioc::bootdir', String)

##### `ca_addr_list`

Data type: `Optional[String]`

Allows to configure the `EPICS_CA_ADDR_LIST` environment variable for the
IOC. Defaults to undefined (environment variable not set).

Default value: `undef`

##### `ca_auto_addr_list`

Data type: `Optional[Boolean]`

Allows to configure the `EPICS_CA_AUTO_ADDR_LIST` environment variable for
the IOC. Valid values are true and false. Defaults to undefined (environment
variable not set).

Default value: `undef`

##### `ca_max_array_bytes`

Data type: `Optional[Integer]`

Allows to configure the `EPICS_CA_MAX_ARRAY_BYTES` environment variable for
the IOC. Defaults to undefined (environment variable not set).

Default value: `undef`

##### `startscript`

Data type: `String`

Base file name of the IOC start script. Defaults to 'st.cmd'.

Default value: lookup('epics::ioc::startscript', String)

##### `enable_console_port`

Data type: `Boolean`

If set to true (the default) procServ will listen on the port specified by
'console_port' for connections to the IOC shell. If this flag is true for at
least one IOC telnet is being installed.

Default value: lookup('epics::ioc::enable_console_port', Boolean)

##### `console_port`

Data type: `Stdlib::Port`

Specify the port number procServ will listen on for connections to the IOC
shell. You can connect to the IOC shell using
`telnet localhost <portnumber>`. Defaults to 4051.

Note that access is not possible if 'enable_console_port' is set to false.

Default value: lookup('epics::ioc::console_port', Stdlib::Port)

##### `enable_unix_domain_socket`

Data type: `Boolean`

If set to true (the default) procServ will create a unix domain socket for
connections to the IOC shell. If this flag is true for at least one IOC the
BSD version of netcat is installed.

Default value: lookup('epics::ioc::enable_unix_domain_socket', Boolean)

##### `unix_domain_socket`

Data type: `String`

Specify the Unix domain socket file procServ will create for connections
to the IOC shell. The file name has to be specified relative to the run-time
directory ('/run'). You can connect to the IOC shell using
`nc -U <unix_domain_socket>`. Defaults to
'softioc-<ioc_name>/procServ.sock'.

Note that the unix domain socket will not be created if
'enable_unix_domain_socket' is set to false.

Default value: "softioc-${name}/procServ.sock"

##### `coresize`

Data type: `Integer`

The maximum size (in Bytes) of a core file that will be written in case the
IOC crashes. Defaults to 10000000.

Default value: lookup('epics::ioc::coresize', Integer)

##### `cfg_append`

Data type: `Array[String]`

Allows to set additional variables in the IOC's config file in '/etc/iocs/'.
This is not used on machines that use systemd.

Default value: lookup('epics::ioc::cfg_append', Array[String])

##### `env_vars`

Data type: `Hash[String, Data, default, default]`

Specify a hash of environment variables that shall be passed to the IOC.
Defaults to {}.

Default value: lookup('epics::ioc::env_vars', Hash[String, Data, default, default])

##### `log_port`

Data type: `Stdlib::Port`

Allows to configure the `EPICS_IOC_LOG_PORT` environment variable for the
IOC. Defaults to 7004 (the default port used by iocLogServer).

Default value: lookup('epics::ioc::log_port', Stdlib::Port)

##### `log_server`

Data type: `Optional[Stdlib::Host]`

Allows to configure the `EPICS_IOC_LOG_INET` environment variable for the
IOC. Defaults to undef (environment variable not set).

Default value: lookup('epics::ioc::log_server', { 'value_type' => Optional[Stdlib::Host], 'default_value' => undef })

##### `ca_sec_file`

Data type: `Optional[String]`

Allows to configure the `EPICS_CA_SEC_FILE` environment variable for the
IOC. Defaults to undef (environment variable not set). Used this with
`asSetFilename(${EPICS_CA_SEC_FILE})` in the IOC start-up script.

Default value: `undef`

##### `procserv_log_file`

Data type: `Stdlib::Absolutepath`

The log file that procServ uses to log activity on the IOC shell. Defaults
to '/var/log/softioc-<ioc_name>/procServ.log'.

Default value: "/var/log/softioc-${name}/procServ.log"

##### `logrotate_compress`

Data type: `Boolean`

Whether to compress the IOC's log files when rotating them. Defaults to
true.

Default value: lookup('epics::ioc::logrotate_compress', Boolean)

##### `logrotate_rotate`

Data type: `Integer`

The time in days after which a the log file for the procServ log will be
rotated. Defaults to 30.

Default value: lookup('epics::ioc::logrotate_rotate', Integer)

##### `logrotate_size`

Data type: `String`

If the log file for the procServ log reaches this size the IOC log will be
rotated. Defaults to '10M'.

Default value: lookup('epics::ioc::logrotate_size', String)

##### `run_make`

Data type: `Boolean`

Whether to compile the IOC when its source code changes. If set to true the
code in the IOC directory will be compiled automatically by running `make`.
This ensures the IOC executable is up to date. Defaults to true.

Note: This module runs `make --question` to determine whether it needs to
rebuild the code by running make. Some Makefiles run a command on every
invocation. This can cause `make --question` to always return a non-zero
exit code. Beware that this will cause Puppet to rebuild your IOC on every
run. Depending on the 'auto_restart_ioc' setting this might also cause the
IOC to restart on every Puppet run! Please verify that your Makefiles are
behaving correctly to prevent surprises.

Default value: lookup('epics::ioc::run_make', Boolean)

##### `run_make_after_pkg_update`

Data type: `Boolean`

If this option is activated the IOC will be recompiled whenever a 'package'
resource tagged as 'epics_ioc_pkg' is refreshed. This can be used to rebuild
IOCs when facility-wide installed EPICS modules like autosave are being
updated. Defaults to true.

Default value: lookup('epics::ioc::run_make_after_pkg_update', Boolean)

##### `uid`

Data type: `Optional[Integer]`

Defines the system user id the IOC process is supposed to run as. The
corresponding user is created automatically. If this is left undefined an
arbitrary user id will be picked. This argument is only used if
'manage_user' is true.

Default value: `undef`

##### `username`

Data type: `String`

The user name the IOC will run as. By default 'softioc-<ioc_name>' is being
used.

Default value: lookup('epics::ioc::username', { 'default_value' => "softioc-${name}" })

##### `manage_user`

Data type: `Boolean`

Whether to create the user account the IOC is running as. Set to false to
use a user account that is managed by Puppet code outside of this module.
Disable if you want multiple IOCs to share the same user account. Defaults
to true.

Default value: lookup('epics::ioc::manage_user', Boolean)

##### `systemd_after`

Data type: `Array[String]`

Ensures the IOC service is started after the specified systemd units have
been activated. Please specify an array of strings. Defaults to
['network.target']. This parameter is ignored on systems that are not using
systemd.

Note: This enforces only the correct order. It does not cause the specified
targets to be activated. Also see 'systemd_requires'. See the
[systemd documentation](https://www.freedesktop.org/software/systemd/man/systemd.unit.html#After=)
for details.

Default value: lookup('epics::ioc::systemd_after', Array[String])

##### `systemd_requires`

Data type: `Array[String]`

Ensures the specified systemd units are activated when this IOC is started.
Defaults to ['network.target']. This parameter is ignored on systems that are
not using systemd.

Note: This only ensures that the required services are started. That
generally means that systemd starts them in parallel to the IOC service.
Use this parameter together with 'systemd_after' to ensure they are started
before the IOC is started. See the
[systemd documentation](https://www.freedesktop.org/software/systemd/man/systemd.unit.html#Requires=)
for details.

Default value: lookup('epics::ioc::systemd_requires', Array[String])

##### `systemd_requires_mounts_for`

Data type: `Array[String]`

Ensures the specified paths are accessible (e.g. the corresponding file
systems are mounted) when this IOC is started. Specify an array of strings.
Defaults to []. This parameter is ignored on systems that are not using
systemd.

See the
[systemd documentation](https://www.freedesktop.org/software/systemd/man/systemd.unit.html#RequiresMountsFor=)
for details.

Default value: lookup('epics::ioc::systemd_requires_mounts_for', Array[String])

##### `systemd_wants`

Data type: `Array[String]`

Tries to start the specified systemd units when this IOC is started.
Defaults to []. This parameter is ignored on systems that are not using
systemd.

Note: systemd will only _try_ to start the services specified here when the
IOC service is started. That generally means that systemd starts them in
parallel to the IOC service. Use this parameter together with
'systemd_after' to ensure systemd has tried starting them _before_ the IOC
is started. See the
[systemd documentation](https://www.freedesktop.org/software/systemd/man/systemd.unit.html#Wants=)
for details.

Default value: lookup('epics::ioc::systemd_wants', Array[String])
