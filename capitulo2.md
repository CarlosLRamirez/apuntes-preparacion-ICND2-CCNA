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

### Que es lo que hace Spanning Tree

STP previene loops, colocando cada puerto del switch, ya sea en estado **forwarding** o **blocking**. Las interfaces en estado forwarding actuan de forma normal. Las interfaces en estado blocking no procesan tramas excepto mensajes STP (BPDU).

Si algún enlace falla o hay algun cmabio en la topologia, STP *converge* para elegir nuevamente que puertos va colocar el estado **forwarding** o en estado **blocking**

#### Algoritmo de Spanning Tree (STA)

* STP elije un switch root. Todas los puertos del switch root se colocan en estado FORWARDING.
* Cada NO-ROOT switch determina que puerto tiene el menor costo administrativo entre si y el root switch (*root cost*). Este puerto es el *root port* (RP). El switch coloca el root port (RP) en estado **FORWARDING**.
* Cada segmento Ethernet (en las redes modernas-sin hubs, solo hay dos switches en cada segmento) el switch con el menor *root cost* es el *switch designado* y el puerto es el  *puerto designado*, (DP) este se pone en modo **FORWARDING**. El resto de puertos del segmento que no son ni designados y ni root port, son puertos  alternativos, y se coloca en estado *BLOCKING*.

> Por cada segmento entre el root switch y un no-root switch, el puerto designado (DP) es el puerto en el root switch, y por eso este se coloca en modo forwarding.

Razones para colocar un puerto de FWD o BLK

|Caracteristicas|Estado STP|Descripcion|
|---------------|----------|-----------|
|Todos los puertos del root switch|Forwarding|Siempre es el switch designado en todos los segmentos conectados|
|Todos los RP de los switches no-root|Forwarding|El puerto que tiene el menor costo para llegar al root switch|
|Cada puerto designado de c/segmento|Forwarding|Es el switch que reenvia los Hello en el segmento, el cual tiene el menor root cost|
|El resto de puertos operativos|Blocking|No se usan para reenviar tramas de usuario|

#### STP Bridge ID y los Hello BPDU

El STP *bridge ID* (BID) es un valor unico de 8 bits para cada switch. El BID consiste en **2 bytes** con la prioridad y **6 bytes** con el system ID. El system ID es la MAC Address de cada switch.

![](priority%20pvstp.png)

Los mensajes que utiliza STP se llaman BPDU. El BPDU mas comun en el Hello BPDU.

Campos de STP Hello BPDU

|Campo|Descripcion|
|-----|-----------|
|Root Bridge ID|El bridge ID del switch que el que envia el Hello actualmente cree que es el root bridge|
|Bridge ID del remitente|El Bridge ID switch que envia el Hello BPDU|
|*root cost* del remitente|El costo STP entre este switch y el root actual|
|Valores de los temporizadores del root switch|Incluye el *Hello timer*,*MaxAge timer* y *forward delay timer*.

##### Elección del Root Switch

El switch con el Bridge ID menor, es el root switch. Si hay empate en la prioridad, el desempate es la MAC Address.

Cada switch empieza a enviar Hello BPDUs creyendo que el es el root bridge, luego que todos enviaron Hello BPDU se define quien es el root, y solo este se queda originando Hello BPDUs, el resto de switches reenvian estos Hello en otras interfaces.

##### Elección del Root Port en cada Switch No-Root

El root port (RP) es el puerto que tiene menor costo para llegar al root switch. El costo es la *suma de los costos de todos los puertos de los switches por los que la trama va salir.*

![](eleccion%20root%20port.png)

El root port envia el Hello BPDU al SW2 con Root Cost = 0, luego el SW2 modifica el Hello Message, añadiendo su propio Root Cost=4, y lo reenvia a SW3.

SW3 al recibir dicho mensaje, suma su propio Root Cost = 4 al del Hello recibido para determinal que el root cost para salir por G01/2 es de 4+4 = 8.

##### Elección del Puerto Designado en cada segmento LAN

El puerto designado (DP) en cada segmento LAN es el puerto del switch que anuncia el Hello con el costo menor en un segmento LAN.

Si existe un empate en el costo anunciado, entonces el desempate es el switch con el menor BID.

El puerto designado de ese segmento se coloca en modo forwarding, y los demas puertos del segmento son los puertos alternos (AP) y se colocan en modo blocking.

> Aunque hoy en dia es muy raro. Un solo switch puede conectar dos o mas interfaces al mismo dominio de colision, en este caso el switch escucha su propio BPDU, en ese caso habria un empate del switch con el mismo. En ese caso se utilizan otros dos criterios de desempate: la interfaz con la prioridad STP menor, y la interfaz con el numero menor.

### Influenciando y cambiando la topologia STP

Los switch cambian su topologia de STP, ya sea por un cambio en la topologia (caida de un enlace), o un cambio en la configuración.
