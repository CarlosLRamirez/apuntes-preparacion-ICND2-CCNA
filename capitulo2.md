# Capitulo 2: Spanning Tree Protocol Concepts

## Spanning Tree Protocol (IEEE 802.1D)

STP previene los loops en los switches, colocando algunos puertos en modo Forwarding y otros en modo Bloqueado.

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

Problemas causados por los enlaces redudantes en una LAN (sin STP)

|Problema|Descripción|
|--------|-----------|
|Tormentas de broadcast|El reenvio de una trama repetidamente en los mismos enlaces, consumiendo una parte significativa de la capacidad del enlace|
|Inestabilidad de tabla MAC|La actualización continua de los switches de su tabla MAC con entradas incorrectas, lo cual resulta en tramas enviadas a ubicaciones incorrectas|
|Transmisión multiple de tramas|Multiples copias de las tramas son enviadas a los hots|

## Algoritmo de Spanning Tree

* STP elije un switch root. Todas los puertos del switch root se colocan en estado FORWARDING.
* Cada No-root switch elige un root port (RP), el cual tiene el *root cost* menor, es decir el costo para poder alcanzar al root bridge. El switch pone el root port (RP) en estado **FORWARDING**.
* Para cada segmento Ethernet (en las redes modernas, solo hay dos switches en cada segmento)  el switch con el root cost menor se pone en modo **FORWARDING**, este switch es el *switch designado*, y el puerto es el *puerto designado* (DP). El resto de puertos se pone en estado **BLOCKING**