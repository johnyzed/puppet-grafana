####Table of Contents

1. [Overview](#overview)
2. [Module Description](#module-description)
3. [Setup](#setup)
    * [Beginning with Grafana](#beginning-with-grafana)
4. [Usage](#usage)
    * [Classes and Defined Types](#classes-and-defined-types)
    * [Templates](#templates)

##Overview

This module installs Grafana, a dashboard and graph editor for Graphite.

##Module Description

This module assumes you will be using, and has only been tested against, Graphite. Therefore it is assumed you have a Graphite installation Grafana will be pulling data from. This module does **not** manage Graphite in any way.

##Setup

This module assumes that you will be serving Grafana using a web server such as Apache . It will:

* Download and extract Grafana to an installation directory or use your package manager
* Create a symlink in the installation directory to enable future version upgrades if not using a package manager
* Configure Grafana with a default Graphite and Elasticsearch host of 'localhost'
* Allow you to override the Grafana version, download URL, and Graphite / Elasticsearch urls

Add the following to your manifest to create an Apache virtual host to serve Grafana. **NOTE** This requires the puppetlabs-apache module.

```puppet
    # Grafana is to be served by Apache
    class { 'apache':
        default_vhost   => false,
    }

    # Create Apache virtual host
    apache::vhost { 'grafana.example.com':
        servername      => 'grafana.example.com',
        port            => 80,
        docroot         => '/opt/grafana',
        error_log_file  => 'grafana-error.log',
        access_log_file => 'grafana-access.log',
        directories     => [
            {
                path            => '/opt/grafana',
                options         => [ 'None' ],
                allow           => 'from All',
                allow_override  => [ 'None' ],
                order           => 'Allow,Deny',
            }
        ]
    }
```

###Beginning with Grafana

To install Grafana with the default parameters:

```puppet
    class { 'grafana': }
```

This assumes that you have Graphite running on the same server as Grafana, and that you want to install Grafana to in /opt. To establish customized parameters:

```puppet
    class { 'grafana':
      install_dir  => '/usr/local',
      datasources  => {
        'graphite' => {
          'type'    => 'graphite',
          'url'     => 'http://172.16.0.10',
          'default' => 'true'
        },
        'elasticsearch' => {
          'type'      => 'elasticsearch',
          'url'       => 'http://172.16.0.10:9200',
          'index'     => 'grafana-dash',
          'grafanaDB' => 'true',
        },
      }
    }
```
##Usage

###Classes and Defined Types

####Class: `grafana`

The Grafana module's primary class, `grafana`, guides the basic setup of Grafana on your system.

```puppet
    class { 'grafana': }
```
**Parameters within `grafana`:**

#####`version`

Controls the version of Grafana that gets downloaded and extracted. The default value is the latest stable version available at the time of module release.

#####`install_dir`

Controls which directory Grafana is downloaded and extracted in. The default value is '/opt'.

#####`symlink`

Determines if a symlink should be created in the installation directory for the extracted archive. The default is 'true'.

#####`symlink_name`

Sets the name to be used for the symlink. The default is '${install_dir}/grafana'.

#####`grafana_user`

The user that will own the installation directory. The default is 'root' and there is no logic in place to check that the value specified is a valid user on the system.

#####`grafana_group`

The group that will own the installation directory. The default is 'root' and there is no login in place to check that the value specified is a valid group on the system.

#####`datasources`

The graphite, elasticsearch, influxdb, and opentsdb connection properties. See init.pp for an example.

###Templates

This module currently makes use of one template to manage Grafana's main configuration file, `config.js`.

