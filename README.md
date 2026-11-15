# Levantamiento de Servidores — Práctica 3

**Asignatura:** Sistemas Operativos I (Linux)  
**Integrantes:** Francisco Morillo y Esteban Nievecela  
**Plataforma:** CentOS Stream en VirtualBox, Windows y MobaXterm.

## Navegación por servicios

| Servicio | Documentación | Puerto |
|---|---|---|
| SSH | [01 — SSH](01-SSH.md) | 2237/TCP |
| NTP | [02 — NTP](02-NTP.md) | 123/UDP |
| FTP | [03 — FTP](03-FTP.md) | 21/TCP y 30000–30010/TCP |
| HTTP | [04 — HTTP](04-HTTP.md) | 80/TCP |

## 5.1. Arquitectura y topología de red (descripción textual)

La práctica utiliza una máquina virtual con CentOS Stream instalada en VirtualBox. El adaptador de red se configura en modo puente para que el servidor y el equipo anfitrión Windows puedan comunicarse dentro de la red local.

Desde Windows se utiliza MobaXterm para administrar el servidor mediante SSH por el puerto TCP 2237 y para realizar las conexiones de transferencia de archivos. El servicio FTP, implementado con vsftpd, recibe conexiones por el puerto TCP 21 y utiliza el rango TCP 30000–30010 para las conexiones pasivas configuradas. El navegador del cliente accede a la página web alojada en Apache HTTP Server mediante el puerto TCP 80. Para la sincronización horaria, Windows consulta el servicio NTP proporcionado por Chrony en CentOS mediante el puerto UDP 123.

Los servicios se ejecutan en el mismo servidor CentOS y están protegidos mediante reglas de Firewalld y políticas de SELinux. Los puertos indicados corresponden a la configuración documentada y deben verificarse en el entorno de laboratorio.

## 5.2. Matriz de puertos y seguridad

| Servicio | Puerto | Firewalld | SELinux / control |
|---|---|---|---|
| SSH | 2237/TCP | Puerto 2237 | `ssh_port_t` |
| NTP | 123/UDP | Servicio `ntp` | Limitar clientes con `allow` en Chrony |
| FTP | 21/TCP, 30000–30010/TCP | Servicio `ftp` y rango pasivo | Contextos y permisos del directorio |
| HTTP | 80/TCP | Servicio `http` | Contexto `httpd_sys_content_t` |

> El puerto 443/TCP se requiere solo si se configura HTTPS. No se declara habilitado en este procedimiento.

## 5.3. Pruebas de aceptación

Cada documento incluye comandos, resultados esperados y las capturas originales disponibles. **No se afirma que las pruebas se hayan repetido o validado en esta revisión.** Agregar evidencias de ejecución real para cada servicio.

## 5.4. Plan de contingencia

1. **Fallo de conectividad:** comprobar IP, adaptador puente, puertos y estado del servicio; conservar sesión administrativa previa.
2. **Bloqueo de seguridad:** consultar logs y SELinux; corregir reglas y contextos sin desactivar la protección.
3. **Fallo de sincronización o transferencia:** verificar configuración Chrony/FTP, permisos, fuentes de tiempo y puertos pasivos.

## 5.5. Organización de la wiki

- [01 — SSH](01-SSH.md)
- [02 — NTP](02-NTP.md)
- [03 — FTP](03-FTP.md)
- [04 — HTTP](04-HTTP.md)

En cada página: diagnóstico, arquitectura, despliegue, aseguramiento, pruebas y troubleshooting.

## Pendientes de comprobación

- Confirmar IP de CentOS y subred del laboratorio (el original muestra dos IP diferentes).
- Verificar conexión real SSH por 2237 y denegación de shell del usuario FTP.
- Confirmar sincronización NTP mediante `w32tm` y `chronyc`.
- Validar subida/descarga FTP y carga HTTP de la imagen.
