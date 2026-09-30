# Home Lab Network Segmentation & VLAN Design

## 1. Purpose and Overview

This document defines the network segmentation architecture for a home lab and smart-home environment.

The design aims to provide:

* Strong network segmentation and isolation
* Least-privilege access between network zones
* Reduced impact from compromised or vulnerable devices
* Secure administration of network infrastructure
* Separation of trusted, untrusted, IoT, and server workloads
* A simple and maintainable network architecture
* Centralised control of inter-VLAN traffic through the firewall

The design follows generally accepted cybersecurity and network-design principles, including:

* Defence in depth
* Least privilege
* Network segmentation
* Default deny
* Separation of administrative functions

Where applicable, the design is informed by Australian cybersecurity guidance, including the **Australian Cyber Security Centre (ACSC) Information Security Manual (ISM)** and **Essential Eight** principles.

> **Note:** These frameworks are primarily intended for organisations. This home-lab implementation adapts relevant security principles and does not claim formal compliance.

---

# 2. Network Architecture

The network is divided into separate security zones using VLANs.

| VLAN | Network           | Security Zone | Primary Purpose                           |
| ---: | ----------------- | ------------- | ----------------------------------------- |
|   10 | `192.168.10.0/27` | Management    | Network and infrastructure administration |
|   20 | `192.168.20.0/24` | Trusted       | Trusted user devices                      |
|   30 | `192.168.30.0/27` | Lab / Servers | Servers, VMs and infrastructure workloads |
|   40 | `192.168.40.0/24` | IoT           | Smart-home and IoT devices                |
|   50 | `192.168.50.0/27` | Guest         | Guest and untrusted user devices          |

The firewall/router is responsible for enforcing communication between VLANs.

**Inter-VLAN communication should be denied by default and explicitly permitted only where a documented operational requirement exists.**

---

# 3. VLAN 10 — Management

**Network:** `192.168.10.0/27`
**Subnet Mask:** `255.255.255.224`
**Usable Addresses:** `192.168.10.1 – 192.168.10.30`

## Purpose

The Management VLAN is the highest-trust network and is used exclusively for administration and management of network infrastructure.

It should contain only infrastructure that requires administrative access.

### Typical Devices

* Firewall / router
* Managed switches
* Wireless access points
* Network management interfaces
* Modem/ONT management interfaces, where appropriate
* Other network infrastructure

Where technically possible, devices should expose their management interfaces only through this VLAN.

## Access Policy

| Source                       | Access                                             |
| ---------------------------- | -------------------------------------------------- |
| Trusted VLAN                 | Blocked by default                                 |
| Trusted administrator device | Explicitly allowed                                 |
| IoT                          | Blocked                                            |
| Guest                        | Blocked                                            |
| Lab / Servers                | Blocked by default                                 |
| Management → Internet        | Restricted                                         |
| Management → Other VLANs     | Restricted and explicitly permitted where required |

Administration should preferably be performed from a designated trusted device.

### Recommended Controls

Where practical:

* Use fixed or reserved IP addresses for administrative devices
* Restrict management services to required protocols and ports
* Use HTTPS/SSH rather than insecure management protocols
* Disable unused management services
* Use MFA where supported
* Avoid exposing management interfaces directly to the Internet

### Security Objective

The Management VLAN should remain isolated even if a normal user device, IoT device, guest device, or server is compromised.

---

# 4. VLAN 20 — Trusted LAN

**Network:** `192.168.20.0/24`
**Subnet Mask:** `255.255.255.0`

## Purpose

The Trusted VLAN is the primary user network for authorised household users and trusted personal devices.

Devices connected to this network are considered trusted from a network-access perspective, but they should not automatically receive unrestricted access to every other security zone.

### Typical Devices

* Personal laptops
* Desktop/workstations
* Mobile phones
* Tablets
* Trusted printers
* Other personally managed devices

## Access Policy

Trusted devices may access:

* Internet
* Required IoT services
* Required Lab/Server services
* Guest network where administrative access is required
* Management network only from specifically authorised administrative devices

### Baseline Policy

| Traffic                 | Policy                          |
| ----------------------- | ------------------------------- |
| Trusted → Internet      | **Allowed**                     |
| Trusted → IoT           | **Allowed where required**      |
| Trusted → Lab / Servers | **Allowed where required**      |
| Trusted → Guest         | **Allowed only where required** |
| Trusted → Management    | **Restricted**                  |
| IoT → Trusted           | **Blocked by default**          |
| Guest → Trusted         | **Blocked**                     |

