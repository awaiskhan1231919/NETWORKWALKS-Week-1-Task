# VirtualBox and Kali Linux Lab

A small, isolated virtual lab built with Oracle VirtualBox and Kali Linux. This project documents installing VirtualBox, creating a VirtualBox NAT Network, connecting a Kali VM, and configuring guest networking for learning and authorized testing.

> Use this lab only on systems and networks you own or are explicitly authorized to test.

## Project goals

- Install Oracle VirtualBox on a Windows host.
- Create a VirtualBox NAT Network for virtual machines to share.
- Connect the Kali Linux VM to that network.
- Configure and verify the guest's IPv4 connectivity.
- Keep a clean VM snapshot as a recovery point before experiments.

## Lab topology

```text
Windows host
└── Oracle VirtualBox 7.2.22
    └── NAT Network: NatNetwork (10.0.0.0/24)
        └── Kali Linux VM
```

A NAT Network allows attached VMs to communicate with each other and provides outbound connectivity through the host. It is suitable for this small lab; it is not a substitute for carefully isolating systems when performing security exercises.

## Configuration

| Component | Configuration |
| --- | --- |
| Host OS | Windows (shown in installation screenshots) |
| Hypervisor | Oracle VirtualBox 7.2.22 (amd64) |
| Guest OS | Kali Linux 2026.3 (VM name shown in VirtualBox) |
| Virtual network | NAT Network, `NatNetwork` |
| Virtual network prefix | `10.0.0.0/24` |
| DHCP | Enabled on the NAT Network |
| Example guest address | `10.0.0.2/24` |
| Example gateway and DNS | `10.0.0.1` |

The screenshots do not show the host hardware allocation, guest RAM/CPU, or successful connectivity test results, so those are intentionally not specified here.

## Setup walkthrough

### 1. Install VirtualBox

Install Oracle VirtualBox on the host. The screenshots show version 7.2.22 completing installation and the VirtualBox Manager opening.

![VirtualBox installation in progress](Screenshots/Screenshot%202026-10-26%20182239.png)

![VirtualBox installation completed](Screenshots/Screenshot%202026-10-26%20182305.png)

### 2. Create the NAT Network

In VirtualBox Manager, open the network tools and create or edit a NAT Network. The captured configuration is:

```text
Name: NatNetwork
IPv4 Prefix: 10.0.0.0/24
DHCP: Enabled
```

![VirtualBox NAT Network configuration](Screenshots/Screenshot%202026-10-26%20184102.png)

### 3. Connect the Kali VM

Open the Kali VM's settings, select **Network**, and attach its network adapter to **NAT Network**. Select `NatNetwork` as the network name and make sure the adapter and virtual cable are enabled. The screenshot shows Kali Linux 2026.3 running.

![Kali VM attached to NatNetwork](Screenshots/Screenshot%202026-10-26%20184537.png)

### 4. Configure guest IPv4 settings

The Kali connection screenshot shows **Automatic (DHCP)** plus an additional address, gateway, and DNS entry. The values shown are `10.0.0.2`, `10.0.0.1`, and `10.0.0.1`.

![Kali IPv4 connection settings](Screenshots/Screenshot%202026-10-26%20185030.png)

**Correct the prefix before relying on this configuration:** the screenshot shows netmask/prefix `8`, which means `/8`; that does not match the VirtualBox network's `10.0.0.0/24`. Use prefix `24` (`255.255.255.0`) for an address on this NAT Network. Also avoid assigning a static address that may be handed out to another VM by DHCP: reserve the address or choose one outside the DHCP pool.

The screenshot uses the **Automatic (DHCP)** method and adds a static address as an additional address. For a simple DHCP-managed setup, use DHCP only. If a stable guest address is required, configure a DHCP reservation or use an address outside the DHCP pool, with the `/24` prefix, gateway `10.0.0.1`, and DNS `10.0.0.1`.

## Verification

Run these commands in Kali after applying the network configuration:

```bash
ip -4 address
ip route
ping -c 4 10.0.0.1
getent hosts example.com
```

Expected checks:

- The intended interface has an IPv4 address in `10.0.0.0/24`.
- The default route uses `10.0.0.1`.
- The gateway responds to ping.
- DNS lookup returns an address.

These results have not been captured in the screenshots; run the checks to confirm the live configuration. If a check fails, verify that the VM adapter is enabled and attached to `NatNetwork`, then check the NAT Network prefix, DHCP settings, and guest prefix/gateway.

## Snapshot and safe use

After verifying the base installation and network, create a VirtualBox snapshot such as `Kali-Lab-Baseline`. Restore it if later lab changes make the VM unusable. Keep experiments confined to systems you own or have permission to test.

## What this project demonstrates

- Installing a desktop hypervisor and running a Kali Linux VM.
- Creating and selecting a VirtualBox NAT Network.
- Configuring IPv4 address, prefix, gateway, and DNS settings.
- Comparing guest network settings against the virtual network prefix.
- Documenting setup evidence and a reproducible verification procedure.

## Screenshots

The screenshots used in this write-up are stored in [`Screenshots/`](Screenshots/).
