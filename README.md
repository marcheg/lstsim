# `lst` - Load and Stress Test Toolset

`lst` is a scalable, configurable **Load and Stress Test Simulator** mainly designed for User Data Repository (UDR). It offers high flexibility and adaptability to various deployment options, any data model, any traffic model definitions and multitude of test scenarios. Its key design architecture, beside configurablity, is centraly managed architecture with horizontally scalable simulator instances. The toolset itself collects wide variety of metrics to evaluate the performance of the underlying UDR. 

> **IMPORTANT**: The project has not yet reached a stable state and is in active evolution. Therefore the APIs, commands, interfaces and configurations are still subject to change as new functionality and re-work activities are under way. Upon reaching version 1.0.0 the interfaces will be stabilized.

---

## Content

1. [Use Cases](#use-cases)
1. [Benefits](#benefits)
1. [Features](#features)
1. [Quick Overview](#quick-overview)
1. [Inventory and Scenarios API](inventory_and_scenarios_api.md)
1. [Deployment Options](deployment_options.md)
1. [Use Case Examples](use_case_examples.md)
1. [`lst` CLI Guide](lst_cli_guide.md)
1. [Metrics](metrics.md)
1. [Installation and Update](installation_and_update.md)
1. [`lstd` and `lstsim` Configuration](lstd_and_lstsim_configuration.md)
1. [Automation API](automation_api.md)
1. [3GPP Standards](#3gpp-standards)


```
 _     _   
| |___| |_ 
| / __| __|
| \__ \ |_ 
|_|___/\__|
 _                    _    ___     ____  _                       ____  _                 _       _             
| |    ___   __ _  __| |  ( _ )   / ___|| |_ _ __ ___  ___ ___  / ___|(_)_ __ ___  _   _| | __ _| |_ ___  _ __ 
| |   / _ \ / _` |/ _` |  / _ \/\ \___ \| __| '__/ _ \/ __/ __| \___ \| | '_ ` _ \| | | | |/ _` | __/ _ \| '__|
| |__| (_) | (_| | (_| | | (_>  <  ___) | |_| | |  __/\__ \__ \  ___) | | | | | | | |_| | | (_| | || (_) | |   
|_____\___/ \__,_|\__,_|  \___/\/ |____/ \__|_|  \___||___/___/ |____/|_|_| |_| |_|\__,_|_|\__,_|\__\___/|_|   
                                                                                                               

```

---



# Use Cases


# Benefits

# Features

# Quick Overview

The `lst` consists of several components:
- `lst` - command line utility used to manage, control and observe simulation scenario. This is the main interface to interact with the `lstd`. This utility is colocated with the `lstd` and it communicates with the daemon over a dedicated API.
- `lstd` - daemon process running on a dedicated server. It offers an API which is used to start, change, stop simulations as well as monitoring metrics and status of the simulation. `lstd` manages all `lstsim` instances directly as a central unit. `lstd` by itself does not generate any traffic.
- `lstsim` - is a specific instance of a simulator that is controlled by `lstd` and runs a configured simulation scenario against destination. `lstsim` instances can be deployed on the same server as `lstd`, subject to sufficient CPU capacity to perform the required scenario. In addition, `lstsim` can be deployed on multiple servers to scale the load or to simulate different traffic on different sites.

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│              lst - command line utility             │
│                                                     │
└───────────────────────────┬─────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────┐
│                                                     │
│        lstd - lst daemon managing simulations       │
│                                                     │
└─────────────────────────────────────────────────────┘
        ▲                   ▲                   ▲
        │                   │                   │
┌───────┴──────┐   ┌────────┴───────┐   ┌───────┴─────┐
│              │   │                │   │             │
│    lstsim    │   │     lstsim     │   │    lstsim   │
│              │   │                │   │             │
└──────────────┘   └────────────────┘   └─────────────┘
```

The tool allows great flexibility to define various traffic models and high configurability in the simulated scenario options. The configuration is based on two main configuration files:

- `Inventory` configuration defining the environment. It defines the target destinations (e.g. UDR in testlab), data ranges (e.g. provisioned subscriptions) and traffic models (e.g. selection of various models)
- `Scenarios` configuration defining specific traffic simulations. Simulations are build on the `inventory` configuration. The same traffic model can be used in various scenarios.

Both files work together to simulate a wide variety of use cases. These files are usually prepared upfront for a specific application use cases. `lstd` is live-watching eventual changes in these files in order to pick up any changes immediately without any restart. Additionally, the configuration can be completely exchanged (by providing new inventory and scenario configurations).

**Inventory**

```yaml
# This file represents inventory for a test environment
# it will be used as a source for a possible test scenario
# it contains 3 sections - destinations, data ranges and traffic models

# Destinations
ldapDestinations:

# Data Ranges
dataRanges:

# Traffic models
trafficModels:
```

- **Destinations** - represent the UDR endpoints that can be used by the simulators to be targetted by traffic. There can be more destinations specified with several endpoints. For example, site-specific destinations or distributed per site. Which Destination Set will be used is later on selected by scenario definition. If more endpoints are specified, the traffic will be distributed across these endpoints.
- **Data Ranges** - allows a specification of several distinct data sets that can be used by various scenarios. Each data range set can consist of multiple keys which will be used during traffic simulation. Each key consists of a prefix, starting index and ending index. Together they form a range if key values (subscriptions).
- **Traffic Models** - define the traffic model which will be executed by a scenario. A model consists of a sequence of operations (1..many). The sequence will be executed for each subscription allowing to simulate realistic application behaviour.

**Scenarios**

```yaml
# This file represents test scenarios that can be started as simulations

scenarioName1:
  interface: ldap
  ldapCredentials:
    bindDN: cn=user
  # definition of the load parameters
  load:
    # type of simulation: traffic (continuous trafic) 
    simulation: traffic
    # start rate per lstsim instance
    defaultRate: 1000
    policy:
      # how the data range is shared/split across lstsims; shared, split or once
      dataRangePolicy: shared
  # failover policy in case of connection error
  failover:
    # maximum retries before failing over - 0 => no failover
    maxRetries: 0
    # delay between retry/rebind in seconds
    retryDelay: 30
  # amount of parallel connections per simulators (more connections => more load)
  connections: 100
  trafficModel: trafficModelRef
  dataRange: dataRangeSetRef
  # specify which simulators use which destinations
  # key is tag name, value is destination group
  simulators:
    - tag: ldap
      destinationGroup: all

scenarioName2:
...
```

- **Credentials** - defines the application user that will be used during the simulation.
- **Load** specification defines differents aspects of the type of simulation, initial rate of requests (can be changed online during simulation)
- **Data range policy** - indicates whether the data range is `shared` or `split` across simulators or even whether is used only `once` (usefull for pre-provisioning or cleaning up test data sets/supscriptions)
- **Failover** policy specifies parameters, how the simulators should react in case the destination endpoints are not reachable or failing. This allows for more realistic simulation of application behaviour (e.g. to simulate site failover).
- **Connections** - number of parallel connections to be opened towards destinations by a single `lstsim` instance (to some extend this drives also the parallelism of simulated traffic)
- **Traffic Model** - reference to `inventory` definition where the traffic model to be used is pre-defined
- **Data Range** - reference to `inventory` definition where the data range set to be used is pre-defined
- **Simulators**  contains a relationship between `lstsim` and destination group. Simulators of specific tags will be connected with specific destination groups. This configurability provides flexibility to connect specific simulators to e.g. dedicated sites and generate local (site-specific) load.

Single `scenarios` file can contain any number of prepared scenarios which can be selected to be run. The rate (requests per second) can be modified at scenario start and then later increased, decreased or changed online during the simulation.

The `lst` collects a wide range of metrics in 15 second intervals to evaluate the performance of the UDR during various load scenarios. Those metrics can be monitored in real-time and exported to a timeseries database and visualized in charts (e.g. prometheus with grafana front-end).

# 3GPP Standards

The following 3GPP Technical Standards are applicable:
- [TS 23.335](https://www.3gpp.org/dynareport/23335.htm) UDC; Technical realization and information flows
- [TS 29.335](https://www.3gpp.org/dynareport/29335.htm) UDC; User Data Repository Access Protocol over the Ud interface
- [TS 29.500](https://www.3gpp.org/dynareport/29500.htm) Technical Realization of Service Based Architecture
- [TS 29.501](https://www.3gpp.org/dynareport/29501.htm) Principles and Guidelines for Services Definition
- [TS 29.504](https://www.3gpp.org/dynareport/29504.htm) Unified Data Repository Services
- [TS 29.505](https://www.3gpp.org/dynareport/29505.htm) Usage of the Unified Data Repository services for Subscription Data
- [TS 29.519](https://www.3gpp.org/dynareport/29519.htm) Usage of the Unified Data Repository Service for Policy Data and Application Data and Structured Data for Exposure
- [TS 29.598](https://www.3gpp.org/dynareport/29598.htm) Unstructured data storage services


> Note: currently only `Ud` interface is supported. 5G SBI interface is foreseen to be supported in near future too.
