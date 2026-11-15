# 03 — FTP | Servidor vsftpd con usuario restringido

**Entorno:** CentOS Stream y cliente FTP de MobaXterm.

> **Nota de evidencias:** Los comandos son procedimientos propuestos y las salidas descritas son criterios de aceptación. Verificar en CentOS/Windows antes de marcar una prueba como aprobada. Las imágenes conservadas provienen del repositorio aportado.

## 01. Diagnóstico inicial

```bash
ip -br a
rpm -q vsftpd
sudo systemctl status vsftpd
sudo firewall-cmd --list-all
getenforce
```

**Corrección importante:** el original crea `usuario_ftp` pero asigna contraseña a `epnbrow_ftp`; se unifica el nombre como `usuario_ftp`.

## 02. Arquitectura y requisitos

```text
Windows / MobaXterm -- FTP TCP 21 + rango pasivo --> CentOS / vsftpd
                                                   └── usuario_ftp (sin shell)
```

FTP tradicional transmite credenciales sin cifrar. Usar únicamente en red de laboratorio controlada; para redes no confiables, preferir FTPS o SFTP (SFTP pertenece a SSH, no a vsftpd).

## 03. Despliegue y configuración

```bash
sudo dnf install -y vsftpd
sudo systemctl enable --now vsftpd
sudo cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak
sudo useradd -m -s /sbin/nologin usuario_ftp
sudo passwd usuario_ftp
sudo nano /etc/vsftpd/vsftpd.conf
```

Ajustar o agregar **sin duplicar directivas**:

```conf
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES
pasv_enable=YES
pasv_min_port=30000
pasv_max_port=30010
```

Para permitir autenticación FTP de la cuenta sin shell, verificar la configuración PAM del sistema. Si `pam_shells.so` exige un shell listado en `/etc/shells`, puede ser necesario añadir `/sbin/nologin` a esa lista según las políticas del laboratorio. No habilitar shell interactivo al usuario.

```bash
sudo systemctl restart vsftpd
sudo systemctl status vsftpd
```

## 04. Aseguramiento de seguridad (SELinux / Firewalld)

```bash
sudo firewall-cmd --permanent --add-service=ftp
sudo firewall-cmd --permanent --add-port=30000-30010/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
getenforce
sudo getsebool -a | grep ftp
```

Mantener el directorio raíz del usuario no escribible si se utiliza `chroot_local_user=YES`; crear un subdirectorio de carga con permisos apropiados:

```bash
sudo chmod a-w /home/usuario_ftp
sudo mkdir -p /home/usuario_ftp/subidas
sudo chown usuario_ftp:usuario_ftp /home/usuario_ftp/subidas
```

SELinux puede requerir contextos específicos según la ruta de escritura; revisar denegaciones antes de aplicar cambios. No desactivar SELinux para solucionar errores.

## 05. Pruebas de aceptación

En MobaXterm crear sesión **FTP** (no SFTP), IP de CentOS, puerto `21`, usuario `usuario_ftp`. Comprobar conexión, subida y descarga de un archivo de prueba en `subidas/`. Comprobar por separado que el usuario no puede iniciar una sesión SSH interactiva.

**Criterio de aprobación:** autenticación correcta, transferencia en ambos sentidos y acceso a shell denegado. Conservar capturas de cada prueba.

<img width="1418" height="907" alt="image" src="https://github.com/user-attachments/assets/13366408-e56c-44ab-82f8-dc454b5f9cc5" />

## 06. Troubleshooting

| Síntoma | Diagnóstico | Corrección |
|---|---|---|
| `530 Login incorrect` | Usuario, PAM, `/etc/shells` | Verificar contraseña y política de shells |
| Conecta pero no lista archivos | Puertos pasivos | Revisar rango 30000–30010 y firewall |
| `550 Permission denied` | `ls -ld`, SELinux | Corregir permisos y contextos de la ruta |
| `500 OOPS: refusing to run with writable root` | Permisos de raíz chroot | Quitar escritura de la raíz y usar `subidas/` |
