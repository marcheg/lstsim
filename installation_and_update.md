# Installation and Update

`lst` is packaged either as **tar.gz** archive or as an **rpm** archive. The easiest way to maintain an installation is with the **rpm** archive. Tar archive can be used in installations where rpm is not supported, but, however, requires more manual work to set up.


## Content

- [Checksum Verification](#checksum-verification)
- [RPM Installation and Update](#rpm-installation-and-update)
- [Install RPM](#install-or-update-rpm)
- [First time install configuration](#first-time-installation)
- [Testing the Installation](#testing-the-installation)
- [RPM Deinstallation](#rpm-deinstallationremoval)
- [TAR Archive installation](#tar-archive-installation)
- [Installation of additional `lstsim`s](#installation-of-additional-lstsims)


## Checksum Verification

> Before any installation, verify sha256 checksums that the software you downloaded is not compromised. Run the following command in the directory where you stored the artefacts and the checksums.
```sh
[vagrant@lst-test dist]$ sha256sum -c lst-0.7.0-100.sha256.checksums
lst-0.7.0-100.tar.gz: OK
lst-0.7.0-100.x86_64.rpm: OK
lstsim-0.7.0-100.x86_64.rpm: OK
```


# RPM Installation and Update

The `lst` RPM Archive is based on the following structure:
- the home is in `/opt/lst` that contains most of the artefacts
- the configuration of services is located in `/etc/lst`
- the first time installation creates `lst` user and `lst` group which are used to run the background services
- the background services are registered with `systemd`, but are not automatically started

| Directory | Purpose |
|-----------|---------|
| `/etc/lst` | contains service configuration files for daemon `lstd.yaml` and for simulator `lstsim.yaml`|
| `/opt/lst` | is home directory for `lst` containing the sample `inventory.yaml` and `scenarios.yaml`. Additionally some helper configurations are provided for prometheus metric scraping configuration or grafana starter dashboard. |
| `/opt/lst/bin` | contains the services and cli tool `lst`. The cli `lst` tool is automatically added to the path and should be available directly. |
| `/opt/lst/docs` | contains documentation for `lst` toolset which is also available at [lstsim](https://marcheg.github.io/lstsim/). |
| `/opt/lst/OSS-LICENSES` | contains all 3rd party software licenses | 
| `/usr/lib/systemd/system` | contains `systemd` service configuration files for `lstd.service` and `lstsim.service` |


## Install or Update RPM

```sh
[vagrant@lst-test dist]$ sudo rpm -U lst-0.7.0-100.x86_64.rpm
Creating user lst with home directory: /opt/lst
Done.
Created symlink /etc/systemd/system/multi-user.target.wants/lstd.service → /usr/lib/systemd/system/lstd.service.
Created symlink /etc/systemd/system/multi-user.target.wants/lstsim.service → /usr/lib/systemd/system/lstsim.service.
```

### First-time installation

You may want to set password for the created `lst` user:

```sh
$ sudo passwd lst
```

By default the services are registered but not started - start them:

```sh
[vagrant@lst-test dist]$ sudo systemctl start lstd
[vagrant@lst-test dist]$ sudo systemctl start lstsim
```

You can verify, whether everything is ok:

```sh
[vagrant@lst-test dist]$ systemctl status lstsim
● lstsim.service - lstsim
   Loaded: loaded (/usr/lib/systemd/system/lstsim.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2022-06-13 09:55:37 CEST; 6s ago
 Main PID: 4002 (lstsim)
    Tasks: 5 (limit: 12393)
   Memory: 5.0M
   CGroup: /system.slice/lstsim.service
           └─4002 /opt/lst/bin/lstsim -c /etc/lst/lstsim.yaml
[vagrant@lst-test dist]$ systemctl status lstd
● lstd.service - lstd
   Loaded: loaded (/usr/lib/systemd/system/lstd.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2022-06-13 09:55:34 CEST; 11s ago
 Main PID: 3991 (lstd)
    Tasks: 7 (limit: 12393)
   Memory: 11.2M
   CGroup: /system.slice/lstd.service
           └─3991 /opt/lst/bin/lstd -c /etc/lst/lstd.yaml
```

Alternatively you can check logs if it is not running:

```sh
[vagrant@lst-test dist]$ sudo journalctl -f -u lstd
-- Logs begin at Mon 2022-06-13 08:24:05 CEST. --
Jun 13 09:11:27 lst-test systemd[1]: Stopped lstd.
Jun 13 09:11:27 lst-test systemd[1]: Started lstd.
Jun 13 09:11:27 lst-test systemd[1]: lstd.service: Main process exited, code=exited, status=1/FAILURE
Jun 13 09:11:27 lst-test systemd[1]: lstd.service: Failed with result 'exit-code'.
Jun 13 09:11:28 lst-test systemd[1]: Stopped lstd.
Jun 13 09:19:51 lst-test systemd[1]: Started lstd.
Jun 13 09:19:51 lst-test lstd[3720]: lst-test 13.06.2022 09:19:51.724983 lstd      NOTE  lstd lst daemon starting
Jun 13 09:19:51 lst-test lstd[3720]: lst-test 13.06.2022 09:19:51.727044 lstd      NOTE  lstd lst daemon initialized
Jun 13 09:19:51 lst-test lstd[3720]: lst-test 13.06.2022 09:19:51.727223 lstd      NOTE  lstd HTTP Server started: :7500
Jun 13 09:19:51 lst-test lstd[3720]: lst-test 13.06.2022 09:19:51.727231 lstd      NOTE  lstd lst daemon started
♥
[vagrant@lst-test dist]$ sudo journalctl -f -u lstsim
-- Logs begin at Mon 2022-06-13 08:24:05 CEST. --
Jun 13 09:20:22 lst-test systemd[1]: Started lstsim.
Jun 13 09:20:22 lst-test lstsim[3734]: lst-test 13.06.2022 09:20:22.497410 lstsim    NOTE  lstsim lstsim starting
Jun 13 09:20:22 lst-test lstsim[3734]: lst-test 13.06.2022 09:20:22.522635 lstsim    NOTE  lstsim lstsim Instance name changed to 'lstsim-lst-test-7501'.
Jun 13 09:20:22 lst-test lstsim[3734]: lst-test 13.06.2022 09:20:22.523038 lstsim    NOTE  lstsim lstsim initialized
Jun 13 09:20:22 lst-test lstsim[3734]: lst-test 13.06.2022 09:20:22.523057 lstsim    NOTE  lstsim HTTP Server started: :7501
Jun 13 09:20:22 lst-test lstsim[3734]: lst-test 13.06.2022 09:20:22.523066 lstsim    NOTE  lstsim lstsim started
Jun 13 09:20:22 lst-test lstsim[3734]: lst-test 13.06.2022 09:20:22.526182 lstsim    NOTE  lstsim Registered at master [127.0.0.1:7500]: 204 No Content. /0/
q⌂♥
```

### Update of RPM

The procedure to update the RPM is similar to first time installation. After the update the post script will restart the systemd `lst` services. Thus, do not update RPM while running a simulation (which is anyway not the best idea).

## Testing the installation

To verify what version are you running, run `lst --help`

```sh
[vagrant@lst-test dist]$ lst --help
LST
Version: lst v0.7.0-100
License: Trial license

  lst

 Usage of lst

   lst [options]


    EXAMPLES
      ...to be done...

 Options:

  -c, --connect string   lst daemon address ip:port (default "127.0.0.1:7500")
  -h, --help             print usage information
      --nocolor          suppress colors in ouputs

(c) marcheg, All rights reserved
[vagrant@lst-test dist]$
```

To test simulator status, start `lst`:

```sh
[vagrant@lst-test dist]$ lst
lst > status
# Simulation Status

ID  SIMULATOR             ADDRESS         STATUS   RATE  SCENARIO  TRAFFIC MODEL  CRUD MODEL  STARTED  ENDED  DURATION
--  --------------------  --------------  -------  ----  --------  -------------  ----------  -------  -----  --------
 1  lstsim-lst-test-7501  127.0.0.1:7501  STOPPED     0
# 1 entries

lst >
```

To see what commands can be used, type `help` in the `lst` prompt:

```sh
lst > help
Supported lst commands:

  help          - prints available commands
  exit          - ends current lst session
  inventory     - shows the available LDAP destinations, data ranges and traffic models
  scenarios     - shows available simulation scenarios
  models        - shows available traffic models
  status        - shows the current status of simulation with metrics and lstsim clients
  start         - starts a load and stress simulation for selected scenario
  stop          - stops load and stress simulation
  metrics       - shows the current metrics of the simulation, either rate or latency or both
  rate          - shows or modifies the current rate of requests per second in simulator instances

Type <command> -h or <command> --help for more information on specific command
lst >
```

# RPM Deinstallation/removal

The deinstallation is as simple as:

```sh
[vagrant@lst-test dist]$ sudo rpm -e lst-0.7.0-100
Removed /etc/systemd/system/multi-user.target.wants/lstd.service.
Removed /etc/systemd/system/multi-user.target.wants/lstsim.service.
[vagrant@lst-test dist]$
```

Eventually you will want to clean up also the `lst` user:

```sh
$ sudo userdel --remove lst
$ sudo groupdel lst
```

To check whether group or user exists:

```sh
$ id lst
$ getent group lst
```

# Tar Archive installation

To extract the archive into a specific directory use the following command:

```sh
[vagrant@lst-test vagrant]$ tar -xvzf dist/lst-0.7.0-100.tar.gz -C /home/vagrant
lst-0.7.0-100/
lst-0.7.0-100/lstsim.yaml
lst-0.7.0-100/lstd.service
lst-0.7.0-100/inventory.yaml
lst-0.7.0-100/LICENSE
lst-0.7.0-100/promgraf-compose.yml
lst-0.7.0-100/demo.ldif
lst-0.7.0-100/lstsim
lst-0.7.0-100/docs/
lst-0.7.0-100/docs/docs/
lst-0.7.0-100/docs/docs/lstd_and_lstsim_configuration.md
lst-0.7.0-100/docs/docs/lst_cli_guide.md
lst-0.7.0-100/docs/docs/README.md
lst-0.7.0-100/docs/docs/inventory_and_scenarios_api.md
lst-0.7.0-100/docs/docs/deployment_options.md
lst-0.7.0-100/docs/docs/use_case_examples.md
lst-0.7.0-100/docs/docs/automation_api.md
lst-0.7.0-100/docs/docs/installation_and_update.md
lst-0.7.0-100/docs/docs/metrics.md
lst-0.7.0-100/prometheus-lstd.yml
lst-0.7.0-100/lst
lst-0.7.0-100/lstd
lst-0.7.0-100/lstd.yaml
lst-0.7.0-100/scenarios.yaml
lst-0.7.0-100/lstsim.service
lst-0.7.0-100/OSS-LICENSES/
...
```

This will create `lst-<version>` subdirectory with the `lst` artefacts.

```sh
[vagrant@lst-test lst-0.7.0-100]$ ll
total 30928
-rw-r--r--. 1 vagrant vagrant      458 Jun 12 21:29 demo.ldif
drwxr-xr-x. 3 vagrant vagrant       18 Jun 12 21:29 docs
-rw-r--r--. 1 vagrant vagrant     4642 Jun 12 21:29 inventory.yaml
-rw-r--r--. 1 vagrant vagrant      910 Jun 12 21:29 LICENSE
-rwxr-xr-x. 1 vagrant vagrant  8658482 Jun 12 21:29 lst
-rwxr-xr-x. 1 vagrant vagrant 13775413 Jun 12 21:29 lstd
-rw-r--r--. 1 vagrant vagrant      234 Jun 12 21:29 lstd.service
-rw-r--r--. 1 vagrant vagrant      674 Jun 12 21:29 lstd.yaml
-rw-r--r--. 1 vagrant vagrant    51858 Jun 12 21:29 lst-grafana-dashboard.json
-rwxr-xr-x. 1 vagrant vagrant  9128697 Jun 12 21:29 lstsim
-rw-r--r--. 1 vagrant vagrant      240 Jun 12 21:29 lstsim.service
-rw-r--r--. 1 vagrant vagrant      861 Jun 12 21:29 lstsim.yaml
drwxr-xr-x. 6 vagrant vagrant      176 Jun 12 21:29 OSS-LICENSES
-rw-r--r--. 1 vagrant vagrant      163 Jun 12 21:29 prometheus-lstd.yml
-rw-r--r--. 1 vagrant vagrant      549 Jun 12 21:29 promgraf-compose.yml
-rw-r--r--. 1 vagrant vagrant     4254 Jun 12 21:29 scenarios.yaml
```

The setup of systemd service must be performed manually. Prepared service files are available in the directory.

# Installation of additional `lstsim`s

to be described