# VXLAN-Multifabric

### Цель:
Настроить две независимые VXLAN фабрики с использованием протокола iBGP
Обьединить две фабрики по технологии Multifabric (VLAN Hand-off, VRF Hand-off)
как наиболее распостраненное решение у отечественных вендоров

### Принципы назначения IP адресов, адресное пространство
Описаны в документе: [IP_Plan.md](IP_Plan.md)

### Итоговая схема
![Topology.png](Topology.png)

## Конфигурации устройств в DataCenter1 (DC01):

<details>
  <summary> SPINE01 </summary>

```
dc01-pod01-spine01#show running-config
! Command: show running-config
! device: dc01-pod01-spine01 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
username Denis secret sha512 $6$y4fXjrcDJ1eVI4aJ$JCiZRs.t0Kx0f3BRqUioTguW5BhatcFeAsKQQkUQaOWS7LWhHXRRKGyeSLP8xF/aU3LOubedln.MaGkP91arx.
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname dc01-pod01-spine01
!
spanning-tree mode mstp
!
interface Ethernet1
   description =LINK-TO_LEAF01=
   no switchport
   ip address 10.11.3.0/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet2
   description =LINK-TO_LEAF02=
   no switchport
   ip address 10.11.3.2/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet3
   description =LINK_TO_LEAF03=
   no switchport
   ip address 10.11.3.4/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet4
   description =LINK-TO_LEAF04=
   no switchport
   ip address 10.11.3.6/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet5
   description =LINK-TO_SLEAF01=
   no switchport
   ip address 10.11.3.8/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet6
   description =LINK-TO_SLEAF02=
   no switchport
   ip address 10.11.3.10/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet7
   description =LINK-TO_BLEAF01=
   no switchport
   ip address 10.11.3.12/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet8
   description =LINK_TO_BLEAF02=
   no switchport
   ip address 10.11.3.14/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet9
   description =LINK-TO_BGW01=
   no switchport
   ip address 10.11.3.16/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet10
   description =LINK_TO_BGW02=
   no switchport
   ip address 10.11.3.18/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet11
!
interface Loopback1
   ip address 10.11.1.1/32
!
interface Loopback2
   ip address 10.11.2.1/32
!
interface Management1
!
ip routing
!
ip prefix-list PL_1
   seq 10 permit 10.11.1.0/24 le 32
   seq 20 permit 10.11.3.0/24 le 32
!
route-map MAP_1 permit 10
   match ip address prefix-list PL_1
!
router bgp 4259840001
   router-id 10.11.1.1
   graceful-restart restart-time 300
   maximum-paths 4 ecmp 64
   neighbor OVERLAY peer group
   neighbor OVERLAY next-hop-unchanged
   neighbor OVERLAY update-source Loopback1
   neighbor OVERLAY ebgp-multihop 3
   neighbor OVERLAY send-community extended
   neighbor 10.11.1.3 peer group OVERLAY
   neighbor 10.11.1.3 remote-as 4259905000
   neighbor 10.11.1.3 description LEAF01_Lo1
   neighbor 10.11.1.3 ebgp-multihop
   neighbor 10.11.1.4 peer group OVERLAY
   neighbor 10.11.1.4 remote-as 4259905001
   neighbor 10.11.1.4 description LEAF02_Lo1
   neighbor 10.11.1.5 peer group OVERLAY
   neighbor 10.11.1.5 remote-as 4259905002
   neighbor 10.11.1.5 description LEAF03_Lo1
   neighbor 10.11.1.6 peer group OVERLAY
   neighbor 10.11.1.6 remote-as 4259905003
   neighbor 10.11.1.6 description LEAF04_Lo1
   neighbor 10.11.1.7 peer group OVERLAY
   neighbor 10.11.1.7 remote-as 4259905101
   neighbor 10.11.1.7 description SLEAF01_Lo1
   neighbor 10.11.1.8 peer group OVERLAY
   neighbor 10.11.1.8 remote-as 4259905102
   neighbor 10.11.1.8 description SLEAF02_Lo1
   neighbor 10.11.1.9 peer group OVERLAY
   neighbor 10.11.1.9 remote-as 4259905201
   neighbor 10.11.1.9 description BLEAF01_Lo1
   neighbor 10.11.1.10 peer group OVERLAY
   neighbor 10.11.1.10 remote-as 4259905202
   neighbor 10.11.1.10 description BLEAF02_Lo1
   neighbor 10.11.1.11 peer group OVERLAY
   neighbor 10.11.1.11 remote-as 4259905301
   neighbor 10.11.1.11 description BGW01_Lo1
   neighbor 10.11.1.12 peer group OVERLAY
   neighbor 10.11.1.12 remote-as 4259905302
   neighbor 10.11.1.12 description BGW02_Lo1
   neighbor 10.11.3.1 remote-as 4259905000
   neighbor 10.11.3.1 bfd
   neighbor 10.11.3.1 description LEAF01
   neighbor 10.11.3.1 route-map MAP_1 in
   neighbor 10.11.3.1 password 7 m3FjC6w2d6o=
   neighbor 10.11.3.3 remote-as 4259905001
   neighbor 10.11.3.3 bfd
   neighbor 10.11.3.3 description LEAF02
   neighbor 10.11.3.3 route-map MAP_1 in
   neighbor 10.11.3.3 password 7 kZcBlBo8kLo=
   neighbor 10.11.3.5 remote-as 4259905002
   neighbor 10.11.3.5 bfd
   neighbor 10.11.3.5 description LEAF03
   neighbor 10.11.3.5 route-map MAP_1 in
   neighbor 10.11.3.5 password 7 kGZWqyCmNl4=
   neighbor 10.11.3.7 remote-as 4259905003
   neighbor 10.11.3.7 bfd
   neighbor 10.11.3.7 description LEAF04
   neighbor 10.11.3.7 route-map MAP_1 in
   neighbor 10.11.3.7 password 7 JVIkOCtPnDM=
   neighbor 10.11.3.9 remote-as 4259905101
   neighbor 10.11.3.9 bfd
   neighbor 10.11.3.9 description SLEAF01
   neighbor 10.11.3.9 route-map MAP_1 in
   neighbor 10.11.3.9 password 7 mGBWW5zgOEE=
   neighbor 10.11.3.11 remote-as 4259905102
   neighbor 10.11.3.11 bfd
   neighbor 10.11.3.11 description SLEAF02
   neighbor 10.11.3.11 route-map MAP_1 in
   neighbor 10.11.3.11 password 7 jSSRC/e2y7E=
   neighbor 10.11.3.13 remote-as 4259905201
   neighbor 10.11.3.13 bfd
   neighbor 10.11.3.13 description BLEAF01
   neighbor 10.11.3.13 route-map MAP_1 in
   neighbor 10.11.3.13 password 7 IVk4JXzOQBg=
   neighbor 10.11.3.15 remote-as 4259905202
   neighbor 10.11.3.15 bfd
   neighbor 10.11.3.15 description BLEAF02
   neighbor 10.11.3.15 route-map MAP_1 in
   neighbor 10.11.3.15 password 7 7aYBVA/JiU8=
   neighbor 10.11.3.17 remote-as 4259905301
   neighbor 10.11.3.17 bfd
   neighbor 10.11.3.17 description BGW01
   neighbor 10.11.3.17 route-map MAP_1 in
   neighbor 10.11.3.17 password 7 e01wILS8YqY=
   neighbor 10.11.3.19 remote-as 4259905302
   neighbor 10.11.3.19 bfd
   neighbor 10.11.3.19 description BGW02
   neighbor 10.11.3.19 route-map MAP_1 in
   neighbor 10.11.3.19 password 7 hBf7gSNWhXU=
   !
   address-family evpn
      neighbor OVERLAY activate
   !
   address-family ipv4
      network 10.11.1.1/32
      network 10.11.3.0/24
      graceful-restart
!
end

```
</details> 

<details>
  <summary>SPINE02 </summary>

