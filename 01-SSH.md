# 01 — SSH | Acceso remoto seguro

**Integrantes:** Francisco Morillo y Esteban Nievecela  
**Entorno:** CentOS Stream en VirtualBox (adaptador puente), Windows y MobaXterm.  
**Puerto del proyecto:** `2237/TCP`.

> **Nota de evidencias:** Los comandos son procedimientos propuestos y las salidas descritas son criterios de aceptación. Verificar en CentOS/Windows antes de marcar una prueba como aprobada. Las imágenes conservadas provienen del repositorio aportado.

## 01. Diagnóstico inicial

```bash
ip -br a
sudo systemctl status sshd
sudo firewall-cmd --state
getenforce
sudo ss -lntp | grep sshd
```

Confirmar que CentOS y Windows están en la misma red y que el acceso SSH por el puerto anterior funciona **antes** de modificarlo.

## 02. Arquitectura y requisitos

```text
Windows / MobaXterm -- SSH TCP 2237 --> CentOS Stream (sshd)
                         Red puente de VirtualBox
```

SSH cifra la administración remota. Requiere `openssh-server`, conectividad IP y permisos en Firewalld y SELinux.

## 03. Despliegue y configuración

```bash
sudo dnf install -y openssh-server policycoreutils-python-utils
sudo systemctl enable --now sshd
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo nano /etc/ssh/sshd_config
```

Establecer una única directiva efectiva `Port 2237` (revisar también archivos en `sshd_config.d`). Validar antes de reiniciar:

```bash
sudo sshd -t
sudo semanage port -a -t ssh_port_t -p tcp 2237
# Si el puerto ya está registrado con otro tipo, investigar antes de cambiarlo.
sudo firewall-cmd --permanent --add-port=2237/tcp
sudo firewall-cmd --reload
sudo systemctl restart sshd
sudo ss -lntp | grep 2237
```

Mantener la sesión SSH actual abierta hasta verificar la nueva conexión. No retirar el puerto 22 del firewall hasta confirmar acceso por 2237.

## 04. Aseguramiento de seguridad (SELinux / Firewalld)

```bash
sudo semanage port -l | grep ssh_port_t
sudo firewall-cmd --list-ports
getenforce
sudo journalctl -u sshd -n 30 --no-pager
```

Conservar SELinux en modo `Enforcing`. Restringir el acceso SSH a equipos autorizados cuando sea posible.

## 05. Pruebas de aceptación

Desde Windows o MobaXterm, configurar host = IP actual de CentOS, usuario = cuenta existente, puerto = `2237`. Alternativamente:

```bash
ssh -p 2237 usuario@IP_CENTOS
```

**Criterio de aprobación:** se solicita autenticación y se abre una terminal remota; `ss` confirma escucha en 2237. Registrar captura de MobaXterm y salida de comandos.

<img width="577" height="186" alt="image" src="https://github.com/user-attachments/assets/c7447515-4ee3-41cf-93d6-1b194b0bf4c3" />

## 06. Troubleshooting

| Síntoma | Diagnóstico | Corrección |
|---|---|---|
| `Connection refused` | `systemctl status sshd`; `ss -lntp` | Revisar configuración y reiniciar servicio |
| Tiempo de espera | `firewall-cmd --list-all`, IP | Revisar red puente y puerto TCP 2237 |
| SELinux bloquea puerto | `semanage port -l` | Registrar 2237 como `ssh_port_t` |
| No autentica | `journalctl -u sshd` | Revisar usuario, contraseña o claves |
