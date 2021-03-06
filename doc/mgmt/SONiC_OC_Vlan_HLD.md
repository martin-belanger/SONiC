# VLAN
OpenConfig support for VLAN interfaces
# High Level Design Document
#### Rev 0.1

# Table of Contents
  * [List of Tables](#list-of-tables)
  * [Revision](#revision)
  * [About This Manual](#about-this-manual)
  * [Scope](#scope)
  * [Definition/Abbreviation](#definitionabbreviation)

# List of Tables
[Table 1: Abbreviations](#table-1-abbreviations)

# Revision
| Rev |     Date    |       Author       | Change Description                |
|:---:|:-----------:|:------------------:|-----------------------------------|
| 0.1 | 09/05/2019  |   Justine Jose      | Initial version                   |

# About this Manual
This document provides information about the northbound interface details for VLANs.

# Scope
This document covers the "configuration" and "show" commands supported for VLANs based on OpenConfig yang
and unit-test cases. It does not include the protocol design or protocol implementation details.

# Definition/Abbreviation

### Table 1: Abbreviations
| **Term**                 | **Meaning**                         |
|--------------------------|-------------------------------------|
| VLAN                      | Virtual Local Area Network         |

# 1 Feature Overview
Add support for VLAN create/set/get via CLI, REST and gNMI using openconfig-interfaces.yang and sonic-mgmt-framework container

## 1.1 Requirements
Provide management framework capabilities to handle:
- VLAN creation and deletion
- MTU configuration
- IPv4 / IPv6 address configuration
- IPv4 / IPv6 address removal
- Addition of tagged / un-tagged ports to VLAN
- Removal of tagged / un-tagged ports from VLAN
- Associated show commands.

### 1.1.1 Functional Requirements

Provide management framework support to existing SONiC capabilities with respect to VLANs.

### 1.1.2 Configuration and Management Requirements
- CLI configuration and show commands
- REST API support
- gNMI Support

Details described in Section 3.

## 1.2 Design Overview

### 1.2.1 Basic Approach
Provide transformer methods in sonic-mgmt-framework container for VLAN handling

### 1.2.2 Container
All code changes will be done in management-framework container

### 1.2.3 SAI Overview
N/A

# 2 Functionality
## 2.1 Target Deployment Use Cases
Manage/configure Vlan interface via CLI, gNMI and REST interfaces
## 2.2 Functional Description
Provide CLI, gNMI and REST support for VLAN related commands handling

# 3 Design
## 3.1 Overview
Enhancing the management framework backend code and transformer methods to add support for VLAN handling

## 3.2 User Interface
### 3.2.1 Data Models
List of yang models required for VLAN interface management.
1. [openconfig-if-interfaces.yang](https://github.com/openconfig/public/blob/master/release/models/interfaces/openconfig-interfaces.yang)
2. [openconfig-if-ethernet.yang](https://github.com/openconfig/public/blob/master/release/models/interfaces/openconfig-if-ethernet.yang)
3. [openconfig-if-ip.yang](https://github.com/openconfig/public/blob/master/release/models/interfaces/openconfig-if-ip.yang)
4. [sonic-vlan.yang](https://github.com/project-arlo/sonic-mgmt-framework/blob/master/models/yang/sonic/sonic-vlan.yang)

Supported yang objects and attributes are highlighted in green:
```diff
module: openconfig-interfaces
+   +--rw interfaces
+      +--rw interface* [name]
+         +--rw name                   -> ../config/name
          +--rw config
          |  +--rw name?            string
          |  +--rw type             identityref
+         |  +--rw mtu?             uint16
          |  +--rw loopback-mode?   boolean
          |  +--rw description?     string
+         |  +--rw enabled?         boolean
          |  +--rw oc-vlan:tpid?    identityref
          +--ro state
          |  +--ro name?            string
          |  +--ro type             identityref
+         |  +--ro mtu?             uint16
          |  +--ro loopback-mode?   boolean
          |  +--ro description?     string
          |  +--ro enabled?         boolean
          |  +--ro ifindex?         uint32
+         |  +--ro admin-status     enumeration
+         |  +--ro oper-status      enumeration
          |  +--ro last-change?     oc-types:timeticks64
          |  +--ro logical?         boolean
          |  +--ro oc-vlan:tpid?    identityref
          +--rw hold-time
          |  +--rw config
          |  |  +--rw up?     uint32
          |  |  +--rw down?   uint32
          |  +--ro state
          |     +--ro up?     uint32
          |     +--ro down?   uint32
          +--rw oc-eth:ethernet
+         |  +--rw oc-vlan:switched-vlan
+         |     +--rw oc-vlan:config
          |     |  +--rw oc-vlan:interface-mode?   oc-vlan-types:vlan-mode-type
          |     |  +--rw oc-vlan:native-vlan?      oc-vlan-types:vlan-id
+         |     |  +--rw oc-vlan:access-vlan?      oc-vlan-types:vlan-id
+         |     |  +--rw oc-vlan:trunk-vlans*      union
+         |     +--ro oc-vlan:state
          |        +--ro oc-vlan:interface-mode?   oc-vlan-types:vlan-mode-type
          |        +--ro oc-vlan:native-vlan?      oc-vlan-types:vlan-id
+         |        +--ro oc-vlan:access-vlan?      oc-vlan-types:vlan-id
+         |        +--ro oc-vlan:trunk-vlans*      union
          +--rw oc-if-aggregate:aggregation
+         |  +--rw oc-vlan:switched-vlan
+         |     +--rw oc-vlan:config
          |     |  +--rw oc-vlan:interface-mode?   oc-vlan-types:vlan-mode-type
          |     |  +--rw oc-vlan:native-vlan?      oc-vlan-types:vlan-id
+         |     |  +--rw oc-vlan:access-vlan?      oc-vlan-types:vlan-id
+         |     |  +--rw oc-vlan:trunk-vlans*      union
+         |     +--ro oc-vlan:state
          |        +--ro oc-vlan:interface-mode?   oc-vlan-types:vlan-mode-type
          |        +--ro oc-vlan:native-vlan?      oc-vlan-types:vlan-id
+         |        +--ro oc-vlan:access-vlan?      oc-vlan-types:vlan-id
+         |        +--ro oc-vlan:trunk-vlans*      union
          +--rw subinterfaces
+            +--rw subinterface* [index]
+               +--rw index         -> ../config/index
+               +--rw oc-ip:ipv4
+               |  +--rw oc-ip:addresses
+               |  |  +--rw oc-ip:address* [ip]
+               |  |     +--rw oc-ip:ip        -> ../config/ip
+               |  |     +--rw oc-ip:config
+               |  |     |  +--rw oc-ip:ip?              oc-inet:ipv4-address
+               |  |     |  +--rw oc-ip:prefix-length?   uint8
+               |  |     +--ro oc-ip:state
+               |  |     |  +--ro oc-ip:ip?              oc-inet:ipv4-address
+               |  |     |  +--ro oc-ip:prefix-length?   uint8
+               +--rw oc-ip:ipv6
+                  +--rw oc-ip:addresses
+                  |  +--rw oc-ip:address* [ip]
+                  |     +--rw oc-ip:ip        -> ../config/ip
+                  |     +--rw oc-ip:config
+                  |     |  +--rw oc-ip:ip?              oc-inet:ipv6-address
+                  |     |  +--rw oc-ip:prefix-length    uint8
+                  |     +--ro oc-ip:state
+                  |     |  +--ro oc-ip:ip?              oc-inet:ipv6-address
+                  |     |  +--ro oc-ip:prefix-length    uint8
```
### 3.2.2 CLI
- sonic-vlan.yang is used for CLI #show commands.

#### 3.2.2.1 Configuration Commands

#### VLAN Creation
`interface Vlan <vlan-id>`
```
sonic(config)# interface Vlan 5
```
#### VLAN Deletion
`no interface Vlan <vlan-id>`
```
sonic(config)# no interface Vlan 5
```
#### MTU Configuration
`mtu <mtu-val>`
```
sonic(conf-if-vlan20)# mtu 2500
sonic(conf-if-vlan20)# no mtu
```
#### MTU Removal
`no mtu <mtu-val>` --> Reset to default
```
sonic(conf-if-vlan20)# no mtu
```
#### IPv4 address configuration
`ip address <IPv4-address with prefix>`
```
sonic(conf-if-vlan20)# ip address 2.2.2.2/24
```
#### IPv4 address removal
`no ip address <IPv4-address>`
```
sonic(conf-if-vlan20)# no ip address 2.2.2.2
```
#### IPv6 address configuration
`ipv6 address <IPv6-address with prefix>`
```
sonic(conf-if-vlan20)# ipv6 address a::b/64
```
#### IPv6 address removal
`no ipv6 address <IPv6-address>`
```
sonic(conf-if-vlan20)# no ipv6 address a::b
```
#### Trunk VLAN addition to Member Port (Ethernet / Port-Channel)
`switchport trunk allowed Vlan <vlan-id>`
```
sonic(conf-if-Ethernet4)# switchport trunk allowed Vlan 5
sonic(conf-if-po4)# switchport trunk allowed Vlan 5
```
#### Trunk VLAN removal from Member Port (Ethernet / Port-Channel)
`no switchport trunk allowed Vlan <vlan-id>`
```
sonic(conf-if-Ethernet4)# no switchport trunk allowed Vlan 5
sonic(conf-if-po4)# no switchport trunk allowed Vlan 5
```
#### Access VLAN addition to Member Port (Ethernet / Port-Channel)
`switchport access Vlan <vlan-id>`
```
sonic(conf-if-Ethernet4)# switchport access Vlan 5
sonic(conf-if-po4)# switchport access Vlan 5
```
#### Access VLAN removal from Member Port (Ethernet / Port-Channel)
`no switchport access Vlan`
```
sonic(conf-if-Ethernet4)# no switchport access Vlan
sonic(conf-if-po4)# no switchport access Vlan
```

#### 3.2.2.2 Show Commands
#### Display VLAN Members detail
`show Vlan`
```
Q: A - Access (Untagged), T - Tagged
    NUM       Status       Q Ports
    5         Active       T Ethernet24
    10        Inactive
    20        Inactive     A PortChannel20
```
#### Display specific VLAN Members detail
`show Vlan <vlan-id>`
```
sonic# show Vlan 5
Q: A - Access (Untagged), T - Tagged
    NUM    Status     Q Ports
    5      Active     T Ethernet24
                      T PortChannel10
                      A Ethernet20
```
#### Display VLAN information
`show interface Vlan`
```
sonic# show interface Vlan
Vlan10 is up, line protocol is up
IP MTU 2500 bytes
IPv4 address is 10.0.0.20/31
Mode of IPv4 address assignment: MANUAL
Mode of IPv6 address assignment: not-set

Vlan20 is up, line protocol is down
IP MTU 5500 bytes
Mode of IPv4 address assignment: not-set
IPv6 address is a::b/64
Mode of IPv6 address assignment: MANUAL
```
#### Display specific VLAN Information
`show interface Vlan <vlan-id>`
```sonic# show interface Vlan 10
Vlan10 is up, line protocol is up
IP MTU 2500 bytes
IPv4 address is 10.0.0.20/31
Mode of IPv4 address assignment: MANUAL
IPv6 address is a::b/64
Mode of IPv6 address assignment: MANUAL
```


#### 3.2.2.3 Debug Commands
N/A

#### 3.2.2.4 IS-CLI Compliance
N/A

### 3.2.3 REST API Support

**PATCH**
- `/openconfig-interfaces:interfaces/ interface={name}`
- `/openconfig-interfaces:interfaces/ interface={name}/config/mtu`
- `/openconfig-interfaces:interfaces/interface={name}/openconfig-if-ethernet:ethernet/openconfig-vlan:switched-vlan/config/[access-vlan | trunk-vlans]`
- `/openconfig-interfaces:interfaces/interface={name}/openconfig-if-aggregate:aggregation/openconfig-vlan:switched-vlan/config/[access-vlan | trunk-vlans]`
- `/openconfig-interfaces:interfaces/interface={name}/subinterfaces/subinterface={index}/openconfig-if-ip:ipv4/addresses/address={ip}`
- `/openconfig-interfaces:interfaces/interface={name}/subinterfaces/subinterface={index}/openconfig-if-ip:ipv6/addresses/address={ip}`


**DELETE**
- `/openconfig-interfaces:interfaces/interface={name}/openconfig-if-ethernet:ethernet/openconfig-vlan:switched-vlan/config/[access-vlan | trunk-vlans]`
- `/openconfig-interfaces:interfaces/interface={name}/openconfig-if-aggregate:aggregation/openconfig-vlan:switched-vlan/config/[access-vlan | trunk-vlans]`
- `/openconfig-interfaces:interfaces/interface={name}`
- `/openconfig-interfaces:interfaces/interface={name}/subinterfaces/subinterface={index}/openconfig-if-ip:ipv4/addresses/address={ip}`
- `/openconfig-interfaces:interfaces/interface={name}/subinterfaces/subinterface={index}/openconfig-if-ip:ipv6/addresses/address={ip}`

**GET**
- `/openconfig-interfaces:interfaces/ interface={name}/state/[admin-status | mtu | oper-status]`
- `/openconfig-interfaces:interfaces/interface={name}/openconfig-if-ethernet:ethernet/openconfig-vlan:switched-vlan/State/[access-vlan | trunk-vlans]`
- `/openconfig-interfaces:interfaces/interface={name}/openconfig-if-aggregate:aggregation/openconfig-vlan:switched-vlan/State/[access-vlan | trunk-vlans]`

# 4 Flow Diagrams
N/A

# 5 Error Handling
TBD

# 6 Serviceability and Debug
TBD

# 7 Warm Boot Support
N/A

# 8 Scalability
N/A

# 9 Unit Test
#### Configuration and Show via CLI

| Test Name | Test Description |
| :------ | :----- |
| Create VLAN | Verify VLAN is configured |
| Configure MTU for VLAN | Verify MTU is configured |
| Remove MTU for VLAN | Verify MTU is reset to default |
| Configure access VLAN  | Verify access VLAN is configured |
| Configure trunk VLAN with access VLAN Id | Error should be thrown |
| Configure access VLAN again | Verify access VLAN is present |
| Configure IPv4 address for VLAN | Verify IPv4 address is configured |
| Configure IPv6 address for VLAN | Verify IPv6 address is configured |
| Remove IPv4 address from VLAN | Verify IPv4 address is removed |
| Remove IPv6 address from VLAN | Verify IPv6 address is removed |
| Configure trunk VLAN to physical and port-channel | Verify trunk VLAN is configured |
| Configure access VLAN with trunk VLAN Id | Error should be thrown |
| Configure trunk VLAN again | Verify trunk VLAN is present |
| Remove access VLAN | Verify access VLAN is removed |
| Remove trunk VLAN | Verify trunk VLANs are removed |
| Delete an Invalid VLAN | Error should be thrown |
| Delete the VLAN | Verify the VLAN is deleted |

#### Configuration via gNMI

Same as CLI configuration test, but using gNMI SET request

#### Get configuration via gNMI

Same as CLI show test, but using gNMI GET request, verify the JSON response.

#### Configuration via REST (PATCH)

Same as CLI configuration test, but using REST request

#### Get configuration via REST (GET)

Same as CLI show test, but using REST GET request, verify the JSON response.

# 10 Internal Design Information
N/A