```
dc01-pod01-spine02#show running-config
! Command: show running-config
! device: dc01-pod01-spine02 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname dc01-pod01-spine02
!
spanning-tree mode mstp
!
interface Ethernet1
   description =LINK-TO_LEAF01=
   no switchport
   ip address 10.11.3.40/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet2
   description =LINK_TO_LEAF02=
   no switchport
   ip address 10.11.3.42/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet3
   description =LINK_TO_LEAF03=
   no switchport
   ip address 10.11.3.44/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet4
   description =LINK_TO_LEAF04=
   no switchport
   ip address 10.11.3.46/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet5
   description =LINK_TO_SLEAF01=
   no switchport
   ip address 10.11.3.48/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet6
   description =LINK_TO_SLEAF02=
   no switchport
   ip address 10.11.3.50/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet7
   description =LINK_TO_BLEAF01=
   no switchport
   ip address 10.11.3.52/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet8
   description =LINK_TO_BLEAF02=
   no switchport
   ip address 10.11.3.54/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet9
   description =LINK_TO_BGW01=
   no switchport
   ip address 10.11.3.56/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet10
   description =LINK_TO_BGW02=
   no switchport
   ip address 10.11.3.58/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet11
!
interface Loopback1
   ip address 10.11.1.2/32
!
interface Management1
!
ip routing
!
ip prefix-list PL_1
   seq 10 permit 10.11.1.0/24 le 32
   seq 20 permit 10.11.3.0/24 le 32
!
route-map MAP_1 permit 10
   match ip address prefix-list PL_1
!
router bgp 4259840002
   router-id 10.11.1.2
   graceful-restart restart-time 300
   maximum-paths 4 ecmp 64
   neighbor OVERLAY peer group
   neighbor OVERLAY next-hop-unchanged
   neighbor OVERLAY update-source Loopback1
   neighbor OVERLAY ebgp-multihop 3
   neighbor OVERLAY send-community extended
   neighbor 10.11.1.3 peer group OVERLAY
   neighbor 10.11.1.3 remote-as 4259905000
   neighbor 10.11.1.3 description LEAF01_Lo1
   neighbor 10.11.1.3 ebgp-multihop 2
   neighbor 10.11.1.4 peer group OVERLAY
   neighbor 10.11.1.4 remote-as 4259905001
   neighbor 10.11.1.4 description LEAF02_Lo1
   neighbor 10.11.1.5 peer group OVERLAY
   neighbor 10.11.1.5 remote-as 4259905002
   neighbor 10.11.1.5 description LEAF03_Lo1
   neighbor 10.11.1.6 peer group OVERLAY
   neighbor 10.11.1.6 remote-as 4259905003
   neighbor 10.11.1.6 description LEAF04_Lo1
   neighbor 10.11.1.7 peer group OVERLAY
   neighbor 10.11.1.7 remote-as 4259905101
   neighbor 10.11.1.7 description SLEAF01_Lo1
   neighbor 10.11.1.8 peer group OVERLAY
   neighbor 10.11.1.8 remote-as 4259905102
   neighbor 10.11.1.8 description SLEAF02_Lo1
   neighbor 10.11.1.9 peer group OVERLAY
   neighbor 10.11.1.9 remote-as 4259905201
   neighbor 10.11.1.9 description BLEAF01_Lo1
   neighbor 10.11.1.10 peer group OVERLAY
   neighbor 10.11.1.10 remote-as 4259905202
   neighbor 10.11.1.10 description BLEAF02_Lo1
   neighbor 10.11.1.11 peer group OVERLAY
   neighbor 10.11.1.11 remote-as 4259905301
   neighbor 10.11.1.11 description BGW01_Lo1
   neighbor 10.11.1.12 peer group OVERLAY
   neighbor 10.11.1.12 remote-as 4259905302
   neighbor 10.11.1.12 description BGW02_Lo1
   neighbor 10.11.3.41 remote-as 4259905000
   neighbor 10.11.3.41 bfd
   neighbor 10.11.3.41 description LEAF01
   neighbor 10.11.3.41 route-map MAP_1 in
   neighbor 10.11.3.41 password 7 X1D+FhPL62M=
   neighbor 10.11.3.43 remote-as 4259905001
   neighbor 10.11.3.43 bfd
   neighbor 10.11.3.43 description LEAF02
   neighbor 10.11.3.43 route-map MAP_1 in
   neighbor 10.11.3.43 password 7 v+YQEIAfTLQ=
   neighbor 10.11.3.45 remote-as 4259905002
   neighbor 10.11.3.45 bfd
   neighbor 10.11.3.45 description LEAF03
   neighbor 10.11.3.45 route-map MAP_1 in
   neighbor 10.11.3.45 password 7 48bUQaGsEyA=
   neighbor 10.11.3.47 remote-as 4259905003
   neighbor 10.11.3.47 bfd
   neighbor 10.11.3.47 description LEAF04
   neighbor 10.11.3.47 route-map MAP_1 in
   neighbor 10.11.3.47 password 7 2L3nuAWmD7Y=
   neighbor 10.11.3.49 remote-as 4259905101
   neighbor 10.11.3.49 bfd
   neighbor 10.11.3.49 description SLEAF01
   neighbor 10.11.3.49 route-map MAP_1 in
   neighbor 10.11.3.49 password 7 69qWpBfLKT0=
   neighbor 10.11.3.51 remote-as 4259905102
   neighbor 10.11.3.51 bfd
   neighbor 10.11.3.51 description SLEAF02
   neighbor 10.11.3.51 route-map MAP_1 in
   neighbor 10.11.3.51 password 7 X1D+FhPL62M=
   neighbor 10.11.3.53 remote-as 4259905201
   neighbor 10.11.3.53 bfd
   neighbor 10.11.3.53 description BLEAF01
   neighbor 10.11.3.53 route-map MAP_1 in
   neighbor 10.11.3.53 password 7 v+YQEIAfTLQ=
   neighbor 10.11.3.55 remote-as 4259905202
   neighbor 10.11.3.55 bfd
   neighbor 10.11.3.55 description BLEAF02
   neighbor 10.11.3.55 route-map MAP_1 in
   neighbor 10.11.3.55 password 7 48bUQaGsEyA=
   neighbor 10.11.3.57 remote-as 4259905301
   neighbor 10.11.3.57 bfd
   neighbor 10.11.3.57 description BGW01
   neighbor 10.11.3.57 route-map MAP_1 in
   neighbor 10.11.3.57 password 7 2L3nuAWmD7Y=
   neighbor 10.11.3.59 remote-as 4259905302
   neighbor 10.11.3.59 bfd
   neighbor 10.11.3.59 description BGW02
   neighbor 10.11.3.59 route-map MAP_1 in
   neighbor 10.11.3.59 password 7 69qWpBfLKT0=
   !
   address-family evpn
      neighbor OVERLAY activate
   !
   address-family ipv4
      neighbor 10.11.3.41 activate
      neighbor 10.11.3.43 activate
      neighbor 10.11.3.45 activate
      neighbor 10.11.3.47 activate
      neighbor 10.11.3.49 activate
      neighbor 10.11.3.51 activate
      neighbor 10.11.3.53 activate
      neighbor 10.11.3.55 activate
      neighbor 10.11.3.57 activate
      neighbor 10.11.3.59 activate
      network 10.11.1.2/32
      network 10.11.3.0/24
      graceful-restart
!
end
dc01-pod01-spine02#
```
</details> 

<details>
  <summary>LEAF01 </summary>

