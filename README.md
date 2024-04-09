InfiniBand
==========

This role installs and configures InfiniBand interfaces.

Role Variables
--------------

### Installation

Define the APT or YUM/DNF repositories, with the path to the GPG key.

    # Default apt repository
    infiniband_apt_repository: 'deb https://linux.mellanox.com/public/repo/mlnx_ofed/latest/ubuntu18.04/amd64/ ./'

    # Default yum/dnf repository
    infiniband_yum_repository: 'https://linux.mellanox.com/public/repo/mlnx_ofed/latest/rhel8.3/$basearch/'

    # Mellanox GPG Key
    infiniband_gpg_key: 'http://www.mellanox.com/downloads/ofed/RPM-GPG-KEY-Mellanox'

To upgrade the packages to the latest version available.

    infiniband_upgrade: True

To pin the priority of the repository (APT only).

    infiniband_apt_priority: 490

The name of the kernel headers package might change across distros. This can be
defined with:

    infiniband_kernel_headers_package: 'linux-headers'

Some appliances like the NVIDIA DGX run their own software stack. When an
appliance already manages the installation of the Mellanox OFED drivers, there
is no need to configure additional repositories or to install the drivers. In
such cases, the parameters below can be set to False in the host inventory.
This is also true when using the kernel modules from the distro.

    infiniband_configure_repos: True
    infiniband_install_kernel_modules: True

### IPoIB

Define the list of interfaces to configure with IPoIB. Each item of the list
must define the interface name (iface), the offset from the default ipv4 address
and the CIDR prefix. If the list is not defined, the role will not configure any
IPoIB interface.

    infiniband_ipoib_interfaces:
      - iface: 'ib0'
        offset: -3064461568
        prefix: 17

To compute the offset, use:

    $ python3 <<EOF
    import ipaddress
    source_net='192.168.121.0'
    target_net='10.0.128.0'
    offset=(int(ipaddress.ip_address(source_net))-int(ipaddress.ip_address(target_net)))
    print(f"offset: -{offset}")
    EOF

### Virtualization - SR-IOV

Configure the list of Mellanox HCAs with the parameters to apply. The pci_bus
must match the value in /sys/bus/pci/devices/ (e.g.
/sys/bus/pci/devices/0000:41:00.0/)

    infiniband_hca_devices:
      - device: mlx5_0
        pci_bus: '0000:41:00.0'
        sriov_en: True
        num_of_vfs: 8

Only parameters `SRIOV_EN` and `NUM_OF_VFS` are supported for now.

Define the prefix to use for the 64-bits IB GUID of VFs.  The role will define
the GUID with:

 - prefix (40 bits)
 - "00" (8 bits)
 - device ID (8 bits): item index in list infiniband_hca_devices above
 - VF ID (8 bits): index of VF in range(infiniband_hca_devices[*].num_of_vfs)

It is highly recommended to define a value different than the default if you
plan to configure more than one host with SR-IOV in your IB fabric.

To allow failover in high availability configuration, make sure to use the same
infiniband_hca_devices and infiniband_guid_prefix for all hosts where a given VM
could run.

    infiniband_guid_prefix: "4d:69:6c:61:00"

By default, the role will reboot the hosts to load any new configuration. It is
possible to forbid the reboot with:

    infiniband_allow_reboot: false

To avoid any unexpected downtime in high availability clusters, the role will
reboot the hosts one after the other. It is possible to increase the throttle
with:

    infiniband_throttle_reboot: "{{ ansible_play_hosts | length }}"

Example Playbook
----------------

Install and configure InfiniBand:

    - hosts: computes:&infiniband
      roles:
        - role: mila.infiniband
          tags: 'role::infiniband'
