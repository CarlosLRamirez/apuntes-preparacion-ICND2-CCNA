# Capítulo 1 - Implementing Ethernet Virtual LANs

Una VLAN = Subnet =  Dominio de Broadcast

Razones para crear dominos de broadcast (VLAN) mas pequeños:
* Reducir la carga de CPU en cada dispositivo, al reducir el numero de dispositivos que reciben las tramas de difusión.
* Reducir los riesgos de seguridad, al reducir el numero de dispositivos que reciben las tramas que el switch hace flooding (broadcast, multicats y unicast desconocidos).
* Mejorar la seguridad en los host que puedan enviar información sensible, al mantenerlos en VLANs separadas.
* Crear un diseño mas flexible que agrupe a los usuarios por departamento, o grupos de trabajo, en lugar de su ubicación física
* Para resolver problemas más rápidamente, ya que el dominio de falla para muchos problemas esta en los dispositivos del mismo dominio de difusión.
* Para reducir la carga de trabajo del Spanning Tree (STP) 

## Protocolos de VLAN Trunking: ISL e 802.1Q
	
**ISL:** creado por Cisco antes del 802.1Q, ya casi no se usa, los 2960 ya no lo tienen habilitado.

**802.1Q** creado por la IEEE

802.1Q agrega un encabezado en la trama Ethernet de 4 bytes (32 bits) después del *SourceAddress* y antes de *Type*.

![](801q%20trunking.png)

De estos 4 bytes, solo los últimos 12 bits se usan para el VLAN ID

2^12=4096, 0 y la 4095 no se usan, por lo que el numero máximo de VLANs es 4094

Los switches Cisco, separan el rango de VLANs en dos:

| Rango de VLANS  |  VLAN ID        |
|-----------------|-----------------|
| Rango normal    |1 a 100 |
| Rango extendido |1006 a 4094 |
| No se pueden usar como acceso <br>No se puede borrar |1002 a 1005 |
| VLAN por defecto<br>No se puede borrar |1          |

VLANs de rango extendido:
* Solo ciertos switches pueden utilizarlos
* Depende de la configuración de VTP

801.Q por definición no agrega ningun encabezado en las tramas de la VLAN nativa.

VLAN nativa por defecto: VLAN 1

Si un switch del otro lado recibe una trama sin encabezado 802.1Q, este sabe que la trama es de la VLAN nativa.

El concepto de VLAN nativa, provee al switch a capacidad de pasar al menos trafico de una VLAN, (la VLAN nativa), para permitir la conexión por telnet a un switch.

## Routing inter-VLANS

**Switch Capa 2 y Router Capa3:** 
*"router-on-a-stick":* El router enruta paquetes Capa 3 entre subredes capa 3, donde cada subred mapea una VLAN capa 2.

**Switch Capa 3:**
Es mas rápido que enrutar por medio de un Router capa 3, ya que el proceso de routing ocurre en el mismo equipo.

## Configuración y Verificación de VLAN y VLAN Trunking

### Creación de una VLAN y asignación del acceso a una interfaz

```plaintext
Switch(config)#vlan 100
Switch(config-vlan)#name STUDENT
Switch(config-vlan)#exit
Switch(config)#interface f0/1
Switch(config-if)#switchport access vlan 100 --> asignar a una vlan, sin cambiar el switchport mode
Switch(config-if)#exit
Switch(config)#
 
Switch#show int f0/1 switchport
Name: Fa0/1
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: down
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: native
Negotiation of Trunking: On
Access Mode VLAN: 100 (STUDENT) --> se asigno la vlan de acceso, pero sigue como dynamic auto, por defecto
Trunking Native Mode VLAN: 1 (default)
Voice VLAN: none
…
```

Si no se asigna un nombre a una VLAN, se le auto asigna el nombre VLANZZZZ donde ZZZZ es el ID.

Si se le da switchport mode access a un puerto, este configura siempre Administrative Mode como acceso (para que nunca negocie y no se pueda volver trunk) (por defecto viene: dynamic auto).

