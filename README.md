# Documentación de Red Segmentada con LAN, DMZ y NAT (Cisco Packet Tracer)

Este proyecto simula una red corporativa segmentada con una LAN, una zona DMZ, y conexión a Internet a través de un router ISP. Incluye configuración de NAT, ACLs para control de acceso, servicios como DHCP, Web y DNS, y reglas para proteger la LAN desde la DMZ.

---

## Topología General

* **Router LAN**

  * Gig0/0 → LAN: `192.168.1.1/24`
  * Gig0/1 → ISP: `200.0.0.2/30`
  * Gig0/2 → DMZ: `192.168.2.1/24`
* **Router ISP**

  * Gig0/1 → Enlace a LAN: `200.0.0.1/30`
  * Gig0/0 → Simulación de Internet: `8.8.8.1/24`

---

## NAT y Rutas

### NAT en Router LAN

```bash
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 interface Gig0/1 overload

access-list 2 permit 192.168.2.0 0.0.0.255
ip nat inside source list 2 interface Gig0/1 overload

interface Gig0/0
 ip nat inside
interface Gig0/2
 ip nat inside
interface Gig0/1
 ip nat outside
```

### Ruta por defecto

```bash
ip route 0.0.0.0 0.0.0.0 200.0.0.1
```

---

## Servidores

| Servicio | IP          | Descripción             |
| -------- | ----------- | ----------------------- |
| DHCP     | 192.168.1.2 | Asigna IP a PCs en LAN  |
| Web      | 192.168.2.2 | Sitio web interno (DMZ) |
| DNS      | 192.168.2.3 | Resuelve dominios (DMZ) |

---

## ACLs de Seguridad

### ACL: ACL-DMZ (aplicada en `Gig0/2 in`)

```bash
ip access-list extended ACL-DMZ
 10 permit tcp 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255 established
 20 permit tcp 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
 25 permit icmp 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
 30 permit udp 192.168.1.0 0.0.0.255 host 192.168.2.3 eq domain
 35 permit udp host 192.168.2.3 192.168.1.0 0.0.0.255
 40 deny ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
 50 permit ip any any
```

> Esta ACL garantiza que:
>
> * La LAN puede iniciar conexiones hacia la DMZ
> * La DMZ **no puede iniciar conexiones hacia la LAN** (ni siquiera `ping`)
> * El tráfico relacionado con sesiones ya establecidas sí está permitido

---

## Pruebas Realizadas

* DHCP funcional en LAN
* Acceso desde LAN a servicios web (`192.168.2.2`) y DNS (`192.168.2.3`)
* Resolución de dominios desde LAN mediante servidor DNS en DMZ
* **Bloqueo de ping y conexiones iniciadas desde DMZ hacia LAN**
*  El `ping` desde LAN hacia DMZ también está bloqueado por política ICMP

---

## Consideraciones Finales

* No se usa NAT estático, por lo tanto no se permite acceso desde Internet hacia la LAN o DMZ directamente.
* Las ACL permiten que LAN inicie conexiones hacia DMZ, pero no al revés.
* Se recomienda añadir un firewall para aplicar políticas más granulares.

---

> Proyecto realizado con fines educativos en Cisco Packet Tracer por **Martínez Vega Emiliano**
>
> Portafolio: [https://github.com/portgasdvega](https://github.com/portgasdvega)
