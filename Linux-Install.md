# Download latest release
[https://github.com/versity/versitygw/releases/latest](https://github.com/versity/versitygw/releases/latest)

# Linux

Choose the appropriate package for your distro and architecture. There are release packages for RedHat and Debian based distros, and binary tarball packages that can be used with any other distros.

You can find the architecture of your linux system with the `uname -m` command. For x86_64 arch, use the amd64 rpm/deb or x86_64 tar.gz. For arm64, use the arm64 rpm/deb/tar release.

## RedHat yum/dnf/rpm based distros
* Download the `versitygw_<version>_linux_<amd64/arm64>.rpm` from the release page choosing the appropriate one for your architecture.
* Install the rpm with `sudo dnf install ./versitygw_*.rpm`

## Debian
* Download the `versitygw_<version>_linux_<amd64/arm64>.deb` from the release page choosing the appropriate one for your architecture.
* Install the deb with `sudo dpkg -i ./versitygw_*.deb`

## Other
* Download the `versitygw_<version>_linux_<x86_64/arm64>.tar.gz` from the release page choosing the appropriate one for your architecture.
* Extract files from tar with `tar -xzf versitygw_*.tar.gz`

# Configuring the service
The package installations will install the versitygw binary as well as the systemd service files and example configuration file. The configuration files go below the directory `/etc/versitygw.d`. The packages install an example config `/etc/versitygw.d/example.conf`.

Copy the example config file to `/etc/versitygw.d/` naming it to a unique service name. For example, if the service name is "mygateway", then the file should be named `/etc/versitygw.d/mygateway.conf`. Edit the config file based on desired options for the gateway service. The systemd template file `/lib/systemd/system/versitygw@.service` will automatically load the configuration file for the service by name.  Replace the references to "mygateway" below with the appropriate name for the gateway based on the config file name you chose above.

To start the gateway, use the following command:
```
systemctl start versitygw@mygateway
```
To enable the gateway to start on boot, use the following command:
```
systemctl enable versitygw@mygateway
```
To stop the gateway, use the following command:
```
systemctl stop versitygw@mygateway
```

To troubleshoot failures starting the service, `journalctl` can give more detailed output:
```
journalctl -u versitygw@mygateway.service
```

There can be multiple gateway services running on the same host. Each gateway service must have a unique service name with a unique configuration file in `/etc/versitygw.d/`. They must also listen on different ports and/or interfaces using the VGW_PORT option.

The `/etc/versitygw.d/example.conf` describes all options (also documented in [Global-Options](./Global-Options)). The default values are specified in the options that are commented out. A minimal config for a `posix` backend would be:
```
VGW_BACKEND=posix
VGW_BACKEND_ARG=<gateway_root_path>
ROOT_ACCESS_KEY_ID=<access_key>
ROOT_SECRET_ACCESS_KEY=<secret_key>
```