```
dc01-pod01-leaf01#show running-config
! Command: show running-config
! device: dc01-pod01-leaf01 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
logging synchronous level critical
!
hostname dc01-pod01-leaf01
!
spanning-tree mode mstp
no spanning-tree vlan-id 4000-4001
!
vlan 10
   name =VLAN_10_test=
!
vlan 20
   name =VLAN20_Assimetric_IRB=
!
vlan 30
   name =VLAN_30_Symmetric_IRB_SVI_L3VNI
!
vlan 40
   name =VLAN_40_Symmetric_IRB_L3VNI=
!
vlan 201
   name TEST_Connect
   trunk group MLAG_Peer
!
vlan 202
!
vlan 301
   name GREEN
!
vlan 302
   name RED
!
vlan 4000
   name =MLAG_Peer_VLAN=
   trunk group MLAG_Peer
!
vlan 4001
   name -VLAN_LEAF-to_LEAF
   trunk group MLAG_Peer
!
vrf instance CUSTOMER_L3VNI
!
vrf instance GREEN
!
vrf instance RED
!
interface Port-Channel4000
   description =MLAG_Peer=
   switchport mode trunk
   switchport trunk group MLAG_Peer
   spanning-tree link-type point-to-point
!
interface Ethernet1
   description =LINK-TO_SPINE01=
   no switchport
   ip address 10.11.3.1/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet2
   description =LINK-TO_SPINE02=
   no switchport
   ip address 10.11.3.41/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet3
   description =HOST=
   switchport access vlan 201
!
interface Ethernet4
!
interface Ethernet5
   description =MLAG_peer_link=
   channel-group 4000 mode active
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback1
   ip address 10.11.1.3/32
!
interface Loopback2
   ip address 10.11.2.3/32
!
interface Management1
   ip address 172.16.0.1/24
!
interface Vlan10
   ip address 10.88.88.2/24
   ip virtual-router address 10.88.88.1/24
!
interface Vlan20
   ip address 10.10.10.2/24
   ip virtual-router address 10.10.10.1/24
!
interface Vlan30
   vrf CUSTOMER_L3VNI
   ip address 10.30.30.3/24
   ip virtual-router address 10.30.30.1/24
!
interface Vlan40
   vrf CUSTOMER_L3VNI
   ip address 10.40.40.3/24
   ip virtual-router address 10.40.40.1/24
!
interface Vlan201
   vrf CUSTOMER_L3VNI
   ip address 1.1.1.222/24
   ip virtual-router address 1.1.1.254/24
!
interface Vlan301
   vrf GREEN
   ip address 192.168.1.10/24
   ip virtual-router address 192.168.1.1/24
!
interface Vlan302
   vrf RED
   ip address 192.168.2.10/24
   ip virtual-router address 192.168.2.1/24
!
interface Vlan4000
   no autostate
   ip address 10.40.0.254/31
!
interface Vlan4001
   mtu 9214
   ip address 10.0.3.0/31
!
interface Vxlan1
   description =VXLAN=
   vxlan source-interface Loopback2
   vxlan udp-port 4789
   vxlan vlan 10 vni 100010
   vxlan vlan 20 vni 100020
   vxlan vlan 30 vni 100030
   vxlan vlan 40 vni 100040
   vxlan vlan 201 vni 100201
   vxlan vlan 301 vni 1000301
   vxlan vlan 302 vni 1000302
   vxlan vrf CUSTOMER_L3VNI vni 1000777
   vxlan vrf GREEN vni 1000888
   vxlan vrf RED vni 1000999
   vxlan learn-restrict any
!
ip virtual-router mac-address ba:be:fa:ce:fa:de
!
ip routing
ip routing vrf CUSTOMER_L3VNI
ip routing vrf GREEN
ip routing vrf RED
!
ip prefix-list PL_1
   seq 10 permit 10.11.1.0/24 le 32
   seq 20 permit 10.11.3.0/24 le 32
!
mlag configuration
   domain-id LEAF01-02
   local-interface Vlan4000
   peer-address 10.40.0.255
   peer-address heartbeat 172.16.0.2 vrf mgmt
   peer-link Port-Channel4000
   dual-primary detection delay 10 action errdisable all-interfaces
!
route-map MAP_1 permit 10
   match ip address prefix-list PL_1
!
router bgp 4259905000
   router-id 10.11.1.3
   graceful-restart restart-time 300
   maximum-paths 4 ecmp 64
   neighbor EVPN peer group
   neighbor EVPN remote-as 4259840001
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor EVPN2 peer group
   neighbor EVPN2 remote-as 4259840002
   neighbor EVPN2 update-source Loopback1
   neighbor EVPN2 ebgp-multihop 3
   neighbor EVPN2 send-community extended
   neighbor underlay_ibgp peer group
   neighbor underlay_ibgp remote-as 4259905001
   neighbor underlay_ibgp maximum-routes 12000 warning-only
   neighbor 10.0.3.1 peer group underlay_ibgp
   neighbor 10.11.1.1 peer group EVPN
   neighbor 10.11.1.1 description SPINE01_Lo1
   neighbor 10.11.1.2 peer group EVPN2
   neighbor 10.11.1.2 description SPINE02_Lo1
   neighbor 10.11.3.0 remote-as 4259840001
   neighbor 10.11.3.0 bfd
   neighbor 10.11.3.0 description SPINE01
   neighbor 10.11.3.0 route-map MAP_1 in
   neighbor 10.11.3.0 password 7 m3FjC6w2d6o=
   neighbor 10.11.3.40 remote-as 4259840002
   neighbor 10.11.3.40 bfd
   neighbor 10.11.3.40 description SPINE02
   neighbor 10.11.3.40 route-map MAP_1 in
   neighbor 10.11.3.40 password 7 X1D+FhPL62M=
   !
   vlan 10
      rd auto
      route-target both 65001:65001
      redistribute learned
      redistribute static
   !
   vlan 20
      rd auto
      route-target both 65001:65001
      redistribute learned
      redistribute static
   !
   vlan 201
      rd 0.0.0.1:1
      route-target both 100:100
      redistribute learned
      redistribute static
   !
   vlan 30
      rd 65001:100030
      route-target both 65001:30
      redistribute learned
      redistribute static
   !
   vlan 301
      rd 65001:1000301
      route-target both 301:301
   !
   vlan 302
      rd 65001:1000302
      route-target both 302:302
   !
   vlan 40
      rd 65001:100040
      route-target both 65001:40
      redistribute learned
      redistribute static
   !
   address-family evpn
      neighbor EVPN activate
      neighbor EVPN2 activate
   !
   address-family ipv4
      network 10.11.1.3/32
      network 10.11.2.3/32
      network 10.11.3.0/31
      network 10.11.3.40/31
      graceful-restart
   !
   vrf CUSTOMER_L3VNI
      rd 10.11.1.4:1
      route-target import evpn 65000:1
      route-target export evpn 65000:1
      redistribute connected
   !
   vrf GREEN
      rd 1000888:888
      route-target import evpn 301:1000888
      route-target export evpn 301:1000888
   !
   vrf RED
      rd 1000999:999
      route-target import evpn 302:1000999
      route-target export evpn 302:1000999
!
end
dc01-pod01-leaf01#
```
</details> 

<details>
  <summary>LEAF02 </summary>

```
dc01-pod01-leaf02#show running-config
! Command: show running-config
! device: dc01-pod01-leaf02 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
logging synchronous level critical
!
hostname dc01-pod01-leaf02
!
spanning-tree mode mstp
no spanning-tree vlan-id 4000-4001
!
vlan 10
   name -=VLAN_10=-
!
vlan 20
!
vlan 30
   name =VLAN_30_Symmetric_IRB_SVI_L3VNI
!
vlan 40
   name =VLAN_40_Symmetric_IRB_L3VNI=
!
vlan 4000
   name =MLAG_Peer_VLAN=
   trunk group MLAG_Peer
!
vlan 4001
   name =VLAN_LEAF_to_Leaf
   trunk group MLAG_Peer
   trunk group mlag-peer
!
vrf instance CUSTOMER_L3VNI
!
interface Port-Channel4000
   description =MLAG_Peer=
   switchport mode trunk
   switchport trunk group MLAG_Peer
   spanning-tree link-type point-to-point
!
interface Ethernet1
   description =LINK_TO_SPINE01=
   no switchport
   ip address 10.11.3.3/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet2
   description =LINK_TO_SPINE02=
   no switchport
   ip address 10.11.3.43/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
   description =MLAG_peer_link=
   channel-group 4000 mode active
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback1
   ip address 10.11.1.4/32
!
interface Loopback2
   ip address 10.11.2.3/32
!
interface Management1
   ip address 172.16.0.2/24
!
interface Vlan10
   ip virtual-router address 10.88.88.1/24
!
interface Vlan20
   ip address 10.10.10.3/24
   ip virtual-router address 10.10.10.1/24
!
interface Vlan30
   vrf CUSTOMER_L3VNI
   ip address 10.30.30.2/24
   ip virtual-router address 10.30.30.1/24
!
interface Vlan40
   vrf CUSTOMER_L3VNI
   ip address 10.40.40.2/24
   ip virtual-router address 10.40.40.1/24
!
interface Vlan4000
   ip address 10.40.0.255/31
!
interface Vlan4001
   mtu 9214
   ip address 10.0.3.1/31
!
interface Vxlan1
   description =VXLAN=
   vxlan source-interface Loopback2
   vxlan udp-port 4789
   vxlan vlan 10 vni 100010
   vxlan vlan 20 vni 100020
   vxlan vlan 30 vni 100030
   vxlan vlan 40 vni 100040
   vxlan vrf CUSTOMER_L3VNI vni 1000777
   vxlan learn-restrict any
!
ip virtual-router mac-address ba:be:fa:ce:fa:de
!
ip routing
ip routing vrf CUSTOMER_L3VNI
!
ip prefix-list PL_1
   seq 10 permit 10.11.1.0/24 le 32
   seq 20 permit 10.11.3.0/24 le 32
!
mlag configuration
   domain-id LEAF01-02
   local-interface Vlan4000
   peer-address 10.40.0.254
   peer-address heartbeat 172.16.0.1 vrf mgmt
   peer-link Port-Channel4000
   dual-primary detection delay 10 action errdisable all-interfaces
!
route-map MAP_1 permit 10
   match ip address prefix-list PL_1
!
router bgp 4259905001
   router-id 10.11.1.4
   graceful-restart restart-time 300
   maximum-paths 4 ecmp 64
   neighbor EVPN peer group
   neighbor EVPN remote-as 4259840001
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor EVPN2 peer group
   neighbor EVPN2 remote-as 4259840002
   neighbor EVPN2 update-source Loopback1
   neighbor EVPN2 ebgp-multihop 3
   neighbor EVPN2 send-community extended
   neighbor underlay_ibgp peer group
   neighbor underlay_ibgp remote-as 4259905000
   neighbor underlay_ibgp maximum-routes 12000 warning-only
   neighbor 10.0.3.0 peer group underlay_ibgp
   neighbor 10.11.1.1 peer group EVPN
   neighbor 10.11.1.1 description SPINE01_Lo1
   neighbor 10.11.1.2 peer group EVPN2
   neighbor 10.11.1.2 description SPINE02_Lo1
   neighbor 10.11.3.2 remote-as 4259840001
   neighbor 10.11.3.2 bfd
   neighbor 10.11.3.2 description SPINE01
   neighbor 10.11.3.2 route-map MAP_1 in
   neighbor 10.11.3.2 password 7 kZcBlBo8kLo=
   neighbor 10.11.3.42 remote-as 4259840002
   neighbor 10.11.3.42 bfd
   neighbor 10.11.3.42 description SPINE02
   neighbor 10.11.3.42 route-map MAP_1 in
   neighbor 10.11.3.42 password 7 v+YQEIAfTLQ=
   !
   vlan 10
      rd 65001:10010
      route-target both 65001:65001
      redistribute learned
      redistribute static
   !
   vlan 20
      rd auto
      route-target both 65001:65001
      redistribute learned
      redistribute static
   !
   vlan 30
      rd 65001:100030
      route-target both 65001:30
      redistribute learned
      redistribute static
   !
   vlan 40
      rd 65001:100040
      route-target both 65001:40
      redistribute learned
      redistribute static
   !
   address-family evpn
      neighbor EVPN activate
      neighbor EVPN2 activate
   !
   address-family ipv4
      network 10.11.1.4/32
      network 10.11.2.3/32
      network 10.11.3.2/31
      network 10.11.3.42/31
      network 10.11.3.0/24
      graceful-restart
   !
   vrf CUSTOMER_L3VNI
      rd 10.11.1.4:1
      route-target import evpn 65000:1
      route-target export evpn 65000:1
      redistribute connected
!
end
dc01-pod01-leaf02#
```
</details> 

