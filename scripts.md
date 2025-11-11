1. VLAN Setup di S1

```sh
enable
conf t
vlan 10
 name SEKRETARIAT
vlan 20
 name KURIKULUM
vlan 30
 name GURU_TENDIK
vlan 40
 name SARPRAS
vlan 50
 name SERVER_ADMIN
exit

! Port ke router dijadikan TRUNK
interface fa0/1
 switchport mode trunk
exit

! Contoh: PC Sekretariat di Fa0/2, PC Kurikulum di Fa0/3, dst.
interface range fa0/2
 switchport mode access
 switchport access vlan 10
exit
interface range fa0/3
 switchport mode access
 switchport access vlan 20
exit
interface range fa0/4
 switchport mode access
 switchport access vlan 30
exit
interface range fa0/5
 switchport mode access
 switchport access vlan 40
exit
interface range fa0/6
 switchport mode access
 switchport access vlan 50
exit
end
wr
```

2. Subinterface di R1

```sh
enable
conf t

! Link ke S1, jadikan trunk subinterfaces
interface g0/0
 no shutdown
exit

! VLAN 10 - Sekretariat (10.48.0.0/23, GW .1)
interface g0/0.10
 encapsulation dot1Q 10
 ip address 10.48.0.1 255.255.254.0

! VLAN 20 - Kurikulum
interface g0/0.20
 encapsulation dot1Q 20
 ip address 10.48.2.1 255.255.255.0

! VLAN 30 - Guru & Tendik
interface g0/0.30
 encapsulation dot1Q 30
 ip address 10.48.3.1 255.255.255.128

! VLAN 40 - Sarpras
interface g0/0.40
 encapsulation dot1Q 40
 ip address 10.48.3.129 255.255.255.192

! VLAN 50 - Server & Admin
interface g0/0.50
 encapsulation dot1Q 50
 ip address 10.48.4.1 255.255.255.248
exit
```

3. Konfig Link P2P R1 <-> R2

- R1

```sh
conf t
interface g0/1
 ip address 10.48.4.9 255.255.255.252
 no shutdown
exit
```

- R2

```sh
enable
conf t
interface g0/1
 ip address 10.48.4.10 255.255.255.252
 no shutdown
exit
```

4. VLAN di S2

```sh
enable
conf t
vlan 110
 name PENGAWAS_BRANCH
vlan 120
 name PENGAWAS_CABANG
exit

! Port ke router trunk
interface fa0/1
 switchport mode trunk
exit

! Misal PC Branch di Fa0/2, PC Cabang di Fa0/3
interface fa0/2
 switchport mode access
 switchport access vlan 110
exit
interface fa0/3
 switchport mode access
 switchport access vlan 120
exit
end
wr
```

5. Subinterface di R2

```sh
conf t
interface g0/0
 no shutdown
exit

! VLAN 110 - Pengawas (Branch) GW 10.48.3.193/27
interface g0/0.110
 encapsulation dot1Q 110
 ip address 10.48.3.193 255.255.255.224

! VLAN 120 - Pengawas (Cabang) GW 10.48.3.225/27
interface g0/0.120
 encapsulation dot1Q 120
 ip address 10.48.3.225 255.255.255.224
exit
```

6. Routing antar R1 dan R2

- R1

```sh
conf t
ip route 10.48.3.192 255.255.255.224 10.48.4.10
ip route 10.48.3.224 255.255.255.224 10.48.4.10
exit
wr
```

- R2

```sh
conf t
! Rute balik ke semua jaringan pusat lewat R1 (next-hop = 10.48.4.9)
ip route 10.48.0.0 255.255.254.0 10.48.4.9    ! Sekretariat
ip route 10.48.2.0 255.255.255.0 10.48.4.9    ! Kurikulum
ip route 10.48.3.0 255.255.255.128 10.48.4.9  ! Guru&Tendik
ip route 10.48.3.128 255.255.255.192 10.48.4.9 ! Sarpras
ip route 10.48.4.0 255.255.255.248 10.48.4.9  ! Server&Admin
exit
wr
```