```plaintext
Switch(config)#interface f0/1
Switch(config-if)#switchport mode access 
Switch(config-if)#end

Switch#show interfaces f0/1 switchport
Name: Fa0/1
Switchport: Enabled
Administrative Mode: static access
Operational Mode: down
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 100 (STUDENT)
Trunking Native Mode VLAN: 1 (default)
Voice VLAN: none
…
```

En el **running-config**,  aparecen los comandos de creación de las VLAN, unicamente si el VTP esta en modo **transparente** (por default viene en modo **server**)

```plaintext
!
vtp mode transparent
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
vlan 100
name STUDENT
!
vlan 200
!
```
Si se asigna aun puerto una VLAN que no existe previamente, el switch la crea con el nombre VLANZZZZ.

```plaintext
Switch(config)#interface range f0/5-7
Switch(config-if-range)#switchport access vlan 300
% Access VLAN does not exist. Creating vlan 300
Switch(config-if-range)#switchport mode access
Switch(config-if-range)#exit
Switch#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/2, Fa0/3, Fa0/4, Fa0/8
                                                Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                Fa0/13, Fa0/14, Fa0/15, Fa0/16
                                                Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                Gig0/1, Gig0/2
100  STUDENT                          active    Fa0/1
200  VLAN0200                         active    
300  VLAN0300                         active    Fa0/5, Fa0/6, Fa0/7
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
Switch#
```

---
### VTP: Vlan Trunking Protocol

VTP: propietario de Cisco, anuncia cada VLAN configurada en un switch, con su ID. Muchas empresas deciden no implementarlo

VTP se puede deshabilitar, poniendolo en modo transparente con el comando global `vtp mode transparent` o apagandolo completamente con `vtp mode off`. Ambas opciones muestran los comandos de **vlan** en el **running-config**

VTP viene configurado por defecto como modo 'Server'

El modo **off** (configurable only in CatOS switches) funciona del mismo modo que el modo **transparent**, con la excepción que no reenvia los anuncios sobre la información de la VLANs.

---
### Configuración de VLAN Trunking

Cuando configuramos un puerto como troncal, podemos definir los siguiente:
* El tipo de trunking: puede ser IEEE 802.1Q, ISL, o que el switch negocie cual utilizar.
* El modo administrativo: Siempre usar trunk, siempre No trunk, o modo negociar.

Los switches CISCO que soportan ISP e 802.1Q pueden negociar cual protocolo utilizar, utilizando **DTP** (Dynamic Trunking Protocol). Si ambos switches soportan ambos protocolos, usan ISL. Si nó, el protocolo que ambos soporten.

Los switches que soportan ambos protocolos se pueden configurar con el comando de interfaz `switchport trunk encapsulation {dot1q | isl | negotiate}`, para establecerlo manual o negociado por DTP.

DTP tambien se utilizar para que lo switches negocien si establecen un enlace troncal o no, dependiendo del **Administrative Mode** que esten configurados. 

El **Operational Mode** indica lo que realmente esta ocurriendo en la interfaz. Para configurar el **Administrative Mode** se usa el subcomando de interfaz `switchport mode`.

| Opcion | Descripción|
|--------|------------|
| **access** | Siempre actua como puerto acceso (No trunk)|
| **trunk**  | Siempre actua como puerto trunk|
| **dynamic desirable** | Inicia los mensajes de negociación y responde a los mensaje de negociación, para dinamicamente establecer el enlace trunk |
| **dynamic auto** | Pasivamente espera a recibir mensajes de negociación de trunk, y responde para negociar si usar establecer el trunk |

Por defecto los puertos en los switches 2960 están en modo **dynamic auto**, por lo que al interconectar dos puertos, no establecen automaticamente el modo trunk, al cambiar uno a **dynamic desirable**, estos empiezan a negociar y se establece el modo trunk, usando 802.1Q, porque los 2960 no soportan ISL.

