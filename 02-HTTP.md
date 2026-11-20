# Servicio 02 — HTTP | Servidor web Apache

**Entorno:** CentOS Stream / httpd y navegador de Windows.

> **Nota de evidencias:** Los comandos son procedimientos propuestos y las salidas descritas son criterios de aceptación. Verificar en CentOS/Windows antes de marcar una prueba como aprobada. Las imágenes conservadas provienen del repositorio aportado.

## 01. Diagnóstico inicial


ip -br a
rpm -q httpd
sudo systemctl status httpd
sudo ss -lntp | grep ':80'
getenforce


## 02. Arquitectura y requisitos


Windows / navegador -- HTTP TCP 80 --> CentOS / Apache httpd
                                        └── /var/www/html/index.html
                                            /var/www/html/images.jpg

## 03. Despliegue y configuración

sudo dnf install -y httpd
sudo systemctl enable --now httpd
httpd -v
sudo mkdir -p /var/www/html
# Si ya existe una página personalizada, respaldarla antes de editar:
[ ! -f /var/www/html/index.html ] || sudo cp /var/www/html/index.html /var/www/html/index.html.bak
sudo nano /var/www/html/index.html

Ejemplo de contenido para index.html:

html
<!doctype html>
<html lang="es">
<head><meta charset="utf-8"><title>Levantamiento de servidores</title></head>
<body>
  <h1>Práctica 3 — Servidor HTTP</h1>
  <p>Francisco Morillo y Esteban Nievecela</p>
  <img src="/images.jpg" alt="Imagen institucional" width="160">
</body>
</html>
```

Si `images.jpg` ya está en `~/Imagenes/`:

```bash
sudo cp ~/Imagenes/images.jpg /var/www/html/images.jpg
sudo chmod 644 /var/www/html/index.html /var/www/html/images.jpg
sudo restorecon -Rv /var/www/html
```

## 04. Aseguramiento de seguridad (SELinux / Firewalld)

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
sudo firewall-cmd --list-services
ls -Z /var/www/html/
getenforce
sudo apachectl configtest
```

El contenido estático debe tener contexto SELinux apropiado (`httpd_sys_content_t`). Abrir `https`/443 solo si se configura TLS realmente.

## 05. Pruebas de aceptación

```bash
curl -I http://localhost/
curl -I http://IP_CENTOS/
curl -I http://IP_CENTOS/images.jpg
```

Desde Windows abrir `http://IP_CENTOS/` y verificar título, integrantes e imagen. **Criterio de aprobación:** respuestas HTTP `200 OK` para página e imagen y visualización correcta en el navegador.

<img src="/noindex/images.jpg" alt="UCUENCATEC" style="width:110px; margin-left:15px; vertical-align:middle;">
<img width="1550" height="362" alt="image" src="https://github.com/user-attachments/assets/2b97b190-c209-4688-abee-023426ff9d65" />
<img width="1492" height="855" alt="image" src="https://github.com/user-attachments/assets/27c78b6c-0625-49ad-b11e-1b4755c7a635" />

## 06. Troubleshooting

| Síntoma | Diagnóstico | Corrección |
|---|---|---|
| No abre desde Windows | `ss`, firewall, IP | Revisar httpd, puerto 80 y red puente |
| Error `403 Forbidden` | `ls -lZ`, `journalctl -u httpd` | Revisar permisos y `restorecon` |
| Imagen rota | `curl -I /images.jpg` | Corregir ruta HTML y nombre del archivo |
| Página de bienvenida en vez del sitio | `ls /var/www/html` | Crear `index.html` en DocumentRoot |