<details>
  <summary> LEAF03 </summary>

```
dc01-pod01-leaf03#show running-config
! Command: show running-config
! device: dc01-pod01-leaf03 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
logging synchronous level critical
!
hostname dc01-pod01-leaf03
!
spanning-tree mode mstp
!
vlan 10
   name VLAN_10_Test
!
vlan 20
   name =VLAN20__Assimetric_IRB=
!
vlan 201-202
   name =ESI_CUSTOMR
!
vlan 301
   name GREEN
!
vlan 302
   name RED
!
vrf instance CUSTOMER_L3VNI
!
vrf instance GREEN
!
vrf instance RED
!
interface Port-Channel1
   switchport trunk allowed vlan 201-202,301-302
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:babe:face:fade:bace
      route-target import ba:be:fa:ce:ba:ce
   lacp system-id fade.babe.face
!
interface Ethernet1
   description =LINK_TO_SPINE01=
   mtu 9214
   no switchport
   ip address 10.11.3.5/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet2
   description =LINK_TO_SPINE02=
   mtu 9214
   no switchport
   ip address 10.11.3.45/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet3
   description =TEST3=
   mtu 9214
   switchport access vlan 201
!
interface Ethernet4
   description =ESI_LAG=
   mtu 9214
   channel-group 1 mode active
!
interface Ethernet5
   mtu 9214
!
interface Ethernet6
   mtu 9214
!
interface Ethernet7
   mtu 9214
!
interface Ethernet8
   mtu 9214
!
interface Loopback1
   ip address 10.11.1.5/32
!
interface Loopback2
   ip address 10.11.2.5/32
!
interface Management1
!
interface Vlan10
   ip address 10.88.88.3/24
   ip virtual-router address 10.88.88.1/24
!
interface Vlan20
   ip address 10.10.10.3/24
   ip virtual-router address 10.10.10.1/24
!
interface Vlan201
   vrf CUSTOMER_L3VNI
   ip address 1.1.1.2/24
   ip virtual-router address 1.1.1.254/24
!
interface Vlan202
   vrf CUSTOMER_L3VNI
   ip address 2.2.2.3/24
   ip virtual-router address 2.2.2.254/24
!
interface Vlan301
   vrf GREEN
   ip address 192.168.1.39/24
   ip virtual-router address 192.168.1.1/24
!
interface Vlan302
   vrf RED
   ip address 192.168.2.39/24
   ip virtual-router address 192.168.2.1/24
!
interface Vxlan1
   description =VXLAN=
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 100010
   vxlan vlan 20 vni 100020
   vxlan vlan 201 vni 100201
   vxlan vlan 202 vni 100202
   vxlan vlan 301 vni 1000301
   vxlan vlan 302 vni 1000302
   vxlan vrf CUSTOMER_L3VNI vni 1000777
   vxlan vrf GREEN vni 1000888
   vxlan vrf RED vni 1000999
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:10
!
ip routing
ip routing vrf CUSTOMER_L3VNI
ip routing vrf GREEN
ip routing vrf RED
!
ip prefix-list PL_1
   seq 10 permit 10.11.1.0/24 le 32
   seq 20 permit 10.11.3.0/24 le 32
!
route-map MAP_1 permit 10
   match ip address prefix-list PL_1
!
router bgp 4259905002
   router-id 10.11.1.5
   graceful-restart restart-time 300
   maximum-paths 4 ecmp 64
   neighbor EVPN peer group
   neighbor EVPN remote-as 4259840001
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor EVPN2 peer group
   neighbor EVPN2 remote-as 4259840002
   neighbor EVPN2 update-source Loopback1
   neighbor EVPN2 ebgp-multihop 3
   neighbor EVPN2 send-community extended
   neighbor 10.11.1.1 peer group EVPN
   neighbor 10.11.1.1 description SPINE01_Lo1
   neighbor 10.11.1.2 peer group EVPN2
   neighbor 10.11.1.2 description SPINE02_Lo1
   neighbor 10.11.3.4 remote-as 4259840001
   neighbor 10.11.3.4 bfd
   neighbor 10.11.3.4 description SPINE01
   neighbor 10.11.3.4 route-map MAP_1 in
   neighbor 10.11.3.4 password 7 kGZWqyCmNl4=
   neighbor 10.11.3.44 remote-as 4259840002
   neighbor 10.11.3.44 bfd
   neighbor 10.11.3.44 description SPINE02
   neighbor 10.11.3.44 route-map MAP_1 in
   neighbor 10.11.3.44 password 7 48bUQaGsEyA=
   !
   vlan 10
      rd auto
      route-target both 65001:65001
      redistribute learned
      redistribute static
   !
   vlan 20
      rd auto
      route-target both 65001:65001
      redistribute learned
      redistribute static
   !
   vlan 201
      rd 0.0.0.1:1
      route-target both 100:100
      redistribute learned
   !
   vlan 202
      rd 0.0.0.2:2
      route-target both 100:100
      redistribute learned
   !
   vlan 301
      rd 65001:1000301
      route-target both 301:301
   !
   vlan 302
      rd 65001:1000302
      route-target both 302:302
   !
   address-family evpn
      neighbor EVPN activate
      neighbor EVPN2 activate
   !
   address-family ipv4
      neighbor 10.11.3.4 activate
      neighbor 10.11.3.44 activate
      network 10.11.1.5/32
      network 10.11.2.5/32
      graceful-restart
   !
   vrf CUSTOMER_L3VNI
      rd 10.11.1.5:1
      route-target import evpn 65000:1
      route-target export evpn 65000:1
      redistribute connected
   !
   vrf GREEN
      rd 1000888:888
      route-target import evpn 301:1000888
      route-target export evpn 301:1000888
      neighbor 192.168.1.200 remote-as 65500
      neighbor 192.168.1.200 description GREEN_VRF-GW
      !
      address-family ipv4
         neighbor 192.168.1.200 activate
   !
   vrf RED
      rd 1000999:999
      route-target import evpn 302:1000999
      route-target export evpn 302:1000999
      neighbor 192.168.2.200 remote-as 65500
      neighbor 192.168.2.200 description RED_VRF-GW
      !
      address-family ipv4
         neighbor 192.168.2.200 activate
!
end
```
</details> 

<details>
  <summary>LEAF04 </summary>