> The Trusted VLAN should not be treated as an unrestricted administrative network.

---

# 5. VLAN 30 — Lab / Servers

**Network:** `192.168.30.0/27`
**Subnet Mask:** `255.255.255.224`
**Usable Addresses:** `192.168.30.1 – 192.168.30.30`

## Purpose

The Lab/Server VLAN hosts server infrastructure, virtual machines, and other services used within the home laboratory.

This network should be treated as a separate security zone because servers may provide services to multiple networks and may contain sensitive configuration, credentials, or data.

### Typical Devices

* Proxmox hosts
* Virtual machines
* NAS services
* TrueNAS
* Home-lab applications
* Internal web services
* Monitoring systems
* Other server infrastructure

## Access Policy

Access should be based on the specific service required rather than unrestricted VLAN-to-VLAN communication.

### Recommended Baseline

| Traffic                    | Policy                            |
| -------------------------- | --------------------------------- |
| Trusted → Lab / Servers    | Allowed for required services     |
| Management → Lab / Servers | Allowed for administration        |
| Lab / Servers → Internet   | Restricted / controlled           |
| IoT → Lab / Servers        | Blocked by default                |
| Guest → Lab / Servers      | Blocked by default                |
| Lab / Servers → Trusted    | Blocked by default where possible |
| Lab / Servers → Management | Blocked by default                |

If an IoT device or guest device genuinely needs access to a server—for example, a TV accessing a media server—the firewall should permit only the required destination IP, port, and protocol.

### Example

```text
IoT TV
   │
   └── TCP/UDP → Jellyfin Server
                  └── Required Port
```

Rather than:

```text
IoT VLAN
   │
   └──→ Entire Server VLAN
```

### Security Objective

Prevent a compromise of an Internet-facing, IoT, or guest device from providing unrestricted access to the server environment.

---

# 6. VLAN 40 — IoT / Smart Home

**Network:** `192.168.40.0/24`
**Subnet Mask:** `255.255.255.0`

## Purpose

The IoT VLAN isolates smart-home devices from trusted user devices and infrastructure.

IoT devices may have:

* Limited security controls
* Infrequent firmware updates
* Third-party cloud dependencies
* Internet connectivity requirements
* Greater exposure to supply-chain and vendor risks
* Limited authentication and logging capabilities

Segmentation therefore reduces the potential impact of a compromised IoT device.

### Typical Devices

* Smart TVs
* Smart speakers
* Amazon Alexa devices
* Smart displays
* Smart appliances
* Smart lighting
* Smart plugs
* Other smart-home devices

## Access Policy

| Traffic             | Policy                                          |
| ------------------- | ----------------------------------------------- |
| IoT → Internet      | Allowed where required                          |
| IoT → Trusted       | **Blocked**                                     |
| IoT → Management    | **Blocked**                                     |
| IoT → Lab / Servers | Blocked by default                              |
| IoT → Guest         | **Blocked**                                     |
| Trusted → IoT       | Allowed where required                          |
| Guest → IoT         | Allowed only for specifically required services |

IoT devices should not be permitted to initiate unrestricted connections to trusted devices or infrastructure.

Where an IoT device requires communication with a server, allow only the required traffic.

### Example — Jellyfin

If a smart TV needs access to a Jellyfin server:

```text
IoT TV
  │
  └──→ Jellyfin Server
          │
          └── Required TCP/UDP Port
```

Instead of:

```text
IoT VLAN
  │
  └──→ Entire Server VLAN
```

---

# 7. VLAN 50 — Guest

**Network:** `192.168.50.0/27`
**Subnet Mask:** `255.255.255.224`
**Usable Addresses:** `192.168.50.1 – 192.168.50.30`

## Purpose

The Guest VLAN provides Internet access to devices that are not under household administrative control.

Guest devices should be considered **untrusted**.

### Typical Devices

* Visitors' smartphones
* Visitors' laptops
* Temporary devices
* Other devices that should not receive access to the trusted network

## Access Policy

| Traffic               | Policy                       |
| --------------------- | ---------------------------- |
| Guest → Internet      | **Allowed**                  |
| Guest → Trusted       | **Blocked**                  |
| Guest → Management    | **Blocked**                  |
| Guest → Lab / Servers | **Blocked**                  |
| Guest → IoT           | Blocked by default           |
| Trusted → Guest       | Allowed only where required  |
| Guest → Guest         | Client isolation recommended |

If guests need to control a specific IoT device, such as a television, access should be restricted to the required device and service.

### Example

```text
Guest Device
     │
     └──→ Smart TV
             │
             └── Required Service
```

