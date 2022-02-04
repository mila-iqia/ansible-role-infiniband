InfiniBand
==========

This role installs and configure InfiniBand interfaces.

Role Variables
--------------

Define the APT or YUM/DNF repositories, with the path to the GPG key.

    # Default apt repository
    infiniband_apt_repository: 'deb https://linux.mellanox.com/public/repo/infiniband/latest/ubuntu18.04/amd64/ ./'

    # Default yum/dnf repository
    infiniband_yum_repository: 'https://linux.mellanox.com/public/repo/infiniband/latest/rhel8.3/$basearch/'

    # Mellanox GPG Key
    infiniband_gpg_key: 'http://www.mellanox.com/downloads/ofed/RPM-GPG-KEY-Mellanox'

Define the list of interfaces to configure with IPoIB. Each item of the list
must define the interface name (iface), the offset from the default ipv4
address and the CIDR prefix.

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

Example Playbook
----------------

Install and configure clustershell:

    - hosts: computes:&infiniband
      roles:
        - role: mila.infiniband
          tags: 'role::infiniband'