```
dc01-pod01-leaf04#show running-config
! Command: show running-config
! device: dc01-pod01-leaf04 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
logging synchronous level critical
!
hostname dc01-pod01-leaf04
!
spanning-tree mode mstp
!
vlan 30
   name =VLAN_30_Symmetric_IRB_SVI_L3VNI
!
vlan 40
   name =VLAN_40_Symmetric_IRB_L3VNI=
!
vlan 201-202
   name =LAG_Customer=
!
vlan 301
   name GREEN
!
vlan 302
   name RED
!
vlan 501-502
!
vrf instance CUSTOMER_L3VNI
!
vrf instance GREEN
!
vrf instance RED
!
interface Port-Channel1
   switchport trunk allowed vlan 201-202,301-302
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:babe:face:fade:bace
      route-target import ba:be:fa:ce:ba:ce
   lacp system-id fade.babe.face
!
interface Ethernet1
   description =LINK_TO_SPINE01=
   mtu 9214
   no switchport
   ip address 10.11.3.7/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet2
   description =LINK_TO_SPINE02=
   mtu 9214
   no switchport
   ip address 10.11.3.47/31
   bfd interval 700 min-rx 500 multiplier 3
   no ip ospf neighbor bfd
   no isis bfd
!
interface Ethernet3
   mtu 9214
   switchport access vlan 30
!
interface Ethernet4
   description =ESI_LAG=
   mtu 9214
   channel-group 1 mode active
!
interface Ethernet5
   mtu 9214
!
interface Ethernet6
   mtu 9214
!
interface Ethernet7
   mtu 9214
!
interface Ethernet8
   mtu 9214
!
interface Loopback1
   ip address 10.11.1.6/32
!
interface Management1
!
interface Vlan30
   vrf CUSTOMER_L3VNI
   ip address 10.30.30.4/24
   ip virtual-router address 10.30.30.1/24
!
interface Vlan40
   vrf CUSTOMER_L3VNI
   ip address 10.40.40.4/24
   ip virtual-router address 10.40.40.1/24
!
interface Vlan201
   vrf CUSTOMER_L3VNI
   ip address 1.1.1.3/24
   ip virtual-router address 1.1.1.254/24
!
interface Vlan202
   vrf CUSTOMER_L3VNI
   ip address 2.2.2.1/24
   ip virtual-router address 2.2.2.254/24
!
interface Vlan301
   vrf GREEN
   ip address 192.168.1.40/24
   ip virtual-router address 192.168.1.1/24
!
interface Vlan302
   vrf RED
   ip address 192.168.2.40/24
   ip virtual-router address 192.168.2.1/24
!
interface Vlan501
   vrf GREEN
   ip address 192.168.22.40/24
!
interface Vlan502
   vrf RED
   ip address 192.168.23.40/24
!
interface Vxlan1
   description =VXLAN=
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 100010
   vxlan vlan 30 vni 100030
   vxlan vlan 40 vni 100040
   vxlan vlan 201 vni 100201
   vxlan vlan 202 vni 100202
   vxlan vlan 301 vni 1000301
   vxlan vlan 302 vni 1000302
   vxlan vrf CUSTOMER_L3VNI vni 1000777
   vxlan vrf GREEN vni 1000888
   vxlan vrf RED vni 1000999
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:10
!
ip routing
ip routing vrf CUSTOMER_L3VNI
ip routing vrf GREEN
ip routing vrf RED
!
ip prefix-list PL_1
   seq 10 permit 10.11.1.0/24 le 32
   seq 20 permit 10.11.3.0/24 le 32
!
route-map MAP_1 permit 10
   match ip address prefix-list PL_1
!
router bgp 4259905003
   router-id 10.11.1.6
   graceful-restart restart-time 300
   maximum-paths 4 ecmp 64
   neighbor EVPN peer group
   neighbor EVPN remote-as 4259840001
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor EVPN2 peer group
   neighbor EVPN2 remote-as 4259840002
   neighbor EVPN2 update-source Loopback1
   neighbor EVPN2 ebgp-multihop 3
   neighbor EVPN2 send-community extended
   neighbor 10.11.1.1 peer group EVPN
   neighbor 10.11.1.1 description SPINE01_Lo1
   neighbor 10.11.1.2 peer group EVPN2
   neighbor 10.11.1.2 description SPINE02_Lo1
   neighbor 10.11.3.6 remote-as 4259840001
   neighbor 10.11.3.6 bfd
   neighbor 10.11.3.6 description SPINE01
   neighbor 10.11.3.6 route-map MAP_1 in
   neighbor 10.11.3.6 password 7 JVIkOCtPnDM=
   neighbor 10.11.3.46 remote-as 4259840002
   neighbor 10.11.3.46 bfd
   neighbor 10.11.3.46 description SPINE02
   neighbor 10.11.3.46 route-map MAP_1 in
   neighbor 10.11.3.46 password 7 2L3nuAWmD7Y=
   !
   vlan 201
      rd 0.0.0.1:1
      route-target both 100:100
      redistribute learned
   !
   vlan 202
      rd 0.0.0.2:2
      route-target both 100:100
      redistribute learned
   !
   vlan 30
      rd 65001:100030
      route-target both 65001:30
      redistribute learned
      redistribute static
   !
   vlan 301
      rd 65001:1000301
      route-target both 301:301
   !
   vlan 302
      rd 65001:1000302
      route-target both 302:302
   !
   vlan 40
      rd 65001:100040
      route-target both 65001:40
      redistribute learned
      redistribute static
   !
   address-family evpn
      neighbor EVPN activate
      neighbor EVPN2 activate
   !
   address-family ipv4
      neighbor 10.11.3.6 activate
      neighbor 10.11.3.46 activate
      neighbor 192.168.1.200 activate
      neighbor 192.168.2.200 activate
      network 10.11.1.6/32
      network 192.168.1.0/24
      network 192.168.2.0/24
      graceful-restart
   !
   vrf CUSTOMER_L3VNI
      rd 10.11.1.6:1
      route-target import evpn 65000:1
      route-target export evpn 65000:1
      redistribute connected
   !
   vrf GREEN
      rd 1000888:888
      route-target import evpn 301:1000888
      route-target export evpn 301:1000888
      neighbor 192.168.1.200 remote-as 65500
      neighbor 192.168.1.200 description GREEN_VRF-GW
      !
      address-family ipv4
         neighbor 192.168.1.200 activate
   !
   vrf RED
      rd 1000999:999
      route-target import evpn 302:1000999
      route-target export evpn 302:1000999
      neighbor 192.168.2.200 remote-as 65500
      neighbor 192.168.2.200 description RED_VRF-GW
      !
      address-family ipv4
         neighbor 192.168.2.200 activate
!
end
```
</details> 

<details>
  <summary> CUSTOMER </summary>

```
CUSTOMER#show running-config
! Command: show running-config
! device: CUSTOMER (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname CUSTOMER
!
spanning-tree mode mstp
!
vlan 201-202
   name TEST_ESI_LAG
   trunk group ESI_LAG
!
vlan 301
   name GREEN
   trunk group ESI_LAG
!
vlan 302
   name RED
   trunk group ESI_LAG
!
vlan 401-402
!
vrf instance GREEN
!
vrf instance RED
!
interface Port-Channel1
   switchport mode trunk
   switchport trunk group ESI_LAG
   traffic-engineering
!
interface Ethernet1
   description =LAG=
   mtu 9214
   channel-group 1 mode active
!
interface Ethernet2
   description =LAG=
   mtu 9214
   channel-group 1 mode active
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 8.8.8.8/32
!
interface Management1
!
interface Vlan201
   ip address 1.1.1.1/24
!
interface Vlan202
   ip address 2.2.2.2/24
!
interface Vlan301
   vrf GREEN
   ip address 192.168.1.200/24
!
interface Vlan302
   vrf RED
   ip address 192.168.2.200/24
!
interface Vlan401
   vrf GREEN
   ip address 192.168.3.200/24
!
interface Vlan402
   vrf RED
   ip address 192.168.4.200/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vrf GREEN vni 20001
   vxlan vrf RED vni 10001
!
ip routing
ip routing vrf GREEN
ip routing vrf RED
!
ip route 0.0.0.0/0 1.1.1.254
!
route-map RM permit 10
   match as 65500
!
route-map RM permit 20
   match as 4259905002
!
route-map RM permit 30
   match as 4259905003
!
router bgp 65500
   router-id 8.8.8.8
   !
   vrf GREEN
      rd 2:2
      route-target import evpn 10:1
      route-target export evpn 20:1
      neighbor 192.168.1.39 remote-as 4259905002
      neighbor 192.168.1.40 remote-as 4259905003
      redistribute dynamic
      redistribute bgp leaked
      !
      address-family ipv4
         neighbor 192.168.1.40 activate
         neighbor 192.168.3.39 activate
         network 8.8.8.8/32
         network 192.168.1.0/24
         redistribute connected
   !
   vrf RED
      rd 8.8.8.8:3
      route-target import evpn 20:1
      route-target export evpn 10:1
      neighbor 192.168.2.39 remote-as 4259905002
      neighbor 192.168.2.40 remote-as 4259905003
      redistribute dynamic
      redistribute bgp leaked
      !
      address-family ipv4
         neighbor 192.168.2.39 activate
         neighbor 192.168.2.40 activate
         network 8.8.8.8/32
         network 192.168.2.0/24
         redistribute connected
!
end
CUSTOMER#
```
</details> 

## Конфигурации устройств в DataCenter2 (DC02):



### Две независимые VXLAN фабрики с iBGP в Underlay, SPINE-коммутаторы выполняют роль Route Reflector.  IP адреса линковых интерфейсов в Underlay на обеих ФАбриках настроены одинакого для упрощения конфигурации.  Все Loopback-адреса уникальные.
### Для обеспечения доступности loopback -интерфейсов VTEP на SPINE в underlay включена настройка next-hop-self.

### На каждой из фабрик настроено два VRF:   VRF GREEN (сети 10.99.10.0/24 и 10.99.20.1/24)   и VRF RED ( сети 10.88.10.0/24 и 10.88.20.0/24).
### Для обеспечения унификации в каждом ЦОД в VRF RED помещены в VLAN 101 и 102, а в VRF GREEN  VLAN 201 и 202

### Для возможности обеспечения миграции виртуальных машин с одного LEAF на другой (в т.ч. между ЦОД) используеются технология anycast gateway и используется одинаковый MAC -адрес шлюза на всех LEAF обоих ЦОДов.

### Блок DCI двух ЦОД организован на паре Border Leaf коммутаторов, подключенных друг к другу по схеме Back-to-Back ESI LAG (EVPN MH) для организации связи на уровне L2 (VLAN Hand-off) и с использованием routed интерфейсов по схеме каждый с каждым
###  для организации L3 связности по схеме VRF Hand-off.  

### Все L2VNI из одной фабрики отображаются на VLAN -интерфейсы внутри единого EVPN-MH транка и вновь мапятся в L2VNI в новой фабрике.  Номера VLAN и номера L2VNI на двух фабриках выбраны одинаковыми для удобства восприятия.

### В рамках VRF Hand-off организованые routed подключения между Border Leaf обеих Фабрик по схеме "каждый с каждым" и поднято BGP -соседство в AF ipv4

