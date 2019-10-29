# Capitulo 2: Spanning Tree Protocol Concepts

## Spanning Tree Protocol (IEEE 802.1D)

STP no cambia el status de una interfaz, incluse si esta bloqueada por STP

```plaintext
Switch#show interfaces f0/2 status
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/2                        connected    1          auto    auto  10/100BaseTX

Switch#show interfaces f0/3 status 
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/3                        connected    1          auto    auto  10/100BaseTX
```



Tampoco cambia el Operational Mode del puerto ya sea troncal o acceso.

```plaintext
Switch#show interfaces f0/2 switchport 
Name: Fa0/2
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: static access
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: native
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 1 (default)
Voice VLAN: none
```

STP agrega su propio status, Bloqueado o Forwarding
```plaintext
Switch#show spanning-tree interface f0/2
Vlan             Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
VLAN0001         Root FWD 19        128.2     P2p
Switch#show spanning-tree interface f0/3
Vlan             Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
VLAN0001         Altn BLK 19        128.3     P2p
```

