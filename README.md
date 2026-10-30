# Lab-AAA-Radius-Cisco-NPS

Laboratorio de seguridad de red desarrollado en **GNS3** utilizando **Cisco IOS**, **Windows Server 2019 (Active Directory + NPS/RADIUS)** y un cliente **Windows 10**.

El objetivo principal es implementar **autenticación, autorización y accounting (AAA)** centralizados mediante RADIUS (NPS), con segmentación de privilegios por grupos de Active Directory (niveles 0, 5 y 10), acceso SSH restringido y registro de todos los eventos de acceso.

## Video de demostración

> (https://www.youtube.com/watch?v=REEMPLAZAR_CON_TU_ENLACE)

---

## Topología

### Topología real en GNS3

![Topología real en GNS3](Topologia/Topologia.png)

### Diagrama de la topología

![Diagrama de topología](Topologia/Diagrama.png)

La topología está compuesta por:

- **R1** – Router Cisco IOS, gateway de la red y cliente RADIUS.
- **SW1** – Switch Cisco IOSvL2, administración y cliente RADIUS.
- **Windows Server 2019** – Controlador de dominio (Active Directory) y servidor NPS/RADIUS.
- **Windows 10** – Equipo cliente para las pruebas de acceso SSH.

### Relación física principal

```text
Windows 10 (10.21.40.20)
   |
   | SSH
   v
R1 (10.21.40.1)  <----RADIUS---->  Windows Server 2019 (10.21.40.10)
   |                                     |
   |                                     +-- Active Directory (seguridad.lab)
   |                                     +-- NPS (POL_RADIUS_NIVEL0/5/10)
   |
SW1 (10.21.40.2) <----RADIUS----> Windows Server 2019 (10.21.40.10)
```

---

## Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway | Función |
|---|---|---|---|---|---|
| R1 | FastEthernet0/0 | 10.21.40.1 | 255.255.255.0 (/24) | — | Router y puerta de enlace |
| SW1 | VLAN 1 | 10.21.40.2 | 255.255.255.0 (/24) | 10.21.40.1 | Administración del switch |
| Windows Server 2019 | Interfaz de red | 10.21.40.10 | 255.255.255.0 (/24) | 10.21.40.1 | Servidor NPS/RADIUS y Active Directory |
| Windows 10 (cliente) | Interfaz de red | 10.21.40.20 | 255.255.255.0 (/24) | 10.21.40.1 | Equipo de pruebas SSH |

Red: `10.21.40.0/24`

### Parámetros RADIUS

| Parámetro | Valor |
|---|---|
| Servidor RADIUS/NPS | 10.21.40.10 |
| Puerto de autenticación | UDP 1812 |
| Puerto de accounting | UDP 1813 |
| NAS de R1 | 10.21.40.1 |
| NAS de SW1 | 10.21.40.2 |
| Origen de pruebas SSH | 10.21.40.20 |

La documentación completa se encuentra en:

[Direccionamiento/Direccionamiento.md](Direccionamiento/Direccionamiento.md)

---

## Configuración del Router (R1)

Se configuró AAA con RADIUS (NPS) y fallback a usuario local.

### AAA implementado

- `aaa new-model`
- Autenticación: `RAD_LOGIN` (RADIUS) con fallback `local`.
- Autorización EXEC: `RAD_EXEC` (RADIUS) con fallback `local`.
- Accounting EXEC: `RAD_ACCOUNT` (start-stop) hacia RADIUS.
- Usuario local: `adminlocal` (privilegio 15).
- AAA aplicado a puertos de consola y auxiliar (vty usa RADIUS para SSH).
- Logs de autenticación/autorización habilitados (`login on-failure/success log`).
- Servicio SSH v2 habilitado (`ip ssh version 2`).
- RADIUS: servidor `10.21.40.10`, puertos UDP 1812 (auth) / 1813 (acct).

[Running-config de R1](Show%20running-config/Show%20running-config%20Router%20R1.txt)

## Configuración del Switch (SW1)

- `aaa new-model`
- Autenticación: `RAD_LOGIN` (grupo RADIUS `GRP_NPS`) con fallback `local`.
- Autorización EXEC: `RAD_EXEC` con fallback `local`.
- Accounting EXEC: `RAD_ACCOUNT` (start-stop) hacia RADIUS.
- Usuario local: `adminlocal` (privilegio 15).
- AAA aplicado a puertos de consola y auxiliar.
- Logs de autenticación/autorización habilitados.
- Servicio SSH v2 habilitado.
- RADIUS: servidor NPS `10.21.40.10`, puertos UDP 1812 (auth) / 1813 (acct).

[Running-config de SW1](Show%20running-config/Show%20running-config%20Switch%20SW1.txt)

---

## Servidor (Windows Server 2019)

### Active Directory

- Dominio: `seguridad.lab`
- Unidad organizativa: `OU_RADIUS`
- Grupos creados:
  - `GRP_RADIUS_NIVEL0`
  - `GRP_RADIUS_NIVEL5`
  - `GRP_RADIUS_NIVEL10`

### NPS / RADIUS

Clientes RADIUS registrados:

| Cliente | IP | Estado |
|---|---|---|
| R1_ROUTER | 10.21.40.1 | Habilitado |
| SW1_SWITCH | 10.21.40.2 | Habilitado |

Políticas de red configuradas:

- `POL_RADIUS_NIVEL0`
- `POL_RADIUS_NIVEL5`
- `POL_RADIUS_NIVEL10` (condición: pertenencia a `SEGURIDAD\GRP_RADIUS_NIVEL10`)

La configuración gráfica del servidor se documenta en:

[Configuración Windows Server (PDF)](Configuracion%20windows%20server/Configuracion%20windows%20server%20.pdf)

---

## Evidencias y pruebas realizadas

Las evidencias documentan, entre otras, las siguientes validaciones:

- Topología general del laboratorio (R1, SW1, servidor Windows, cliente Windows 10).
- Configuración IPv4 estática del servidor (10.21.40.10).
- Usuarios y grupos de Active Directory (`OU_RADIUS`, niveles 0/5/10).
- Clientes RADIUS registrados en NPS (R1 y SW1).
- Políticas de red habilitadas (NIVEL0, NIVEL5, NIVEL10).
- Inicio de sesión SSH con usuario de nivel 0 (modo EXEC reducido).
- Prueba SSH con usuario de nivel 5.
- Registros de contabilidad (accounting) de NPS (eventos hacia R1 y SW1).
- Resultado de pruebas `test aaa group` (autenticación exitosa, privilegios 0/5/10).
- Conexión SSH al router R1 desde Windows.

[Ver documento de evidencias](Evidencias/Evidencias.pdf) *(si aplica)*

---

## Running Config

Se incluyen las configuraciones actuales de los dispositivos:

- [R1 running-config](Show%20running-config/Show%20running-config%20Router%20R1.txt)
- [SW1 running-config](Show%20running-config/Show%20running-config%20Switch%20SW1.txt)

> **Nota de seguridad:** los archivos publicados en `Show running-config/` fueron sanitizados. Las contraseñas, hashes y otros valores sensibles se reemplazaron por `<REDACTED>` o se omitieron del extracto público.

---

## Estructura del repositorio

```text
Lab-AAA-Radius-Cisco-NPS/
├── README.md
├── Configuracion windows server/
│   └── Configuracion windows server .pdf
├── Direccionamiento/
│   └── Direccionamiento.md
├── Evidencias/
│   └── Evidencias.pdf
├── Show running-config/
│   ├── Show running-config Router R1.txt
│   └── Show running-config Switch SW1.txt
└── Topologia/
    ├── Diagrama.png
    └── Topologia.png
```

---

## Tecnologías utilizadas

- GNS3
- Cisco IOS / IOSvL2 15.2
- Windows Server 2019 (Active Directory + NPS)
- Windows 10
- RADIUS (UDP 1812/1813)
- SSH v2

---

## Objetivos cumplidos

El laboratorio demuestra una arquitectura de seguridad basada en **AAA centralizado con RADIUS (NPS)**, utilizando **Cisco IOS** y **Windows Server 2019**.

La implementación aplica autenticación y autorización en los dispositivos de red, restringe el acceso a puertos de consola y auxiliar, diferencia privilegios por grupos de Active Directory (niveles 0, 5 y 10), habilita el servicio SSH y registra (accounting) todos los eventos de acceso hacia el servidor NPS.

---

## Autor

**Carlos-2140**

Proyecto desarrollado como práctica de configuración y seguridad de redes con Cisco IOS, Active Directory y NPS/RADIUS en GNS3.