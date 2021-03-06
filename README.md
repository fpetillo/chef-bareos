Chef-Bareos Cookbook
====================

[![BuildStatus](https://travis-ci.org/sitle/chef-bareos.svg?branch=master)](https://travis-ci.org/sitle/chef-bareos)
[![Join the chat at https://gitter.im/EMSL-MSC/chef-bareos](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/EMSL-MSC/chef-bareos?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)  

This cookbook installs and configures backups based on [BAREOS](https://www.bareos.org/en/).

[Official BAREOS Documentation](http://doc.bareos.org/master/html/bareos-manual-main-reference.html).

## Supported Platforms:
 * Ubuntu 14.04 (plan to add 16.04 as soon as binary is released)
 * Debian 7 (8+ may or may not work, you'll need a repo basically)
 * CentOS 6+
 * RHEL 6+ (Assumed to work just as well as on CentOS)

## Supported Chef Versions:

 * Chef 11+

## Important Notable Attributes

### Repository
Assists with adding necessary sources for installing Bareos

| Attribute        | Default Value | Description
|------------------|---------------|------------
| ['bareos']['url'] | 'http://download.bareos.org/bareos/release' | Main installation URL
| ['bareos']['contrib_url'] | 'http://download.bareos.org/bareos/contrib' | Main contrib installation URL
| ['bareos']['version'] | '15.2' | Default Bareos Version

For other platform specific attributes please see default attributes file for more detail.

### Messages
Defines default admin e-mail address for service notifications and what messages to care about

| Attribute        | Default Value | Description
|------------------|---------------|------------
| ['bareos']['messages']['mail_to'] | "bareos@#{node['domain']}" | Default messages e-mail destination
| ['bareos']['messages']['default_messages'] | 'Standard' | Default client message capture level
| ['bareos']['messages']['default_admin_messages'] | 'all, !skipped, !restored' | Default server message capture level
### Database
Populates the Catalog resource in the main Director configuration

| Attribute        | Default Value | Description
|------------------|---------------|------------
| ['bareos']['database']['catalog_name'] | 'MyCatalog' | Default catalog name
| ['bareos']['database']['database_type'] | 'postgresql' | Default database installment indicator
| ['bareos']['database']['dbdriver'] | 'postgresql' | Config entry for type of database
| ['bareos']['database']['dbname'] | 'bareos' | Default database name
| ['bareos']['database']['dbuser'] | 'bareos' | Default database user
| ['bareos']['database']['dbpassword'] | *blank string* | Default gets generated by postgresql cookbook, unless specified here
| ['bareos']['database']['dbaddress'] | nil | This is often just the localhost, so this is only needed in certain cases

### Clients
Provides resources for the Catalog (Director configuration) and Filedaemon configurations/templates

| Attribute        | Default Value | Description
|------------------|---------------|------------
| ['bareos']['clients']['name'] | node['fqdn'] | Uses the node FQDN by default as prefix for filedaemon name
| ['bareos']['clients']['net_name'] | node['fqdn'] | DNS Network name used to resolve client
| ['bareos']['clients']['fd_port'] | 9102 | Default filedaemon port
| ['bareos']['clients']['max_concurrent_jobs'] | 20 | Default number of concurrent jobs
| ['bareos']['clients']['heartbeat_interval'] | 600 | Proven a useful default value, change as needed
| ['bareos']['clients']['client_search_query'] | 'roles:bareos_client' | Default search query to find Bareos clients
| ['bareos']['clients']['client_list'] | %w(node) | Useful if you need a list of hosts if running in solo mode
| ['bareos']['clients']['bootstrap_file'] | '/var/lib/bareos/%c.bsr' | Default bootstrap file structure/location
| ['bareos']['clients']['jobdef_default_messages'] | 'Standard' | Default value for setting the message level in a job definition to override the messages section
| ['bareos']['clients']['jobdef_default_fileset'] | "#{node['fqdn']}-Fileset" | Default naming convention for filesets
| ['bareos']['clients']['storage'] | 'File' | Default storage for new clients when added via search
| ['bareos']['clients']['sensitive_configs'] | true | Enables or disables sensitive resource configuration files

### Storage Daemon
Provides for a baseline Storage Daemon Config with configurable options

| Attribute        | Default Value | Description
|------------------|---------------|------------
| ['bareos']['storage']['name'] | node['fqdn'] | Uses FQDN for naming storages found via search
| ['bareos']['storage']['storage_search_query'] | 'roles:bareos_storage' | Default search query string for finding storage servers
| ['bareos']['storage']['sd_port'] | 9103 | Default Storage communication port
| ['bareos']['storage']['servers'] | %w(node) | List of storage servers you can use if using solo mode
| ['bareos']['storage']['max_concurrent_jobs'] | 20 | Default max number of concurrent storage daemon jobs
| ['bareos']['storage']['autochanger_enabled'] | false | Used to control autochanger support
| ['bareos']['storage']['sensitive_configs'] | true | Enables or disables sensitive resource configuration files

### Director
Provides standard variables for a typical Director configuration

| Attribute        | Default Value | Description
|------------------|---------------|------------
| ['bareos']['director']['name'] | node['fqdn'] | Uses FQDN for director naming
| ['bareos']['director']['net_name'] | node['fqdn'] | Uses FQDN for DNS resolution
| ['bareos']['director']['dir_search_query'] | 'roles:bareos_director' | Default search string to find bareos directors
| ['bareos']['director']['dir_port'] | 9101 | Default director communication port
| ['bareos']['director']['dir_max_concurrent_jobs'] | 20 | Default max allowable jobs running
| ['bareos']['director']['servers'] | %w(node) | List of directors if running in solo mode
| ['bareos']['director']['console_commandacl'] | 'status, .status' | Default ACL for console commands
| ['bareos']['director']['heartbeat_interval'] | 600 | Proven useful as a default network timeout for communication to director
| ['bareos']['director']['catalog_jobdef'] | 'default-catalog-def' | Default name for the Catalog Backup Jobdef name
| ['bareos']['director']['conf']['help']['Example Block'] | '# You can put extra configs here.' | Area where you can add any number of possible things to expand your configs
| ['bareos']['director']['config_change_notify'] | 'restart' | Default action when director config changes (restart/reload)
| ['bareos']['director']['sensitive_configs'] | true | Enables or disables sensitive resource configuration files


### Subscription Management (Director)
Provides a system counter method if you have a paid service subscription

| Attribute        | Default Value | Description
|------------------|---------------|------------
| ['bareos']['director']['dir_subscription'] | nil | Required if you have a support contract/licensed installation (activates if not nil)
| ['bareos']['director']['dir_subs'] | nil | Max number of subs you have signed up for

### Workstation
Determines if you want to use FQDN or some other way of defining hosts in your management workstation deployment

| Attribute        | Default Value | Description
|------------------|---------------|------------
| ['bareos']['workstation']['name'] | node['fqdn'] | Used to determine header information for bconsole/bat configs
| ['bareos']['workstation']['sensitive_configs'] | true | Enables or disables sensitive resource configuration files

### Graphite Plugin
A new plugin that will send statistics to a graphite server which can then be used in various ways.

| Attribute        | Default Value | Description
|------------------|---------------|------------
| ['bareos']['plugins']['graphite']['plugin_path'] | '/usr/sbin' | Default location for the plugin that runs in a defined cron job
| ['bareos']['plugins']['graphite']['config_path'] | '/etc/bareos' | Default directory for the plugin config
| ['bareos']['plugins']['graphite']['search_query'] | 'roles:bareos_director' | Default search string to populate the director name
| ['bareos']['plugins']['graphite']['server'] | 'graphite' | Placeholder string for the graphite server DNS name
| ['bareos']['plugins']['graphite']['graphite_port'] | '2003' | Default graphite communication port
| ['bareos']['plugins']['graphite']['graphite_data_prefix'] | 'bareos.' | Default prefix for graphite data
| ['bareos']['plugins']['graphite']['graphite_plugin_src_url'] | See attributes file | Default URL to the plugin
| ['bareos']['plugins']['graphite']['cron_job'] | nil | Activates a general minutely cronjob if defined other than nil
| ['bareos']['plugins']['sensitive_configs'] | true | Enables or disables sensitive resource configuration files

## Recipes

### default
Installs the Bareos repos (via the repo recipe) and the client filedaemon (via the client recipe). Please NOTE, although it will install these parts, the director will not pick up on the client unless you (by default) create and attach a bareos_client role to hosts you wish to configure automatically. You can also add the host to the director in the unmanaged-host key value hashes.

### client
Installs the Bareos filedaemon and creates a config file that is linked to available directors on chef server.
You may also feed directors to the config via attributes if running in solo mode.

### repo
Installs base Bareos repo as well as the Bareos Contrib repo.

### database
Installs whichever database is desired per attributes (PostgreSQL/MySQL), installs Bareos database packages and creates the bareos database and user for you. Should also set the database password by default. You may need to recover this from the attributes or set a new one via vault via wrapper recipe.

### server
Installs necessary Bareos server packages and sets up base configs necessary for server to start. Also creates the config directory (bareos-dir.d) so you can drop whatever outside config files into place and have them get automatically included in your setup.

### storage
Installs necessary Bareos storage packages and sets up a default file storage for you to start backing stuff up to right away (configured for ~250GB spread over 25 10GB volumes).

### autochanger
This bit will setup an autochanger based on a pretty straight forward has table. Tested with IBM TS3500 Tape Library with 10 Frames and 16 Tape drives.

### workstation
Installs the bconsole utility. There are future plans to create a recipe to install bat (Bareos Administration Tool) and the Bareos Web UI.

### graphite_plugin
Installs a Bareos graphite plugin, configuration file, necessary python packages, and a cronjob to gather statistics periodically and forward them to an available graphite server.

## Searchable Roles (Used by default)

### bareos\_client
This example shows how the ```bareos_client``` role can both install the Bareos client side software and when searched against via the server recipe, will add itself to the bareos-dir (Bareos director) configuration and setup a default set of jobs for a client.

```
{
  "name": "bareos_client",
  "description": "Example Role for Bareos clients using the chef-bareos Cookbook, used in searches, throws down sources for installs",
  "json_class": "Chef::Role",
  "default_attributes": {
  },
  "override_attributes": {
  },
  "chef_type": "role",
  "run_list": [
    "recipe[chef-bareos]"
  ],
  "env_run_lists": {
  }
}
```

### bareos\_storage
This example shows a ```bareos_storage``` role which will create a Bareos storage-daemon host. It will install the necessary packages and lay down configuration files you can populate with any number of key value hash tables. You should be able to install this independent of the director(s), please file a ticket if this doesn't work as expected.

```
{
  "name": "bareos_storage",
  "description": "Example Role for a Bareos storage",
  "json_class": "Chef::Role",
  "default_attributes": {
  },
  "override_attributes": {
  },
  "chef_type": "role",
  "run_list": [
    "recipe[chef-bareos::storage]"
  ],
  "env_run_lists": {
  }
}
```

### bareos\_director
This role example will install an all-in-one Bareos director server. This includes a "first" client, bareos database, bareos storage, and deploy bconsole to interact with the director.

This will also allow clients to populate their filedaemon config via defined search string.

You'll need to run ```chef-client``` on the director after a client gets configured so the director can add and generate the appropriate client related configs.

You can populate the ```['bareos']['clients']['unmanaged']``` hash table space with any number of client related configuration lines if you have hosts you either don't plan to search for or want to do custom configurations for.

```
{
  "name": "bareos_director",
  "description": "Example Role for a Bareos director",
  "json_class": "Chef::Role",
  "default_attributes": {
  },
  "override_attributes": {
  },
  "chef_type": "role",
  "run_list": [
    "role[bareos_client]",
    "recipe[chef-bareos::database]",
    "recipe[chef-bareos::server]",
    "recipe[chef-bareos::workstation]"
  ],
  "env_run_lists": {
  }
}
```

## Example customizable key value hash template configurations
These are the preset default hashes to get a baseline configuration on a new bareos server. You can manipulate these as you see fit via recipe logic or searches or whatever you want. These will at least get you going.

### clients
```
# Default Client Config populated via search
default['bareos']['clients']['conf'] = {
  'FDPort' => '9102',
  'File Retention' => '30 days',
  'Job Retention' => '6 months',
  'AutoPrune' => 'yes',
  'Maximum Concurrent Jobs' => '20'
}
```

```
# Example Unmanaged client if client is unmanaged or custom
default['bareos']['clients']['unmanaged']['unmanaged-client-fd'] = {
  'Address' => 'unmanaged-client',
  'Password' => 'onefbofnerwob',
  'Catalog' => 'MyCatalog',
  'FDPort' => '9102'
}
```

### autochanger (if using tape storage)
```
# Example/Test Tape Autochanger Configurations
if node['bareos']['storage']['autochanger_enabled'] == true
  default['bareos']['storage']['autochangers']['autochanger-0'] = {
    'Device' => [
      'tapedrive-0',
      'tapedrive-1'
    ],
    'Changer Device' => ['/dev/tape/by-id/scsi-1TANDBERGStorageLoader_SOMEAUTOCHANGER'],
    'Changer Command' => ['"/usr/lib/bareos/scripts/mtx-changer %c %o %S %a %d"']
  }

  default['bareos']['storage']['autochangers']['autochanger-1'] = {
    'Device' => [
      'tapedrive-0'
    ],
    'Changer Device' => ['/dev/tape/by-id/scsi-1TANDBERGStorageLoader_SOMEAUTOCHANGER'],
    'Changer Command' => ['"/usr/lib/bareos/scripts/mtx-changer %c %o %S %a %d"']
  }

  default['bareos']['storage']['devices']['tapedrive-0'] = {
    'DeviceType' => 'tape',
    'DriveIndex' => '0',
    'ArchiveDevice' => 'dev/nst0',
    'MediaType' => 'lto',
    'Autochanger' => 'no',
    'AutomaticMount' => 'no',
    'MaximumFileSize' => '10GB'
  }

  default['bareos']['storage']['devices']['tapedrive-1'] = {
    'DeviceType' => 'tape',
    'DriveIndex' => '0',
    'ArchiveDevice' => 'dev/nst0',
    'MediaType' => 'lto',
    'Autochanger' => 'no',
    'AutomaticMount' => 'no',
    'MaximumFileSize' => '10GB'
  }
```

### dir\_helper
```
default['bareos']['director']['conf']['help']['Example Block'] = '# You can put extra configs here.'
```

### filesets
```
# Default Filesets
default['bareos']['clients']['filesets']['default'] = {
  'options' => {
    'signature' => 'MD5'
  },
  'include' => {
    'File' => ['/', '/home'],
    'Exclude Dir Containing' => ['.bareos_ignore']
  },
  'exclude' => {
    'File' => [
      '/var/lib/bareos',
      '/var/lib/bareos/storage',
      '/var/lib/pgsql',
      '/var/lib/mysql',
      '/proc',
      'tmp',
      '/.journal',
      '/.fsck',
      '/spool'
    ]
  }
}
```

### job\_definitions (jobdefs)
```
# Default Job Definitions
default['bareos']['clients']['job_definitions']['default-def'] = {
  'Level' => 'Incremental',
  'Fileset' => 'default-fileset',
  'Schedule' => 'monthly',
  'Storage' => 'default-file-storage',
  'Messages' => 'Standard',
  'Pool' => 'default-file-pool',
  'Priority' => '10',
  'Write Bootstrap' => '"/var/lib/bareos/%c.bsr"',
  'SpoolData' => 'no'
}

default['bareos']['clients']['job_definitions']['default-catalog-def'] = {
  'Level' => 'Full',
  'Fileset' => 'Catalog',
  'Schedule' => 'WeeklyCycleAfterBackup',
  'Storage' => 'default-file-storage',
  'Messages' => 'Standard',
  'Pool' => 'default-file-pool',
  'Allow Duplicate Jobs' => 'no'
}

default['bareos']['clients']['job_definitions']['default-restore-def'] = {
  'Fileset' => 'default-fileset',
  'Storage' => 'default-file-storage',
  'Messages' => 'Standard',
  'Pool' => 'default-file-pool',
  'Priority' => '7',
  'Where' => '/tmp/bareos-restores'
}
```

### jobs
```
# Director Jobs, basically the same as client but meant to be more admin related:
default['bareos']['director']['jobs'] = nil
```

```
# Example Client Jobs:
default['bareos']['clients']['jobs']["#{node.default['bareos']['clients']['name']}-job"] = {
  'Client' => "#{node['bareos']['clients']['name']}-fd",
  'Type' => 'Backup',
  'JobDefs' => 'default-def'
}

default['bareos']['clients']['jobs']["#{node.default['bareos']['clients']['name']}-restore-job"] = {
  'Client' => "#{node['bareos']['clients']['name']}-fd",
  'Type' => 'Restore',
  'JobDefs' => 'default-restore-def'
}
```

### pools
```
# Default Pools
default['bareos']['clients']['pools']['default-file-pool'] = {
  'Pool Type' => 'Backup',
  'Recycle' => 'yes',
  'Volume Retention' => '30 days',
  'Maximum Volume Bytes' => '10G',
  'Maximum Volumes' => '25',
  'LabelFormat' => 'FileVolume-'
}
```

### schedules
```
# Default Schedules
default['bareos']['clients']['schedules']['monthly'] = {
  'Description' => [
    'Default Monthly Schedule'
  ],
  'Run' => [
    'Full 1st sun at 23:05',
    'Differential 2nd-5th sun at 23:05',
    'Incremental mon-sat at 23:05'
  ],
  'Enabled' => [
    'yes'
  ]
}
```

### sd\_helper
```
default['bareos']['storage']['conf']['help']['Example Block'] = '# You can put extra configs here.'
```

### storages
```
# Default Storages
default['bareos']['clients']['storages']['default-file-storage'] = {
  'Address' => node['bareos']['storage']['name'], # N.B. Use a fully qualified name here
  'Device' => 'FileStorage',
  'Media Type' => 'File'
}
```

# Contributing
1. Fork the repository on Github
2. Create a named feature branch (like ```add_component_x```)
3. Write your change
4. Write kitchen and/or chefspec (rspec) tests for your change (if possible)
5. Run the tests, ensuring they all pass or travis-ci will do it for you
6. Submit a Pull Request using GitHub

## License and Authors

### License
Copyright (C) 2016 Leonard TAVAE

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

### Authors
* Leonard TAVAE
* Ian Smith
* Gerhard Sulzberger
