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

> **Note:** These frameworks are primarily intended for organisations. This home-lab implementation adapts relevant security principles and does not