Configuración por default de un puerto:
```plaintext
Switch#show interfaces g0/1 switchport 
Name: Gig0/1
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: static access
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: native
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 1 (default)
Voice VLAN: none
...
Trunking VLANs Enabled: All
```

`Operational Mode: static access` significa que no es troncal

`Administrative Trunking Encapsulation: dot1q` El 2960 no soporta ISL, en un switch que soporta ambos protocolos, apareceria *negotiate*

`Operational Trunking Encapsulation: native` indica que el puerto no esta en modo troncal, y que las tramas no tiene encabezados de trunk, por eso indica native.

Por default no se establece el enlace troncal
```plaintext
Switch#show interfaces trunk 

Switch#
```

Al configurar una de las interfaces como `dynamic desirable`, entonces los switches negocian y se establece el enlace troncal.

Cuando se envia el comando, la interfaz se cae y se levanta nuevamente, por causa de la transición de modo acceso a troncal


```plaintext
Switch(config)#interface g0/1
Switch(config-if)#switchport mode dynamic desirable
Switch(config-if)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up

Switch#show interfaces g0/1 switchport
Name: Gig0/1
Switchport: Enabled
Administrative Mode: dynamic desirable
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 1 (default)
Voice VLAN: none
...


Switch#show interfaces trunk 
Port        Mode         Encapsulation  Status        Native vlan
Gig0/1      desirable    n-802.1q       trunking      1

Port        Vlans allowed on trunk
Gig0/1      1-1005

Port        Vlans allowed and active in management domain
Gig0/1      1

Port        Vlans in spanning tree forwarding state and not pruned
Gig0/1      1

Switch#
```

> NOTA:
> Según la guia oficial (pag 34), el comando `show vlan id` debería de mostrar los puertos de acceso y tambien los puertos troncales. Al parecer en equipo real si funciona así, pero en Packet Tracer solo aparecen los puertos de acceso asignados a esa vlan. 
 
 ```plaintext
 Switch#show vlan id 200
 
 VLAN Name                             Status    Ports
 ---- -------------------------------- --------- -------------------------------
 200  TEACHERS                         active    Fa0/5, Fa0/6, Fa0/7
 
 VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
 ---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
 200  enet  100200     1500  -      -      -        -    -        0      0
 
 Switch#
 ```
**Operational Mode** esperado según la combinación de **Administrative Mode** en los enlaces troncales.


| Administrative Mode | Access              | Dynamic Auto        | Trunk               | Dynamic Desirable   |
|---------------------|---------------------|---------------------|---------------------|---------------------|
| access              | Access              | Access              | Do Not Use          | Access              |
| dynamic auto        | Access              | Access              | Trunk               | Trunk               |
| trunk               | Do Not Use          | Trunk               | Trunk               | Trunk               |
| dynamic desirable   | Access              | Trunk               | Trunk               | Trunk               |

Por seguridad, Cisco recomienda deshabilitar a negociación de troncales en la mayoria de puertos, ya que estos se conectarán con usuarios. Esto se hace con el subcomando de interfaz `switchport nonegotiate`.

El comando `switchport nonegotiate` solo funciona si el modo administrativo esta en **trunk** o en **access**. 

Por ejemplo si en el SW1 se pone el puerto en modo **trunk**, pero sin negociación; y en el SW2 esta en **dynamic desirable**, no se establecerá el enlace troncal. Solo en el SW1, pero en SW2 no aparece. (comprobado en PT).

---
#### Interfaces conectadas a Telefonos

Los telefonos IPs incluyen un pequeño switch de tres puertos, que permite que un solo cable de red llegue a cada puestro de trabajo, directamete al telefono y luego hay un cable que va del telefono a la PC.

En este caso particular, el puerto se comporta como puerto de acceso (para el trafico de la PC) y un poco como puerto troncal (para el tráfico del telefono). Para esto se define una VLAN de voz.

* VLAN de datos: se configura como una VLAN de acceso normal. Se define como la VLAN donde se enviara el trafico al equipo que esta conectado al telefono (la PC).
* VLAN de voz: por aqui se envia el trafico del telefono. El trafico esta etiquetado con el encabezado 802.1Q

