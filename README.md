# Puppet module springboot

#### Table of Contents

1. [Overview](#overview)
2. [Module Description - What the module does and why it is useful](#module-description)
3. [Setup - The basics of getting started with springboot](#setup)
    * [What springboot affects](#what-springboot-affects)
    * [Setup requirements](#setup-requirements)
    * [Beginning with springboot](#beginning-with-springboot)
4. [Usage - Configuration options and additional functionality](#usage)
5. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
5. [Limitations - OS compatibility, etc.](#limitations)
6. [Development - Guide for contributing to the module](#development)
7. [Disclaimer](#disclaimer)

## Overview

This Puppet module implements the `springboot` class and the `springboot::app` defined type.

The class intents to create OS structures (packages, SELinux modules, systemd service templates,
filesystems, group/user, ...) in order to prepare the OS to host Springboot application(s).
The defined type intents to implement the deployment of a Springboot app instance on top of a
prepared OS.

This Puppet module is part of the overall HubertQC/*springboot GitHub repositories family:
   https://github.com/hubertqc/selinux_springboot

## Module Description

The `hubertqc-springboot` Puppet module implements the `springboot` Puppet class
to create the base OS structures to host one ore more Springboot applications.

These OS structures are:

* OS user and group,
* LVM LV and filesystems (optional),
* Directory trees
* systemd units
* packages

The `hubertqc-springboot` Puppet module also implements the `springboot::app`
Puppet defined-type to deploy a specific Springboot application on top of the
OS structures.

## Setup

### What `springboot` affects

#### OS user and group

Group `springboot`and  user `springboot` will be created.

#### Packages

Packages `springboot-selinux`, `springboot-selinux-devel` will be installed, thus installing the
`springboot` SELinux policy module (see https://github.com/hubertqc/selinux_springboot)

The `springboot-systemd` package will also be installed, thus creating the `springboot.target`
and `springboot@.service` systemd units.

#### Files and Directories

This Puppet module will create the following directory trees, optionally on dedicated filesystems,
and partially manage their contents:

* `/opt/springboot`: location of static/permanent files (not data),
* `/srv/springboot`, `/var/lib/springboot`: location of working files (permament/temporary data, ...),
* `/var/tmp/springboot`: location of transient files,
* `/var/log/springboot`: location of log files,
* `/etc/springboot`: location of system-wide configuration files.

#### Logical volumes and filesystems

When asked to, this Puppet module will create the Springboot directory trees on dedicated LVM devices
and filesystems, using the LVM groups supplied:

* `lv_opt_springboot`,
* `lv_srv_springboot`,
* `lv_varlib_springboot`,
* `lv_tmp_springboot`,
* `lv_log_springboot`.

### Setup Requirements **OPTIONAL**

This module requires the following Puppet modules to be available:
    - puppetlabs-stdlib, version 1.0.0 or later,
    - puppetlabs-lvm, version 1.1.0 or later,
    - hubertqc-mountpoint, version 1.0.0 or later,
    - voxpopuli-selinux, version 3.0.0 or later,

### Beginning with springboot

The very basic steps needed for a user to get the module up and running.

#### Preparing for your host to run the `springboot` class

#### Preparing for your host to run the `springboot::app` defined type

## Usage

Put the classes, types, and resources for customizing, configuring, and doing
the fancy stuff with your module here.

## Reference

### Class `springboot`

#### Parameters

* `springboot_uid` (Integer): UID number for the `springboot` OS user (default: `5000`),
* `springboot_gid` (Integer): GID number for the `springboot` OS group (default: `5000`),

* `enable_systemd_target` (Boolean): enable the `spingboot.target` systemd unit (default: `true`),

When the following parameters are `undef` (default value) no dedicated LV/FS is created:

* `opt_vg` (Optional[String]): name of the LVM group for the /opt related LV/FS,
* `srv_vg` (Optional[String]): name of the LVM group for the /srv and /var (except logs) related LV/FS,
* `log_vg` (Optional[String]): name of the LVM group for the /var/log/springboot related LV/FS,

The following parameters govern SELinux booleans for the `springboot` SELinux policy module:

* `allow_springboot_connectto_http` (Boolean, default `true`)
* `allow_springboot_connectto_self` (Boolean, default `false`)
* `allow_springboot_connectto_ldap` (Boolean, default `false`)
* `allow_springboot_connectto_smtp` (Boolean, default `false`)
* `allow_springboot_connectto_oracle` (Boolean, default `false`)
* `allow_springboot_connectto_mysql` (Boolean, default `false`)
* `allow_springboot_connectto_pgsql` (Boolean, default `false`)
* `allow_springboot_connectto_redis` (Boolean, default `false`)
* `allow_springboot_connectto_couchdb` (Boolean, default `false`)
* `allow_springboot_connectto_mongodb` (Boolean, default `false`)
* `allow_springboot_dynamic_libs` (Boolean, default `false`)
* `allow_springboot_purge_logs` (Boolean, default `false`)
* `allow_webadm_read_springboot_files` (Boolean, default `false`)
* `allow_sysadm_write_springboot_files` (Boolean, default `false`)
* `allow_sysadm_manage_springboot_auth_files` (Boolean, default `false`)

### Defined type `springboot::app`

Note: the resource name/title should not contain hyphens. If so, the directories and the service will be created
with all hyphens replaced with underscores.

#### Parameters

* `app_name` (String, default: `<resource title>`): Name for the Springboot application,

* `listen_port` (Stdlib::Port, no default value): TCP port the Springboot application will listnen on,

* `jar_distribution_uri` (Oprional[Stdlib::Filesource], default: `undef`): URI where Puppet can find the Springboot JAR file
* `jar_name` (String, default: `<resource name>.jar`): name of the JAR file,

* `deployment_user` (Optional[String], default: `undef`): additional deployment OS user,

* `java_version` (Integer, default: `11`): version of the Java JVM to run the Springboot application,
* `java_flavour` (Enum['openjdk','oracle'], default: `'openjdk'`): flavour of the Java JVM to run the Springboot application,

* `memory_max_heapsize_mb` (Integer, default: `256`): maximum heap size for the Sprinbgboot application JVM,
* `memory_min_heapsize_mb` (Optional[Integer], default: `undef`): minimum heap size for the Sprinbgboot application JVM,
* `memory_stacksize_kb` (Integer, default: `512`): stack size for the Sprinbgboot application JVM,
* `memory_hugepages` (Boolean, default: `false`): enable the JVM to use Linux memory huge pages,
* `java_args` (Optional[String], default: `undef`): additional arguments to the JVM,
* `jar_opts` (Optional[String], default: `undef`): additional options to the JAR application,
* `i18n_lang` (Optional[String], default: `undef`): i18n lang specification for the JVM environment,

The Following parameters govern the way Puppet manages the systemd service unit for the Springboot application:

* `manage_service` (Boolean, default: `true`): define a springboot@<application name>.service systemd unit,
* `service_started` (Boolean, default: `false`): make Puppet starts the Springboot application service unit,
* `service_enabled` (Boolean, default: `false`): make Puppet enable the Springboot application service unit,
* `startup_timeout_sec` (Integer[5,300], default: `10`): delay for the Springboot application to startup,
* `restart_app_upon_jar_change` (Boolean, default: `false`): make Puppet restart the Springboot application whenever the JAR file changes,

The following parameters govern the creation of dedicated LV/FS for the Springboot application:

* `dedicated_fs_log_vg` (Optional[String], default: `undef`): LVM group where the /var/log/springboot/<app> LV/FS will be created,
* `dedicated_fs_srv_vg` (Optional[String], default: `undef`): LVM group where the /srv/springboot/<app> LV/FS will be created,
* `dedicated_fs_opt_vg` (Optional[String], default: `undef`): LVM group where the /opt/springboot/<app> LV/FS will be created,
* `dedicated_fs_initial_log_size_mb` (Integer, default: `1024`),
* `dedicated_fs_initial_srv_size_mb` (Integer, default: `1024`),
* `dedicated_fs_initial_opt_size_mb` (Integer, default: `256`),

## Limitations

Works only on Red Hat Linux and Red Hat like distributions.

## Development

See https://github.com/hubertqc/puppet-springboot/CONTRIBUTING.md

## Disclaimer

The code of the this Puppet module is provided AS-IS. People and organisation
willing to use it must be fully aware that they are doing so at their own risks and
expenses.

The Author(s) of this Puppet module SHALL NOT be held liable nor accountable, in
 any way, of any malfunction or limitation of said module, nor of the resulting damage, of
 any kind, resulting, directly or indirectly, of the usage of this Puppet module.

It is strongly advised to always use the last version of the code, to check for the
compatibility of the platform where it is about to be deployed, to compile the module on
the target specific Linux distribution and version where it is intended to be used.

Finally, users should check regularly for updates.