### Для дополнительной демонстрации работоспособности L2VNI создан дополнительный VLAN 333 не принадлежащий VRF RED или VRF Green  (принадлежит Global таблице маршрутизации для которой не организован L3VNI).

### Для дополнительной демонстрации работоспособности L3VNI на коммутаторе DC01_LEAF01  поднят в VRF RED Loopback интерфейс  адресом 1.1.1.0/32 и на коммутаторов DC02_LEAF01 поднят в VRF RED Loopback интерфейс  адресом 2.2.2.0/32
.
### Выход во внешние сети организован через маршрутизатор Internet, с которым поднятые BGP сессии в AF ipv4 со всех четырех Border Leaf обеих фабрик.  На маршрутизаторе Internet не выполнено разделение на VRF, что позволяет обеспечить Leaking маршрутов между VRF RED в VRF GREEN и обратно.  Также обеспечена редистрибьюция сети 8.8.8.8/32 в сторону обоих VRF обеих ФАбрик.

## Проверки:

## Проверка работоспособности L2 (работает ли VLAN- Hand-off)

<details>
  <summary> VPCS> ping 10.88.10.60 -t </summary>

```
VPCS> ping 10.88.10.60 -t

84 bytes from 10.88.10.60 icmp_seq=1 ttl=64 time=84.080 ms
84 bytes from 10.88.10.60 icmp_seq=2 ttl=64 time=105.162 ms
84 bytes from 10.88.10.60 icmp_seq=3 ttl=64 time=87.270 ms
84 bytes from 10.88.10.60 icmp_seq=4 ttl=64 time=156.643 ms
84 bytes from 10.88.10.60 icmp_seq=5 ttl=64 time=88.106 ms
84 bytes from 10.88.10.60 icmp_seq=6 ttl=64 time=91.032 ms
```
</details> 

### Дополнительно проверим связность в VLAN 333


## Проверка работоспособности L3 (работает ли VRF - Hand-off) и редистрибьюции между vrf GREEN и RED.

<details>
  <summary> dc01-leaf01#show ip bgp vrf RED </summary>

```

dc01-leaf01#show ip bgp vrf RED
BGP routing table information for VRF RED
Router identifier 10.88.20.2, local AS number 65501
Route status codes: s - suppressed contributor, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI Origin Validation codes: V - valid, I - invalid, U - unknown
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  AIGP       LocPref Weight  Path
 * >      1.1.1.0/32             -                     -       -          -       0       i
 * >Ec    2.2.2.0/32             10.11.1.5             0       -          100     0       65502 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    2.2.2.0/32             10.11.1.5             0       -          100     0       65502 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    2.2.2.0/32             10.11.1.6             0       -          100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    2.2.2.0/32             10.11.1.6             0       -          100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    8.8.8.8/32             10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    8.8.8.8/32             10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    8.8.8.8/32             10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    8.8.8.8/32             10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 * >Ec    10.11.5.0/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.5.0/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.11.5.0/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.5.0/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 * >Ec    10.11.5.2/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.5.2/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.11.5.2/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.5.2/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 * >Ec    10.11.5.4/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.5.4/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.11.5.4/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.5.4/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 * >Ec    10.11.5.6/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.5.6/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.11.5.6/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.5.6/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 * >Ec    10.11.6.0/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.6.0/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.11.6.0/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.6.0/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 * >Ec    10.11.6.2/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.6.2/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.11.6.2/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.6.2/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 * >Ec    10.11.6.4/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.6.4/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.11.6.4/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.6.4/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 * >Ec    10.11.6.6/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.6.6/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.11.6.6/31           10.11.1.6             0       -          100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.11.6.6/31           10.11.1.5             0       -          100     0       128001 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 * >Ec    10.88.0.0/16           10.11.1.6             0       -          100     0       128001 ? Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.88.0.0/16           10.11.1.5             0       -          100     0       128001 ? Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.88.0.0/16           10.11.1.5             0       -          100     0       128001 ? Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.88.0.0/16           10.11.1.6             0       -          100     0       128001 ? Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >      10.88.10.0/24          -                     -       -          -       0       i
 *  Ec    10.88.10.0/24          10.11.1.5             0       -          100     0       65502 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.88.10.0/24          10.11.1.5             0       -          100     0       65502 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.88.10.0/24          10.11.1.6             0       -          100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.88.10.0/24          10.11.1.6             0       -          100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *        10.88.10.0/24          10.11.1.4             0       -          100     0       i Or-ID: 10.11.1.4 C-LST: 10.11.1.1
 *        10.88.10.0/24          10.11.1.4             0       -          100     0       i Or-ID: 10.11.1.4 C-LST: 10.11.1.1
 * >      10.88.20.0/24          -                     -       -          -       0       i
 *  Ec    10.88.20.0/24          10.11.1.5             0       -          100     0       65502 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.88.20.0/24          10.11.1.5             0       -          100     0       65502 i Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.88.20.0/24          10.11.1.6             0       -          100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.88.20.0/24          10.11.1.6             0       -          100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *        10.88.20.0/24          10.11.1.4             0       -          100     0       i Or-ID: 10.11.1.4 C-LST: 10.11.1.1
 *        10.88.20.0/24          10.11.1.4             0       -          100     0       i Or-ID: 10.11.1.4 C-LST: 10.11.1.1
 * >Ec    10.99.0.0/16           10.11.1.6             0       -          100     0       128001 ? Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    10.99.0.0/16           10.11.1.5             0       -          100     0       128001 ? Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.99.0.0/16           10.11.1.5             0       -          100     0       128001 ? Or-ID: 10.11.1.5 C-LST: 10.11.1.1
 *  ec    10.99.0.0/16           10.11.1.6             0       -          100     0       128001 ? Or-ID: 10.11.1.6 C-LST: 10.11.1.1
```
</details> 

<details>
  <summary> dc01-leaf01#show ip route vrf RED </summary>

```

dc01-leaf01#show ip route vrf RED

VRF: RED
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 C        1.1.1.0/32 is directly connected, Loopback3
 B I      2.2.2.0/32 [200/0] via VTEP 10.11.1.5 VNI 1000100 router-mac 50:00:00:c6:c8:d3 local-interface Vxlan1
                             via VTEP 10.11.1.6 VNI 1000100 router-mac 50:00:00:c6:63:96 local-interface Vxlan1
 B I      8.8.8.8/32 [200/0] via VTEP 10.11.1.5 VNI 1000100 router-mac 50:00:00:c6:c8:d3 local-interface Vxlan1
                             via VTEP 10.11.1.6 VNI 1000100 router-mac 50:00:00:c6:63:96 local-interface Vxlan1
 B I      10.11.5.0/31 [200/0] via VTEP 10.11.1.5 VNI 1000100 router-mac 50:00:00:c6:c8:d3 local-interface Vxlan1
                               via VTEP 10.11.1.6 VNI 1000100 router-mac 50:00:00:c6:63:96 local-interface Vxlan1
 B I      10.11.5.2/31 [200/0] via VTEP 10.11.1.5 VNI 1000100 router-mac 50:00:00:c6:c8:d3 local-interface Vxlan1
                               via VTEP 10.11.1.6 VNI 1000100 router-mac 50:00:00:c6:63:96 local-interface Vxlan1
 B I      10.11.5.4/31 [200/0] via VTEP 10.11.1.5 VNI 1000100 router-mac 50:00:00:c6:c8:d3 local-interface Vxlan1
                               via VTEP 10.11.1.6 VNI 1000100 router-mac 50:00:00:c6:63:96 local-interface Vxlan1
 B I      10.11.5.6/31 [200/0] via VTEP 10.11.1.5 VNI 1000100 router-mac 50:00:00:c6:c8:d3 local-interface Vxlan1
                               via VTEP 10.11.1.6 VNI 1000100 router-mac 50:00:00:c6:63:96 local-interface Vxlan1
 B I      10.11.6.0/31 [200/0] via VTEP 10.11.1.5 VNI 1000100 router-mac 50:00:00:c6:c8:d3 local-interface Vxlan1
                               via VTEP 10.11.1.6 VNI 1000100 router-mac 50:00:00:c6:63:96 local-interface Vxlan1
 B I      10.11.6.2/31 [200/0] via VTEP 10.11.1.5 VNI 1000100 router-mac 50:00:00:c6:c8:d3 local-interface Vxlan1
                               via VTEP 10.11.1.6 VNI 1000100 router-mac 50:00:00:c6:63:96 local-interface Vxlan1
 B I      10.11.6.4/31 [200/0] via VTEP 10.11.1.5 VNI 1000100 router-mac 50:00:00:c6:c8:d3 local-interface Vxlan1
                               via VTEP 10.11.1.6 VNI 1000100 router-mac 50:00:00:c6:63:96 local-interface Vxlan1
 B I      10.11.6.6/31 [200/0] via VTEP 10.11.1.5 VNI 1000100 router-mac 50:00:00:c6:c8:d3 local-interface Vxlan1
                               via VTEP 10.11.1.6 VNI 1000100 router-mac 50:00:00:c6:63:96 local-interface Vxlan1
 C        10.88.10.0/24 is directly connected, Vlan101
 C        10.88.20.0/24 is directly connected, Vlan102
 B I      10.88.0.0/16 [200/0] via VTEP 10.11.1.5 VNI 1000100 router-mac 50:00:00:c6:c8:d3 local-interface Vxlan1
                               via VTEP 10.11.1.6 VNI 1000100 router-mac 50:00:00:c6:63:96 local-interface Vxlan1
 B I      10.99.0.0/16 [200/0] via VTEP 10.11.1.5 VNI 1000100 router-mac 50:00:00:c6:c8:d3 local-interface Vxlan1
                               via VTEP 10.11.1.6 VNI 1000100 router-mac 50:00:00:c6:63:96 local-interface Vxlan1
```
</details> 