Ejemplo de configuración de la VLAN de datos y de voz, para un switch con telefonos y PCs conectadas en los puertos f0/1-4

```plaintext
Switch(config)#
Switch(config)#vlan 10
Switch(config-vlan)#name DATA
Switch(config-vlan)#vlan 11
Switch(config-vlan)#name VOICE
Switch(config)#interface range f0/1-4
Switch(config-if-range)#switchport mode access 
Switch(config-if-range)#switchport access vlan 10
Switch(config-if-range)#switchport voice vlan 11
```
> Para que funcione en los Telefonos IP Cisco, debe estar habilitado CDP

```plaintext
Switch#show interfaces f0/4 switchport 
Name: Fa0/4
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 10 (DATA)
Trunking Native Mode VLAN: 1 (default)
Voice VLAN: 11
...
```
En este caso el puerto actua como un puerto de acceso, de hecho el `Operational Mode: static access` indica que efectivamente es un puerto de acceso. Sin embargo otros comandos **show** muestra que se comporta como trunk, agregando un encabezado 802.1Q para las tramas de voz.

Por ejemlo, en el `show interface trunk`, no se muestra nada, sin embargo en el `show interface f0/4 trunk`, si nos muestra que es un enlace troncal.

> Este ultimo comando no funciona en Packet Tracer, peor la guía oficial lo afirma en la pagina 38

 
```plaintext
SW1# show interfaces trunk

SW1# show interfaces F0/4 trunk
Port    Mode    Encapsulation   Status          Native vlan
Fa0/4   off     802.1q          not-trunking    1

Port    Vlans allowed on trunk
Fa0/4   10-11

Port    Vlans allowed and active in management domain
Fa0/4   10-11

Port    Vlans in spanning tree forwarding state and not pruned
Fa0/4   10-11
```
Se ve que el status esta en non-trunking, sin embargo las VLAN 10 y 11 si estan permitidas en el trunk.

## REFERENCIA DE COMANDOS

|Command |Description|
|--------|-----------|
|`vlan vlan-id`|Global config command that both creates the  VLAN and puts the CLI into VLAN configuration mode|
|`name vlan-name`|VLAN subcommand that names the VLAN|
|`[no] shutdown`|VLAN mode subcommand that enables (no shutdown) or disables (shutdown) the VLAN|
|`[no] shutdown vlan vlan-id`|Global config command that has the same effect as the [no] shutdown VLAN mode subcommands|
|`vtp mode {server | client | transparent | off}`|Global config command that defines the VTP mode|
|`switchport mode {access | dynamic {auto | desirable} | trunk}`| Interface subcommand that configures the trunking administrative mode on the interface|
|`switchport access vlan vlan-id` |Interface subcommand that statically configures the interface into that one VLAN|
|`switchport trunk encapsulation {dot1q | isl | negotiate}` |Interface subcommand that defines which type of trunking to use, assuming that trunking is configured or negotiated
|`switchport trunk native vlan vlan-id` |Interface subcommand that defines the native VLAN for a trunk port
|`switchport nonegotiate` |Interface subcommand that disables the negotiation of VLAN trunking
|`switchport voice vlan vlan-id` |Interface subcommand that defines the voice VLAN on a port, meaning that the switch uses 802.1Q tagging for frames in this VLAN
|`switchport trunk allowed vlan {add | all | except | remove} vlan-list` |Interface subcommand that defines the list of allowed VLANs

### Comandos SHOW

|Command |Description|
|--------|-----------|
|`show interfaces interface-id switchport` |Lists information about any interface regarding administrative settings and operational state
|`show interfaces interface-id trunk` |Lists information about all operational trunks (but no other interfaces), including the list of VLANs that can be forwarded over the trunk
|`show vlan [brief | id vlan-id | name vlan-name | summary]` |Lists information about the VLAN
|`show vlan [vlan]` |Displays VLAN information
|`show vtp status` |Lists VTP configuration and status information


