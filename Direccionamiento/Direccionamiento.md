# Direccionamiento IP

## Objetivo

Documentar el direccionamiento IPv4 utilizado en el laboratorio de autenticación, autorización y accounting (AAA) con Cisco IOS, Windows Server 2019 NPS y el cliente Windows 10 utilizado para las pruebas.

## Plan de direccionamiento

| Dispositivo | Interfaz | Dirección IPv4 | Máscara | Puerta de enlace | DNS | Función |
|---|---|---:|---|---|---|---|
| R1 | FastEthernet0/0 | 10.21.40.1 | 255.255.255.0 (/24) | — | — | Router y puerta de enlace de la red |
| SW1 | VLAN 1 | 10.21.40.2 | 255.255.255.0 (/24) | 10.21.40.1 | — | Administración del switch |
| Windows Server 2019 | Interfaz de red | 10.21.40.10 | 255.255.255.0 (/24) | Según la configuración del laboratorio | — | Servidor NPS/RADIUS y Active Directory |
| Windows10-Usuario-1 | Interfaz de red | 10.21.40.20 | 255.255.255.0 (/24) | 10.21.40.1 | 10.21.40.10 | Cliente Windows 10 para realizar pruebas de acceso y autenticación |

## Parámetros RADIUS

| Parámetro | Valor |
|---|---|
| Servidor RADIUS/NPS | 10.21.40.10 |
| Puerto de autenticación | UDP 1812 |
| Puerto de accounting | UDP 1813 |
| Dirección NAS de R1 | 10.21.40.1 |
| Dirección NAS de SW1 | 10.21.40.2 |
| Dirección de origen de las pruebas SSH | 10.21.40.20 (Windows10-Usuario-1) |

## Enrutamiento

Los dispositivos documentados pertenecen a la misma red IPv4: **10.21.40.0/24**. Por ello, la comunicación entre R1, SW1, Windows Server 2019 y el cliente Windows 10 ocurre directamente dentro de la subred y no requiere rutas estáticas entre ellos.

En SW1 se utiliza como puerta de enlace de administración **10.21.40.1** (R1), mediante el comando:

```text
ip default-gateway 10.21.40.1
```

En Windows10-Usuario-1, la puerta de enlace es **10.21.40.1** y el servidor DNS configurado es **10.21.40.10**.

En R1, la red conectada directamente es **10.21.40.0/24**, a través de FastEthernet0/0.

> Nota: las direcciones y puertos se documentan a partir de la configuración y las pruebas mostradas durante el laboratorio. No se incluyen secretos RADIUS, contraseñas ni hashes de credenciales.
