Home Lab Network Segmentation and VLAN Design1. Purpose and OverviewThis document defines the network segmentation architecture for the home lab and smart-home environment.
The design is intended to provide:
Strong network segmentation and isolation
Least-privilege access between network zones
Reduced impact from compromised or vulnerable devices
Secure administration of network infrastructure
Separation of trusted, untrusted, IoT and server workloads
A simple and maintainable network architecture
Centralised control of inter-VLAN traffic through the firewall
The design follows generally accepted cybersecurity and network-design principles, including defence in depth, least privilege, network segmentation, default-deny access, and separation of administrative functions.
Where applicable, the design is informed by Australian cybersecurity guidance, including the Australian Cyber Security Centre (ACSC) Information Security Manual (ISM) and Essential Eight principles. These frameworks are primarily intended for organisations; this home-lab implementation adapts their relevant security principles rather than claiming formal compliance.
2. Network ArchitectureThe network is divided into separate security zones using VLANs.
VLANNetworkSecurity ZonePrimary Purpose10192.168.10.0/27ManagementNetwork and infrastructure administration20192.168.20.0/24TrustedTrusted user devices30192.168.30.0/27Lab/ServersServers, VMs and infrastructure workloads40192.168.40.0/24IoTSmart-home and IoT devices50192.168.50.0/27GuestGuest and untrusted user devicesThe firewall/router is responsible for enforcing communication between VLANs.
Inter-VLAN communication should be denied by default and explicitly permitted only where a documented business or operational requirement exists.
3. VLAN 10 — ManagementNetwork: 192.168.10.0/27
Subnet Mask: 255.255.255.224
Usable Addresses: 192.168.10.1 – 192.168.10.30
PurposeThe Management VLAN is the highest-trust network and is used exclusively for administration and management of network infrastructure.
It should contain only infrastructure that requires administrative access.
DevicesExamples include:
Firewall/router
Managed switches
Wireless access points
Network management interfaces
Modem/ONT management interfaces where appropriate
Other network infrastructure
Where technically possible, devices should expose their management interfaces only on this VLAN.
Access PolicyAdministrative access should follow a least-privilege model.
Trusted VLAN → Management: Blocked by default
Trusted administrator device → Management: Explicitly allowed
IoT → Management: Blocked
Guest → Management: Blocked
Lab/Servers → Management: Blocked by default
Management → Internet: Restricted
Management → other VLANs: Restricted and explicitly permitted where required
Administration should preferably be performed from a designated trusted device.
Where practical:
Use fixed/reserved IP addresses for administrative devices
Restrict management services to required protocols and ports
Use HTTPS/SSH rather than insecure management protocols
Disable unused management services
Use MFA where supported
Avoid exposing management interfaces directly to the Internet
Security ObjectiveThe Management VLAN should remain isolated even if a normal user device, IoT device, guest device or server is compromised.
4. VLAN 20 — Trusted LANNetwork: 192.168.20.0/24
Subnet Mask: 255.255.255.0
PurposeThe Trusted VLAN is the primary user network for authorised household users and trusted personal devices.
Devices connected to this network are considered trusted from a network-access perspective, but they should not automatically receive unrestricted access to every other security zone.
DevicesExamples include:
Personal laptops
Desktop/workstations
Mobile phones
Tablets
Trusted printers
Other personally managed devices
Access PolicyTrusted devices may access:
Internet
Required IoT services
Required Lab/Server services
Guest network where administrative access is required
Management network only from specifically authorised administrative devices
Default policy:
Trusted → Internet: Allowed
Trusted → IoT: Allowed where required
Trusted → Lab/Servers: Allowed where required
Trusted → Guest: Allowed only where required
Trusted → Management: Restricted
IoT → Trusted: Blocked by default
Guest → Trusted: Blocked
The Trusted VLAN should not be treated as an unrestricted administrative network.
5. VLAN 30 — Lab / ServersNetwork: 192.168.30.0/27
Subnet Mask: 255.255.255.224
Usable Addresses: 192.168.30.1 – 192.168.30.30
PurposeThe Lab/Server VLAN hosts server infrastructure, virtual machines and other services used within the home laboratory.
This network should be treated as a separate security zone because servers may provide services to multiple networks and may also contain sensitive configuration, credentials or data.
DevicesExamples include:
Proxmox hosts
Virtual machines
NAS services
TrueNAS
Home-lab applications
Internal web services
Monitoring systems
Other server infrastructure
Access PolicyAccess should be based on the specific service required rather than allowing unrestricted VLAN-to-VLAN communication.
Recommended baseline:
Trusted → Lab/Servers: Allowed for required services
Management → Lab/Servers: Allowed for administration
Lab/Servers → Internet: Restricted/controlled
IoT → Lab/Servers: Blocked by default
Guest → Lab/Servers: Blocked by default
Lab/Servers → Trusted: Blocked by default where possible
Lab/Servers → Management: Blocked by default
If an IoT device or guest device genuinely needs access to a server—for example, a TV accessing a media server—the firewall should permit only the required destination IP, port and protocol, rather than allowing access to the entire server VLAN.
Security ObjectiveThe objective is to prevent a compromise of an Internet-facing, IoT or guest device from providing unrestricted access to the server environment.
6. VLAN 40 — IoT / Smart HomeNetwork: 192.168.40.0/24
Subnet Mask: 255.255.255.0
PurposeThe IoT VLAN isolates smart-home devices from trusted user devices and infrastructure.
IoT devices may have:
Limited security controls
Infrequent firmware updates
Third-party cloud dependencies
Internet connectivity requirements
Greater exposure to supply-chain and vendor risks
Limited authentication and logging capabilities
Segmentation therefore reduces the potential impact of a compromised IoT device.
DevicesExamples include:
Smart TVs
Smart speakers
Amazon Alexa devices
Smart displays
Smart appliances
Smart lighting
Smart plugs
Other smart-home devices
Access PolicyBaseline policy:
IoT → Internet: Allowed where required
IoT → Trusted: Blocked
IoT → Management: Blocked
IoT → Lab/Servers: Blocked by default
IoT → Guest: Blocked
Trusted → IoT: Allowed where required
Guest → IoT: Allowed only for specifically required services
IoT devices should not be permitted to initiate unrestricted connections to trusted devices or infrastructure.
Where a particular IoT device requires communication with a server, allow only the required traffic.
ExampleIf a smart TV needs to access a Jellyfin server:
IoT TV → Jellyfin Server → Required TCP/UDP Port
should be allowed rather than:
IoT VLAN → Entire Server VLAN
7. VLAN 50 — GuestNetwork: 192.168.50.0/27
Subnet Mask: 255.255.255.224
Usable Addresses: 192.168.50.1 – 192.168.50.30
PurposeThe Guest VLAN provides Internet access to devices that are not under household administrative control.
Guest devices should be considered untrusted.
DevicesExamples include:
Visitors' smartphones
Visitors' laptops
Temporary devices
Other devices that should not receive access to the trusted network
Access PolicyBaseline policy:
Guest → Internet: Allowed
Guest → Trusted: Blocked
Guest → Management: Blocked
Guest → Lab/Servers: Blocked
Guest → IoT: Blocked by default
Trusted → Guest: Allowed only where required
Guest → Guest: Client isolation recommended
If guests need to control a specific IoT device, such as a television, access should be restricted to the required device and service.
For example:
Guest Device → Smart TV → Required Service
rather than:
Guest VLAN → Entire IoT VLAN
Additional Security ControlsWhere supported, enable:
Wireless client isolation
DNS filtering
Internet-only access
Rate limiting
Device/session limits
Blocking of private network access
8. Inter-VLAN Firewall PolicyThe firewall should operate on a default-deny principle for inter-VLAN traffic.
A simplified policy matrix is:
SourceManagementTrustedLab/ServersIoTGuestInternetManagement—RestrictedRestrictedRestrictedRestrictedRestrictedTrustedRestricted—Allow*Allow*Allow*AllowLab/ServersRestrictedRestricted—RestrictedRestrictedRestrictedIoTBlockBlockBlock*—BlockAllowGuestBlockBlockBlockBlock*—Allow* Access should be explicitly restricted to the required destination, protocol and port.
This table represents the baseline security policy, not necessarily every individual firewall rule.
9. Firewall Rule Design PrinciplesFirewall rules should follow these principles:
Default DenyTraffic between security zones should be denied unless there is a documented requirement.
Least PrivilegeAllow only:
Required source
Required destination
Required protocol
Required port
Avoid "Any-to-Any" RulesRules such as:
IoT → Any → Any
or
Guest → Any → Any
should be avoided.
Administrative AccessManagement interfaces should only be accessible from designated trusted administrative devices.
LoggingImportant denied and permitted inter-VLAN traffic should be logged where practical to assist with troubleshooting, monitoring and incident investigation.
10. Network Management and Address AllocationInfrastructure devices should use predictable IP addressing and DHCP reservations or static addresses where appropriate.
A consistent addressing scheme should be maintained for:
Firewall
Switches
Access points
Hypervisors
Servers
NAS
Network monitoring
Other critical infrastructure
An IP address and asset register should be maintained so that every important network device can be identified.
The register should ideally include:
AssetTypeVLANIP AddressMAC AddressHostnameOwnerPurpose
11. Design RationaleManagement IsolationThe Management VLAN is intentionally small and isolated because compromise of network-management interfaces could provide control over the wider network.
Trusted Network SeparationTrusted devices are separated from infrastructure and untrusted devices so that ordinary user devices do not automatically have administrative access.
Server IsolationServers and virtual machines are placed in their own security zone to prevent user, IoT or guest devices from obtaining unrestricted access to server infrastructure.
IoT IsolationIoT devices are isolated because they frequently have different security characteristics from personally managed computers and phones. Restricting their access reduces the potential impact of a compromised device.
Guest IsolationGuest devices are treated as untrusted and are prevented from accessing internal systems unless a specific service is intentionally exposed.
Default DenyThe architecture uses a default-deny approach between security zones. New communication paths must therefore be deliberately created rather than being automatically permitted.

Design PrincipleThe fundamental principle of this network is:
No device should receive access to another security zone unless that access is required, understood and explicitly permitted.
Segmentation is therefore not only based on VLAN membership; it is enforced through firewall policy, identity, device trust, service requirements and least-privilege access.
