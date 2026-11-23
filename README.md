# 🌐 utp-redes-lab1 — Subnetting IPv4 y segmentación con VLAN

![Estado](https://img.shields.io/badge/estado-completado-2ea043)
![Herramienta](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?logo=cisco&logoColor=white)
![Licencia](https://img.shields.io/badge/licencia-MIT-blue)

Laboratorio académico de la Licenciatura en Ciberseguridad — **Universidad Tecnológica de Panamá (UTP), FISC**.
Autor: **Ethan Beltrán** · 2026

**Documentación publicada en GitHub Pages:** https://EthanBeltranutp.github.io/utp-redes-lab1/

---

## Propósito

Diseñar el direccionamiento IPv4 de una pequeña oficina dividiendo una red `/24` en cuatro subredes, y aislar cada departamento en su propia VLAN. Separar la red en segmentos es una medida básica de seguridad: limita el alcance de un incidente y permite aplicar controles por área.

## Escenario

Una oficina necesita cuatro segmentos independientes:

| VLAN | Departamento   | Necesidad                    |
|------|----------------|------------------------------|
| 10   | Administración | Equipos de personal          |
| 20   | Ventas         | Equipos de atención          |
| 30   | TI             | Servidores y administración  |
| 40   | Invitados      | Acceso aislado para visitas  |

Red asignada: **192.168.10.0/24**

## Topología

```
                 [ R1 - Router 2911 ]
                        | G0/0  (subinterfaces .10 .20 .30 .40)
                        | enlace troncal 802.1Q
                        | G0/1
                 [ SW1 - Switch 2960 ]
       Fa0/1-5     Fa0/6-10     Fa0/11-15     Fa0/16-20
          |            |             |             |
      VLAN 10      VLAN 20       VLAN 30       VLAN 40
       PC-ADM      PC-VEN        PC-TI         PC-INV
```

Diseño *router-on-a-stick*: un solo enlace troncal transporta las cuatro VLAN y el router hace el enrutamiento entre ellas.

## Cálculo de subredes

Se necesitan 4 subredes → se toman **2 bits** prestados (2² = 4).
Nueva máscara: **/26 = 255.255.255.192** · Bloque de 64 direcciones · **62 hosts útiles** por subred (2⁶ − 2).

| Subred | VLAN | Dirección de red  | Rango de hosts               | Broadcast       | Gateway        |
|--------|------|-------------------|------------------------------|-----------------|----------------|
| 1      | 10   | 192.168.10.0/26   | 192.168.10.1 – 192.168.10.62    | 192.168.10.63   | 192.168.10.1   |
| 2      | 20   | 192.168.10.64/26  | 192.168.10.65 – 192.168.10.126  | 192.168.10.127  | 192.168.10.65  |
| 3      | 30   | 192.168.10.128/26 | 192.168.10.129 – 192.168.10.190 | 192.168.10.191  | 192.168.10.129 |
| 4      | 40   | 192.168.10.192/26 | 192.168.10.193 – 192.168.10.254 | 192.168.10.255  | 192.168.10.193 |

## Procedimiento

### 1. Switch SW1 — VLAN y puertos

```
enable
configure terminal
hostname SW1
vlan 10
 name ADMINISTRACION
vlan 20
 name VENTAS
vlan 30
 name TI
vlan 40
 name INVITADOS
exit
interface range fa0/1-5
 switchport mode access
 switchport access vlan 10
interface range fa0/6-10
 switchport mode access
 switchport access vlan 20
interface range fa0/11-15
 switchport mode access
 switchport access vlan 30
interface range fa0/16-20
 switchport mode access
 switchport access vlan 40
interface g0/1
 switchport mode trunk
end
write memory
```

### 2. Router R1 — subinterfaces

```
enable
configure terminal
hostname R1
interface g0/0
 no shutdown
interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.192
interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.10.65 255.255.255.192
interface g0/0.30
 encapsulation dot1Q 30
 ip address 192.168.10.129 255.255.255.192
interface g0/0.40
 encapsulation dot1Q 40
 ip address 192.168.10.193 255.255.255.192
end
write memory
```

### 3. Equipos finales

| Equipo | IP              | Máscara           | Gateway        |
|--------|-----------------|-------------------|----------------|
| PC-ADM | 192.168.10.10   | 255.255.255.192   | 192.168.10.1   |
| PC-VEN | 192.168.10.70   | 255.255.255.192   | 192.168.10.65  |
| PC-TI  | 192.168.10.140  | 255.255.255.192   | 192.168.10.129 |
| PC-INV | 192.168.10.200  | 255.255.255.192   | 192.168.10.193 |

## Verificación

| Prueba | Comando | Resultado esperado |
|--------|---------|--------------------|
| VLAN creadas | `show vlan brief` (SW1) | VLAN 10, 20, 30, 40 con sus puertos |
| Enlace troncal | `show interfaces trunk` (SW1) | G0/1 en modo trunk |
| Subinterfaces | `show ip interface brief` (R1) | G0/0.10–.40 *up/up* |
| Conectividad | `ping 192.168.10.1` desde PC-ADM | Respuesta exitosa |
| Enrutamiento entre VLAN | `ping 192.168.10.140` desde PC-ADM | Respuesta exitosa |

## Reflexión de seguridad

- La segmentación evita que un equipo comprometido en *Invitados* vea directamente el tráfico de *Administración* en capa 2.
- Como mejora futura se puede aplicar una **ACL** en `g0/0.40` para que Invitados solo salga a Internet y no alcance las demás VLAN.
- Los puertos no usados del switch deberían apagarse (`shutdown`) y asignarse a una VLAN sin uso.

## Estructura del repositorio

```
utp-redes-lab1/
├── README.md        ← esta documentación
├── index.html       ← sitio publicado con GitHub Pages
├── docs/            ← capturas y archivo .pkt del laboratorio
├── .gitignore
└── LICENSE          ← Licencia MIT
```

## 📄 Licencia

Distribuido bajo la [Licencia MIT](LICENSE).
