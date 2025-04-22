# netways.icinga_contrib.icingaweb2_modules

This role is meant to **manage Icinga Web 2 modules** which are not managed by the [**official Ansible Collection by Icinga**](https://github.com/Icinga/ansible-collection-icinga/).  
It tries to follow the same general structure as its inspiration but also adds to it.

> You have to ensure compatibility between module versions you install and the rest of your system. The module "grafana" needs a specific version of PHP for example.

Tasks it can do:

* Install Icinga Web 2 modules from source, git or tar ball
* Enable/Disable modules
* Configure modules

Table of contents:

* [Variables](#variables)
  * [General](#general)
  * [Grafana](#grafana)
  * [Map](#map)
  * [Map Datatype](#map-datatype)
  * [perfdatagraphs](#perfdatagraphs)
  * [perfdatagraphsgraphite](#perfdatagraphsgraphite)
  * [perfdatagraphsinfluxdbv1](#perfdatagraphsinfluxdbv1)
  * [perfdatagraphsinfluxdbv2](#perfdatagraphsinfluxdbv2)

* [Example Playbooks](#example-playbooks)
  * [Grafana Playbooks](#grafana-playbooks)
  * [Map Playbooks](#map-playbooks)
  * [Map Datatype Playbooks](#map-datatype-playbooks)
  * [perfdatagraphs with Backend Playbooks](#perfdatagraphs-playbooks)

# Variables

## General

- `icingaweb2_modules_config_path`: `string`  
  The path where your Icinga Web 2 configuration is stored.  
  Default: `/etc/icingaweb2`

- `icingaweb2_modules_path`: `string`  
  The path where your Icinga Web 2 modules are located.  
  Default: `/usr/share/icingaweb2/modules`

- `icingaweb2_modules`: `dictionary`  
  This variable holds keys for all modules to be managed. Each key is a dictionary itself and holds information regarding that specific module.  
  Each module might provide a specific set of options to be used. The handling of those options happens within that module's task file (e.g. "*tasks/`module`/configure_`module`*").  
  All module keys share a common set of options that handle the installation of the modules.  
  Default: `none`  
  Common options:
    - `source`: `string`  
      The installation method (`package | git | archive`).  
      **Required**
    - `package_name`: `string`  
      The name of the package to be installed if different from the module's name.  
      **Important:** you **should** set this key regardless of `source` to avoid accidentally removing a package with the same name as your module (e.g. "*grafana*").
    - `url`: `string`  
      The URL to either the Git repository or the tar ball.  
      **Required if `source: git | archive`**
    - `version`: `string`  
      The version / tag for Git installation (e.g. `version: "v2.0.2"`)
    - `enabled`: `boolean`  
      If `true`, enables the module. If `false`, disables the module. If not set, does nothing.
    - `remote_source:` `boolean`
      If set to `false`, the archive is copied from the Ansible controller to the managed node. Only has an effect if `source` is set to `archive`.  
      This is useful when the managed node cannot freely access the given URL, e.g. no access to the internet.  
      Default: `true`

## Grafana

- `icingaweb2_modules.grafana`: `dictionary`  
  - `backend`: `string`  
    The backend used in Grafana (`influxdb | graphite `)  
    Default: `influxdb`
  - `local_dashboards`: `string`  
    The directory where your local dashboards are located on your Ansible Controller (uses `lookup`).  
    Default: `none`
  - `data_source`: `string`  
    The name of your data source within Grafana.  
    Default: `none`
  - `grafana_host`: `string`  
    The Ansible inventory hostname of your Grafana server. It is used to delegate tasks for dashboard imports.  
    Default: `icingaweb2_modules.grafana.config.grafana.host`
  - `import_dashboards`: `boolean`  
    Whether to import the dashboards provided by the module into your Grafana server.  
    Default: `false`
  - `import_graphs_ini`: `boolean`  
    Whether to import all \*.ini files within the `local_dashboards` directory to be part of **graphs.ini**.  
    With this you can provide parts of graphs.ini without having to convert existing \*.ini files to YAML.  
    `config.graphs` **takes precedence** over the \*.ini files when redefining the same section.  
    Default: `false`
  - `config.grafana`: `dictionary`  
    Manages *config.ini* to configure the module. Its keys are equal to the modules [configuration file](https://github.com/netwaays/icingaweb2-module-grafana/blob/master/doc/03-module-configuration.md#example-configini-etcicingaweb2modulesgrafanaconfigini), though not necessarily complete yet.  
    For possible default values have look at [templates/grafana/config.ini.j2](templates/grafana/config.ini.j2).
  - `config.graphs`  
    Manages *graphs.ini* to configure the module. Its keys are equal to the modules [graphs file](https://github.com/netways/icingaweb2-module-grafana/blob/master/doc/04-graph-configuration.md#options), though not necessarily complete yet.  
    Here you can set specialized dashboards for specific Icinga Services or CheckCommands.  
    For possible default values have look at [templates/grafana/graphs.ini.j2](templates/grafana/graphs.ini.j2).
- `icingaweb2_modules_grafana_server_config_path`: `string`  
  The path where your Grafana server's configuration is located.  
  Default: `/etc/grafana`
- `icingaweb2_modules_grafana_server_dashboard_path`: `string`  
  The path where imported dashboards should be deployed on your Grafana server.  
  Default: `/var/lib/grafana/dashboards/icingaweb2-grafana`

## Map

- `icingaweb2_modules.map`: `dictionary`  
  - `config.map`: `dictionary`  
    Manages the section `map` within *config.ini* to configure the module. Its keys are equal to the module's [settings](https://github.com/nbuchwitz/icingaweb2-module-map/blob/master/doc/03-Exploring-the-map.md#settings).  
    For possible default values have look at [templates/map/config.ini.j2](templates/map/config.ini.j2).
  - `config.director`: `dictionary`  
    Manages the section `director` within *config.ini* to configure the module. It shares some keys with `config.map`.  
    Have a look at [templates/map/config.ini.j2](templates/map/config.ini.j2).

## Map Datatype

- `icingaweb2_modules.mapDatatype`: `none`  
  - No keys for this module.

## perfdatagraphs

- `icingaweb2_modules.perfdatagraphs`: `directory`
  - `config.perfdatagraphs`: `dictionary`  
    Manages the Section perfdatagraphs within *config.ini* to configure the module. Its keys are equal to the module's [settings](https://github.com/NETWAYS/icingaweb2-module-perfdatagraphs/blob/main/doc/03-Configuration.md).  
    For possible default values have look at [templates/perfdatagraphs/config.ini.j2](templates/perfdatagraphs/config.ini.j2).

## perfdatagraphsgraphite

- `icingaweb2_modules.perfdatagraphsgraphite`: `directory`
  - `config.perfdatagraphsgraphite`: `dictionary`  
    Manages the Section perfdatagraphsgraphite within *config.ini* to configure the module. Its keys are equal to the module's [settings](https://github.com/NETWAYS/icingaweb2-module-perfdatagraphs-graphite/tree/main/doc).  
    For possible default values have look at [templates/perfdatagraphsgraphite/config.ini.j2](templates/perfdatagraphsgraphite/config.ini.j2).

## perfdatagraphsinfluxdbv1

- `icingaweb2_modules.perfdatagraphsinfluxdbv1`: `directory`
  - `config.perfdatagraphsinfluxdbv1`: `dictionary`  
    Manages the Section perfdatagraphsinfluxdbv1 within *config.ini* to configure the module. Its keys are equal to the module's [settings](https://github.com/NETWAYS/icingaweb2-module-perfdatagraphs-influxdbv1/tree/development/doc).  
    For possible default values have look at [templates/perfdatagraphsinfluxdbv1/config.ini.j2](templates/perfdatagraphsinfluxdbv1/config.ini.j2).

## perfdatagraphsinfluxdbv2

- `icingaweb2_modules.perfdatagraphsinfluxdbv2`: `directory`
  - `config.perfdatagraphsinfluxdbv2`: `dictionary`  
    Manages the Section perfdatagraphsinfluxdbv2 within *config.ini* to configure the module. Its keys are equal to the module's [settings](https://github.com/NETWAYS/icingaweb2-module-perfdatagraphs-influxdbv2/tree/main/doc).  
    For possible default values have look at [templates/perfdatagraphsinfluxdbv2/config.ini.j2](templates/perfdatagraphsinfluxdbv2/config.ini.j2).

# Example Playbooks

## Grafana Playbooks

Install grafana module via package management. Use basic authentication for connection to the Grafana server. Use existing dashboard.

```yaml
- name: Manage Grafana Module
  hosts: 
    - icingaweb2

  vars:
    icingaweb2_modules:
      grafana:
        # Avoid removing Grafana as in "the Grafana server"
        package_name: "icinga-grafana"
        source: package
        package_name: icinga-grafana
        data_source: "influxdb_icinga"
        config:
          grafana:
            host: "grafana.localdomain:3000"
            defaultdashboard: "icinga2-default"
            defaultdashboarduid: "a515ad9f-20e2-4b4f-b068-24f3eb3e30ba"
            authentication: "basic"
            username: "admin"
            password: "12345678"

  roles:
    - netways.icinga_contrib.icingaweb2_modules
```

---

Install grafana module via Git and configure connection to Grafana server. Also import dashboards to Grafana server.

```yaml
- name: Manage Grafana Module
  hosts: 
    - icingaweb2

  vars:
    icingaweb2_modules:
      grafana:
        # Avoid removing Grafana as in "the Grafana server"
        package_name: "icinga-grafana"
        source: git
        url: "https://github.com/netways/icingaweb2-module-grafana.git"
        version: "v2.0.2"
        enabled: true
        data_source: "influxdb_icinga"
        import_dashboards: true
        config:
          grafana:
            host: "localhost:3000"
            defaultdashboard: "icinga2-default"
            authentication: "token"
            token: "glsa_VBTDQJ998i6fHPxeeYrvFrKBXfjHjaGx_4e52ed40"
            enableLink: true

  roles:
    - netways.icinga_contrib.icingaweb2_modules
```

Install grafana module via Git and configure connection to Grafana server. Also import dashboards to Grafana server. Let the Role automatically find the correct dashboard uid for the CheckCommand *disk*.

```yaml
- name: Manage Grafana Module
  hosts: 
    - icingaweb2

  vars:
    icingaweb2_modules:
      grafana:
        # Avoid removing Grafana as in "the Grafana server"
        package_name: "icinga-grafana"
        source: git
        url: "https://github.com/netways/icingaweb2-module-grafana.git"
        version: "v2.0.2"
        enabled: true
        data_source: "influxdb_icinga"
        import_dashboards: true
        config:
          grafana:
            host: "localhost:3000"
            defaultdashboard: "icinga2-default"
            authentication: "token"
            token: "glsa_VBTDQJ998i6fHPxeeYrvFrKBXfjHjaGx_4e52ed40"
            enableLink: true
          graphs:
            disk:

  roles:
    - netways.icinga_contrib.icingaweb2_modules
```

## Map Playbooks

Install the map module via Git and set it to show the number of problems in a clustered view instead of the number of markers within that cluster.

```yaml
- name: Manage map module
  hosts:
    - icingaweb2

  vars:
    icingaweb2_modules:
      map:
        config:
          map:
            cluster_problem_count: 1
```

## Map Datatype Playbooks

Install the mapDatatype module via Git.

```yaml
- name: Manage mapDatatype module
  hosts:
    - icingaweb2

  vars:
    icingaweb2_modules:
      mapDatatype:
        package_name: "icinga-mapdatatype"
        source: git
        url: "https://github.com/nbuchwitz/icingaweb2-module-mapDatatype.git"
```

## perfdatagraphs with Backend Playbooks

Install the perfdatagraphs module via Git.

```yaml
- name: Manage perfdatagraphs module
  hosts:
    - icingaweb2

  vars:
    icingaweb2_modules:
      perfdatagraphs:
        package_name: "icinga-perfdatagraphs"
        source: git
        url: "https://github.com/netways/icingaweb2-module-perfdatagraphs"
        enabled: true
        config:
          perfdatagraphs:
            default_backend: InfluxDBv2
      perfdatagraphsinfluxdbv2:
        package_name: "icinga-perfdatagraphs-influxdbv2"
        source: git
        url: "https://github.com/netways/icingaweb2-module-perfdatagraphs"
        enabled: true
        config:
          perfdatagraphsinfluxdbv2:
            api_url: "localhost:8086"
            api_org: "monitoring"
            api_bucket: "icinga2"
            api_token: "glsa_VBTDQJ998i6fHPxeeYrvFrKBXfjHjaGx"
            api_timeout: "0"
            api_tls_insecure: "1"
  roles:
    - netways.icinga_contrib.icingaweb2_modules
```