<details>
  <summary> dc01-leaf01#ping vrf RED 2.2.2.0 source 1.1.1.0 </summary>

```
dc01-leaf01#ping vrf RED 2.2.2.0 source 1.1.1.0
PING 2.2.2.0 (2.2.2.0) from 1.1.1.0 : 72(100) bytes of data.
80 bytes from 2.2.2.0: icmp_seq=1 ttl=62 time=149 ms
80 bytes from 2.2.2.0: icmp_seq=2 ttl=62 time=152 ms
80 bytes from 2.2.2.0: icmp_seq=3 ttl=62 time=153 ms
80 bytes from 2.2.2.0: icmp_seq=4 ttl=62 time=190 ms
80 bytes from 2.2.2.0: icmp_seq=5 ttl=62 time=194 ms

--- 2.2.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 65ms
rtt min/avg/max/mdev = 149.424/168.184/194.355/19.837 ms, pipe 5, ipg/ewma 16.412/160.235 ms
```
</details> 

<details>
  <summary> dc01-leaf01#show bgp evpn route-type ip-prefix ipv4 </summary>

```
show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.11.1.5, local AS number 65501
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.11.1.3:100 ip-prefix 1.1.1.0/32
                                 10.11.1.3             -       100     0       i Or-ID: 10.11.1.3 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.3:100 ip-prefix 1.1.1.0/32
                                 10.11.1.3             -       100     0       i Or-ID: 10.11.1.3 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.5:100 ip-prefix 2.2.2.0/32
                                 -                     -       100     0       65502 i
 *  ec    RD: 10.11.1.5:100 ip-prefix 2.2.2.0/32
                                 -                     -       100     0       65502 i
 *        RD: 10.11.1.5:100 ip-prefix 2.2.2.0/32
                                 -                     -       100     0       128001 65502 i
 * >      RD: 10.11.1.5:200 ip-prefix 2.2.2.0/32
                                 -                     -       100     0       128001 65502 i
 * >Ec    RD: 10.11.1.6:100 ip-prefix 2.2.2.0/32
                                 10.11.1.6             -       100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:100 ip-prefix 2.2.2.0/32
                                 10.11.1.6             -       100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.6:200 ip-prefix 2.2.2.0/32
                                 10.11.1.6             -       100     0       128001 65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:200 ip-prefix 2.2.2.0/32
                                 10.11.1.6             -       100     0       128001 65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >      RD: 10.11.1.5:100 ip-prefix 8.8.8.8/32
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:100 ip-prefix 8.8.8.8/32
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:100 ip-prefix 8.8.8.8/32
                                 -                     -       100     0       65502 128001 i
 * >      RD: 10.11.1.5:200 ip-prefix 8.8.8.8/32
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:200 ip-prefix 8.8.8.8/32
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:200 ip-prefix 8.8.8.8/32
                                 -                     -       100     0       65502 128001 i
 * >Ec    RD: 10.11.1.6:100 ip-prefix 8.8.8.8/32
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:100 ip-prefix 8.8.8.8/32
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.6:200 ip-prefix 8.8.8.8/32
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:200 ip-prefix 8.8.8.8/32
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >      RD: 10.11.1.5:100 ip-prefix 10.11.5.0/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:100 ip-prefix 10.11.5.0/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:100 ip-prefix 10.11.5.0/31
                                 -                     -       100     0       65502 128001 i
 * >      RD: 10.11.1.5:200 ip-prefix 10.11.5.0/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:200 ip-prefix 10.11.5.0/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:200 ip-prefix 10.11.5.0/31
                                 -                     -       100     0       65502 128001 i
 * >Ec    RD: 10.11.1.6:100 ip-prefix 10.11.5.0/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:100 ip-prefix 10.11.5.0/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.6:200 ip-prefix 10.11.5.0/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:200 ip-prefix 10.11.5.0/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >      RD: 10.11.1.5:100 ip-prefix 10.11.5.2/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:100 ip-prefix 10.11.5.2/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:100 ip-prefix 10.11.5.2/31
                                 -                     -       100     0       65502 128001 i
 * >      RD: 10.11.1.5:200 ip-prefix 10.11.5.2/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:200 ip-prefix 10.11.5.2/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:200 ip-prefix 10.11.5.2/31
                                 -                     -       100     0       65502 128001 i
 * >Ec    RD: 10.11.1.6:100 ip-prefix 10.11.5.2/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:100 ip-prefix 10.11.5.2/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.6:200 ip-prefix 10.11.5.2/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:200 ip-prefix 10.11.5.2/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >      RD: 10.11.1.5:100 ip-prefix 10.11.5.4/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:100 ip-prefix 10.11.5.4/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:100 ip-prefix 10.11.5.4/31
                                 -                     -       100     0       65502 128001 i
 * >      RD: 10.11.1.5:200 ip-prefix 10.11.5.4/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:200 ip-prefix 10.11.5.4/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:200 ip-prefix 10.11.5.4/31
                                 -                     -       100     0       65502 128001 i
 * >Ec    RD: 10.11.1.6:100 ip-prefix 10.11.5.4/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:100 ip-prefix 10.11.5.4/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.6:200 ip-prefix 10.11.5.4/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:200 ip-prefix 10.11.5.4/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >      RD: 10.11.1.5:100 ip-prefix 10.11.5.6/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:100 ip-prefix 10.11.5.6/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:100 ip-prefix 10.11.5.6/31
                                 -                     -       100     0       65502 128001 i
 * >      RD: 10.11.1.5:200 ip-prefix 10.11.5.6/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:200 ip-prefix 10.11.5.6/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:200 ip-prefix 10.11.5.6/31
                                 -                     -       100     0       65502 128001 i
 * >Ec    RD: 10.11.1.6:100 ip-prefix 10.11.5.6/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:100 ip-prefix 10.11.5.6/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.6:200 ip-prefix 10.11.5.6/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:200 ip-prefix 10.11.5.6/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >      RD: 10.11.1.5:100 ip-prefix 10.11.6.0/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:100 ip-prefix 10.11.6.0/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:100 ip-prefix 10.11.6.0/31
                                 -                     -       100     0       65502 128001 i
 * >      RD: 10.11.1.5:200 ip-prefix 10.11.6.0/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:200 ip-prefix 10.11.6.0/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:200 ip-prefix 10.11.6.0/31
                                 -                     -       100     0       65502 128001 i
 * >Ec    RD: 10.11.1.6:100 ip-prefix 10.11.6.0/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:100 ip-prefix 10.11.6.0/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.6:200 ip-prefix 10.11.6.0/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:200 ip-prefix 10.11.6.0/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >      RD: 10.11.1.5:100 ip-prefix 10.11.6.2/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:100 ip-prefix 10.11.6.2/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:100 ip-prefix 10.11.6.2/31
                                 -                     -       100     0       65502 128001 i
 * >      RD: 10.11.1.5:200 ip-prefix 10.11.6.2/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:200 ip-prefix 10.11.6.2/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:200 ip-prefix 10.11.6.2/31
                                 -                     -       100     0       65502 128001 i
 * >Ec    RD: 10.11.1.6:100 ip-prefix 10.11.6.2/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:100 ip-prefix 10.11.6.2/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.6:200 ip-prefix 10.11.6.2/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:200 ip-prefix 10.11.6.2/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >      RD: 10.11.1.5:100 ip-prefix 10.11.6.4/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:100 ip-prefix 10.11.6.4/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:100 ip-prefix 10.11.6.4/31
                                 -                     -       100     0       65502 128001 i
 * >      RD: 10.11.1.5:200 ip-prefix 10.11.6.4/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:200 ip-prefix 10.11.6.4/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:200 ip-prefix 10.11.6.4/31
                                 -                     -       100     0       65502 128001 i
 * >Ec    RD: 10.11.1.6:100 ip-prefix 10.11.6.4/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:100 ip-prefix 10.11.6.4/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.6:200 ip-prefix 10.11.6.4/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:200 ip-prefix 10.11.6.4/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >      RD: 10.11.1.5:100 ip-prefix 10.11.6.6/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:100 ip-prefix 10.11.6.6/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:100 ip-prefix 10.11.6.6/31
                                 -                     -       100     0       65502 128001 i
 * >      RD: 10.11.1.5:200 ip-prefix 10.11.6.6/31
                                 -                     -       100     0       128001 i
 *  Ec    RD: 10.11.1.5:200 ip-prefix 10.11.6.6/31
                                 -                     -       100     0       65502 128001 i
 *  ec    RD: 10.11.1.5:200 ip-prefix 10.11.6.6/31
                                 -                     -       100     0       65502 128001 i
 * >Ec    RD: 10.11.1.6:100 ip-prefix 10.11.6.6/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:100 ip-prefix 10.11.6.6/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.6:200 ip-prefix 10.11.6.6/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:200 ip-prefix 10.11.6.6/31
                                 10.11.1.6             -       100     0       128001 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >      RD: 10.11.1.5:100 ip-prefix 10.88.0.0/16
                                 -                     -       100     0       128001 ?
 *  Ec    RD: 10.11.1.5:100 ip-prefix 10.88.0.0/16
                                 -                     -       100     0       65502 128001 ?
 *  ec    RD: 10.11.1.5:100 ip-prefix 10.88.0.0/16
                                 -                     -       100     0       65502 128001 ?
 * >      RD: 10.11.1.5:200 ip-prefix 10.88.0.0/16
                                 -                     -       100     0       128001 ?
 *  Ec    RD: 10.11.1.5:200 ip-prefix 10.88.0.0/16
                                 -                     -       100     0       65502 128001 ?
 *  ec    RD: 10.11.1.5:200 ip-prefix 10.88.0.0/16
                                 -                     -       100     0       65502 128001 ?
 * >Ec    RD: 10.11.1.6:100 ip-prefix 10.88.0.0/16
                                 10.11.1.6             -       100     0       128001 ? Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:100 ip-prefix 10.88.0.0/16
                                 10.11.1.6             -       100     0       128001 ? Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.6:200 ip-prefix 10.88.0.0/16
                                 10.11.1.6             -       100     0       128001 ? Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:200 ip-prefix 10.88.0.0/16
                                 10.11.1.6             -       100     0       128001 ? Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.3:100 ip-prefix 10.88.10.0/24
                                 10.11.1.3             -       100     0       i Or-ID: 10.11.1.3 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.3:100 ip-prefix 10.88.10.0/24
                                 10.11.1.3             -       100     0       i Or-ID: 10.11.1.3 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.4:100 ip-prefix 10.88.10.0/24
                                 10.11.1.4             -       100     0       i Or-ID: 10.11.1.4 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.4:100 ip-prefix 10.88.10.0/24
                                 10.11.1.4             -       100     0       i Or-ID: 10.11.1.4 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.5:100 ip-prefix 10.88.10.0/24
                                 -                     -       100     0       65502 i
 *  ec    RD: 10.11.1.5:100 ip-prefix 10.88.10.0/24
                                 -                     -       100     0       65502 i
 * >Ec    RD: 10.11.1.6:100 ip-prefix 10.88.10.0/24
                                 10.11.1.6             -       100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:100 ip-prefix 10.88.10.0/24
                                 10.11.1.6             -       100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.3:100 ip-prefix 10.88.20.0/24
                                 10.11.1.3             -       100     0       i Or-ID: 10.11.1.3 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.3:100 ip-prefix 10.88.20.0/24
                                 10.11.1.3             -       100     0       i Or-ID: 10.11.1.3 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.4:100 ip-prefix 10.88.20.0/24
                                 10.11.1.4             -       100     0       i Or-ID: 10.11.1.4 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.4:100 ip-prefix 10.88.20.0/24
                                 10.11.1.4             -       100     0       i Or-ID: 10.11.1.4 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.5:100 ip-prefix 10.88.20.0/24
                                 -                     -       100     0       65502 i
 *  ec    RD: 10.11.1.5:100 ip-prefix 10.88.20.0/24
                                 -                     -       100     0       65502 i
 * >Ec    RD: 10.11.1.6:100 ip-prefix 10.88.20.0/24
                                 10.11.1.6             -       100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:100 ip-prefix 10.88.20.0/24
                                 10.11.1.6             -       100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >      RD: 10.11.1.5:100 ip-prefix 10.99.0.0/16
                                 -                     -       100     0       128001 ?
 *  Ec    RD: 10.11.1.5:100 ip-prefix 10.99.0.0/16
                                 -                     -       100     0       65502 128001 ?
 *  ec    RD: 10.11.1.5:100 ip-prefix 10.99.0.0/16
                                 -                     -       100     0       65502 128001 ?
 * >      RD: 10.11.1.5:200 ip-prefix 10.99.0.0/16
                                 -                     -       100     0       128001 ?
 *  Ec    RD: 10.11.1.5:200 ip-prefix 10.99.0.0/16
                                 -                     -       100     0       65502 128001 ?
 *  ec    RD: 10.11.1.5:200 ip-prefix 10.99.0.0/16
                                 -                     -       100     0       65502 128001 ?
 * >Ec    RD: 10.11.1.6:100 ip-prefix 10.99.0.0/16
                                 10.11.1.6             -       100     0       128001 ? Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:100 ip-prefix 10.99.0.0/16
                                 10.11.1.6             -       100     0       128001 ? Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.6:200 ip-prefix 10.99.0.0/16
                                 10.11.1.6             -       100     0       128001 ? Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:200 ip-prefix 10.99.0.0/16
                                 10.11.1.6             -       100     0       128001 ? Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.3:200 ip-prefix 10.99.10.0/24
                                 10.11.1.3             -       100     0       i Or-ID: 10.11.1.3 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.3:200 ip-prefix 10.99.10.0/24
                                 10.11.1.3             -       100     0       i Or-ID: 10.11.1.3 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.4:200 ip-prefix 10.99.10.0/24
                                 10.11.1.4             -       100     0       i Or-ID: 10.11.1.4 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.4:200 ip-prefix 10.99.10.0/24
                                 10.11.1.4             -       100     0       i Or-ID: 10.11.1.4 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.5:200 ip-prefix 10.99.10.0/24
                                 -                     -       100     0       65502 i
 *  ec    RD: 10.11.1.5:200 ip-prefix 10.99.10.0/24
                                 -                     -       100     0       65502 i
 * >Ec    RD: 10.11.1.6:200 ip-prefix 10.99.10.0/24
                                 10.11.1.6             -       100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:200 ip-prefix 10.99.10.0/24
                                 10.11.1.6             -       100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.3:200 ip-prefix 10.99.20.0/24
                                 10.11.1.3             -       100     0       i Or-ID: 10.11.1.3 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.3:200 ip-prefix 10.99.20.0/24
                                 10.11.1.3             -       100     0       i Or-ID: 10.11.1.3 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.4:200 ip-prefix 10.99.20.0/24
                                 10.11.1.4             -       100     0       i Or-ID: 10.11.1.4 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.4:200 ip-prefix 10.99.20.0/24
                                 10.11.1.4             -       100     0       i Or-ID: 10.11.1.4 C-LST: 10.11.1.1
 * >Ec    RD: 10.11.1.5:200 ip-prefix 10.99.20.0/24
                                 -                     -       100     0       65502 i
 *  ec    RD: 10.11.1.5:200 ip-prefix 10.99.20.0/24
                                 -                     -       100     0       65502 i
 * >Ec    RD: 10.11.1.6:200 ip-prefix 10.99.20.0/24
                                 10.11.1.6             -       100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
 *  ec    RD: 10.11.1.6:200 ip-prefix 10.99.20.0/24
                                 10.11.1.6             -       100     0       65502 i Or-ID: 10.11.1.6 C-LST: 10.11.1.1
```
</details> 

