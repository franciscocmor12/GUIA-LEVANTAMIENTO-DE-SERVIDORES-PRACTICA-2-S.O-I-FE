# 02 — NTP | Sincronización horaria con Chrony

**Entorno:** CentOS Stream (servidor) y Windows (cliente) en red puente.

> **Nota de evidencias:** Los comandos son procedimientos propuestos y las salidas descritas son criterios de aceptación. Verificar en CentOS/Windows antes de marcar una prueba como aprobada. Las imágenes conservadas provienen del repositorio aportado.

## 01. Diagnóstico inicial

```bash
ip -br a
timedatectl
rpm -q chrony
systemctl status chronyd
chronyc tracking
```

**Observación del material original:** aparecen `192.168.3.28` y `192.168.18.198` como IP de CentOS; son distintas. Se debe usar **la IP real actual** en todos los pasos.

## 02. Arquitectura y requisitos

```text
Fuentes horarias externas --> CentOS / chronyd (UDP 123)
                                      |
                             red puente VirtualBox
                                      |
                           Windows / servicio w32time
```

CentOS actúa como servidor de hora para Windows; necesita una fuente horaria confiable y la red cliente autorizada.

## 03. Despliegue y configuración

```bash
sudo dnf install -y chrony
sudo systemctl enable --now chronyd
sudo cp /etc/chrony.conf /etc/chrony.conf.bak
sudo nano /etc/chrony.conf
```

Añadir al archivo **una línea** que autorice solo la subred del laboratorio, sustituyendo el ejemplo:

```conf
allow 192.168.3.0/24
```

No añadir la línea mediante `tee -a` sin comprobar antes si ya existe. Reiniciar:

```bash
sudo systemctl restart chronyd
chronyc sources -v
chronyc tracking
```

En Windows, abrir PowerShell **como administrador** y sustituir `IP_CENTOS` por la IP actual:

```powershell
Set-Service -Name w32time -StartupType Automatic
Start-Service -Name w32time
w32tm /config /manualpeerlist:"IP_CENTOS,0x8" /syncfromflags:manual /update
w32tm /resync /rediscover
w32tm /query /source
w32tm /query /status
```

En equipos administrados por dominio, la política corporativa puede sobrescribir la configuración de hora.

## 04. Aseguramiento de seguridad (SELinux / Firewalld)

```bash
sudo firewall-cmd --permanent --add-service=ntp
sudo firewall-cmd --reload
sudo firewall-cmd --list-services
getenforce
sudo ss -lunp | grep ':123'
```

Restringir la directiva `allow` a la subred autorizada; no abrir NTP a redes innecesarias.

## 05. Pruebas de aceptación

```bash
chronyc tracking
chronyc clients
```

```powershell
w32tm /query /source
w32tm /query /status
w32tm /stripchart /computer:IP_CENTOS /samples:5 /dataonly
```

**Criterio de aprobación:** Windows identifica la fuente CentOS y registra intercambio de tiempo. Una captura de la hora coincidente por sí sola no demuestra sincronización NTP.

<img width="507" height="96" alt="image" src="https://github.com/user-attachments/assets/c133d221-682a-429f-ab7d-27dbb93f988e" />
<img width="658" height="480" alt="Captura de pantalla 2026-11-15 170918" src="https://github.com/user-attachments/assets/9dfc6b8e-9436-41e2-9f33-4a81c51c85c5" />
<img width="477" height="327" alt="Captura de pantalla 2026-11-15 171412" src="https://github.com/user-attachments/assets/1d30aa8b-eb14-43a2-9dd7-a2ab1a5ea79c" />
<img width="423" height="511" alt="image" src="https://github.com/user-attachments/assets/0f3eb511-f36a-4bb8-adae-53937810111e" />

## 06. Troubleshooting

| Síntoma | Diagnóstico | Corrección |
|---|---|---|
| Windows no sincroniza | `w32tm /query /source` | Revisar IP, servicio `w32time` y UDP 123 |
| CentOS sin fuente válida | `chronyc sources -v` | Revisar conectividad y servidores upstream |
| Cliente no autorizado | `chronyc clients`, `chrony.conf` | Corregir la subred `allow` |
| Hora no cambia | `w32tm /resync /rediscover` | Revisar permisos y políticas de Windows |