Rather than:

```text
Guest VLAN
     │
     └──→ Entire IoT VLAN
```

## Additional Security Controls

Where supported, enable:

* Wireless client isolation
* DNS filtering
* Internet-only access
* Rate limiting
* Device/session limits
* Blocking of private network access

---

# 8. Inter-VLAN Firewall Policy

The firewall should operate using a **default-deny principle** for inter-VLAN traffic.

## Baseline Policy Matrix

| Source ↓ / Destination → | Management | Trusted    | Lab / Servers | IoT        | Guest      | Internet   |
| ------------------------ | ---------- | ---------- | ------------- | ---------- | ---------- | ---------- |
| **Management**           | —          | Restricted | Restricted    | Restricted | Restricted | Restricted |
| **Trusted**              | Restricted | —          | Allow*        | Allow*     | Allow*     | Allow      |
| **Lab / Servers**        | Restricted | Restricted | —             | Restricted | Restricted | Restricted |
| **IoT**                  | Block      | Block      | Block*        | —          | Block      | Allow*     |
| **Guest**                | Block      | Block      | Block         | Block*     | —          | Allow      |

* Access should be explicitly restricted to the required destination, protocol, and port.

> This table represents the baseline security policy and does not necessarily represent every individual firewall rule.

---

# 9. Firewall Rule Design Principles

Firewall rules should follow these principles.

## Default Deny

Traffic between security zones should be denied unless there is a documented requirement.

## Least Privilege

Allow only the:

* Required source
* Required destination
* Required protocol
* Required port

## Avoid Any-to-Any Rules

Rules such as the following should be avoided:

```text
IoT → Any → Any
```

```text
Guest → Any → Any
```

Instead, define the exact communication requirement.

## Administrative Access

Management interfaces should only be accessible from designated trusted administrative devices.

## Logging

Important denied and permitted inter-VLAN traffic should be logged where practical.

Logging assists with:

* Troubleshooting
* Monitoring
* Incident investigation
* Identifying unexpected communication

---

# 10. Network Management and Address Allocation

Infrastructure devices should use predictable IP addressing and DHCP reservations or static addresses where appropriate.

A consistent addressing scheme should be maintained for:

* Firewall
* Switches
* Access points
* Hypervisors
* Servers
* NAS
* Network monitoring
* Other critical infrastructure

## Asset Register

An IP address and asset register should be maintained so that every important network device can be identified.

The register should ideally include:

| Asset | Type | VLAN | IP Address | MAC Address | Hostname | Owner | Purpose |
| ----- | ---- | ---: | ---------- | ----------- | -------- | ----- | ------- |

---

# 11. Design Rationale

## Management Isolation

The Management VLAN is intentionally small and isolated because compromise of network-management interfaces could provide control over the wider network.

## Trusted Network Separation

Trusted devices are separated from infrastructure and untrusted devices so that ordinary user devices do not automatically have administrative access.

## Server Isolation

Servers and virtual machines are placed in their own security zone to prevent user, IoT, or guest devices from obtaining unrestricted access to server infrastructure.

## IoT Isolation

IoT devices are isolated because they frequently have different security characteristics from personally managed computers and phones.

Restricting their access reduces the potential impact of a compromised device.

## Guest Isolation

Guest devices are treated as untrusted and are prevented from accessing internal systems unless a specific service is intentionally exposed.

## Default Deny

The architecture uses a default-deny approach between security zones.

New communication paths must therefore be deliberately created rather than automatically permitted.

---

# 12. Core Security Principle

> **No device should receive access to another security zone unless that access is required, understood, and explicitly permitted.**

Segmentation is therefore not based solely on VLAN membership.

It is enforced through:

* Firewall policy
* Identity
* Device trust
* Service requirements
* Least-privilege access

---

## Summary

The network architecture separates infrastructure into five security zones:

```text
                         ┌─────────────────────┐
                         │      Internet       │
                         └──────────┬──────────┘
                                    │
                              ┌─────▼─────┐
                              │ Firewall  │
                              └─────┬─────┘
                                    │
       ┌──────────────┬─────────────┼─────────────┬──────────────┐
       │              │             │             │              │
       ▼              ▼             ▼             ▼              ▼
   VLAN 10        VLAN 20       VLAN 30       VLAN 40        VLAN 50
 Management       Trusted      Lab/Servers       IoT          Guest
   /27              /24            /27            /24            /27
```

The firewall provides the central enforcement point between these zones, with **default-deny, least-privilege access** used as the fundamental security model.