### Проверим L3 связность между фабриками и route leaking между VRF Green и RED пропинговав интерфейс DC02_LEAF01 на Фабрике№2 в vrf RED с интерфейса DC01_LEAF01 Фабрики №1 в VRF GREEN:

<details>
  <summary> dc01-bleaf01#ping vrf GREEN 10.88.20.20 source 10.99.10.4 </summary>

```
dc01-bleaf01#ping vrf GREEN 10.88.20.20 source 10.99.10.4
PING 10.88.20.20 (10.88.20.20) from 10.99.10.4 : 72(100) bytes of data.
80 bytes from 10.88.20.20: icmp_seq=1 ttl=62 time=842 ms
80 bytes from 10.88.20.20: icmp_seq=2 ttl=62 time=837 ms
80 bytes from 10.88.20.20: icmp_seq=3 ttl=62 time=836 ms
80 bytes from 10.88.20.20: icmp_seq=4 ttl=62 time=906 ms
80 bytes from 10.88.20.20: icmp_seq=5 ttl=62 time=912 ms

--- 10.88.20.20 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 47ms
rtt min/avg/max/mdev = 836.440/867.047/912.667/34.717 ms, pipe 5, ipg/ewma 11.960/857.408 ms
```
</details> 

### И сделаем traceroute:

<details>
  <summary> dc01-bleaf01#traceroute vrf GREEN 10.88.20.20 source 10.99.10.4 </summary>

```
dc01-bleaf01#traceroute vrf GREEN 10.88.20.20 source 10.99.10.4
traceroute to 10.88.20.20 (10.88.20.20), 30 hops max, 60 byte packets
 1  10.11.6.1 (10.11.6.1)  38.852 ms  45.358 ms  54.062 ms
 2  10.11.5.0 (10.11.5.0)  360.667 ms  380.478 ms  392.617 ms
 3  10.88.20.20 (10.88.20.20)  342.415 ms  352.691 ms  397.516 ms
```
</details> 

# ВЫВОДЫ:
