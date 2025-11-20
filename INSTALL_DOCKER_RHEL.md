# Manual de Instalación y Configuración de Docker Engine en RHEL

Este documento detalla los pasos para instalar Docker Engine en Red Hat Enterprise Linux (RHEL versiones 8, 9 o 10), configurar el servicio y habilitar su uso sin privilegios de root.

**Aplicable a:** RHEL 8/9/10

---

## 1. Requisitos Previos

- Acceso a una terminal con privilegios `sudo` o `root`
- Conexión a internet
- Un sistema RHEL de 64 bits (x86_64, aarch64 o s390x)
- Sistema operativo actualizado

---

## 2. Limpieza del Sistema

Antes de comenzar, es crítico eliminar versiones antiguas de Docker o paquetes conflictivos como Podman (que viene preinstalado en muchas versiones de RHEL).

Ejecuta el siguiente comando:

```bash
sudo dnf remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine \
    podman \
    runc
```

> **Nota:** Es normal si `dnf` reporta que no hay paquetes instalados. Si tenías datos en `/var/lib/docker/`, estos no se borrarán automáticamente.

---

## 3. Instalación (Método de Repositorio)

Este es el método recomendado para facilitar las actualizaciones futuras.

### Paso 3.1: Configurar el repositorio

Instala las herramientas necesarias y agrega el repositorio oficial de Docker:

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

### Paso 3.2: Instalar los paquetes

Instala la última versión de Docker Engine, la CLI, containerd y los plugins necesarios:

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Si aparece un error de clave GPG**, acepta la clave cuando se solicite o importala manualmente:

```bash
sudo rpm --import https://download.docker.com/linux/centos/gpg
```

### Paso 3.3: Iniciar el servicio

Una vez instalado, inicia Docker y habilítalo para que arranque con el sistema:

```bash
sudo systemctl enable --now docker
```

Este comando hace dos cosas:
- `enable`: Configura Docker para iniciar automáticamente al arrancar el sistema
- `--now`: Inicia el servicio inmediatamente

**Verificar el estado:**

```bash
sudo systemctl status docker
```

---

## 4. Post-Instalación (Gestión de Usuarios)

Por defecto, Docker solo puede ser ejecutado por el usuario `root`. Para permitir que tu usuario actual ejecute comandos Docker sin usar `sudo`, sigue estos pasos:

### Paso 4.1: Crear el grupo y asignar usuario

1. Crea el grupo `docker` (si no existe ya):

```bash
sudo groupadd docker
```

2. Agrega tu usuario actual al grupo:

```bash
sudo usermod -aG docker $USER
```

### Paso 4.2: Aplicar los cambios

Para que los cambios de grupo surtan efecto, tienes dos opciones:

- **Opción A:** Cerrar sesión y volver a entrar en el sistema
- **Opción B:** Ejecutar el siguiente comando para activar el grupo en la sesión actual:

```bash
newgrp docker
```

### Paso 4.3: Verificar permisos

Ejecuta un comando Docker sin `sudo` para confirmar que funciona:

```bash
docker ps
```

Si no recibes un error de permisos, la configuración es correcta.

---

## 5. Configuración de Logs (Recomendado)

Por defecto, Docker no limita el tamaño de los logs de los contenedores, lo que puede llenar el disco duro del servidor. Se recomienda configurar la rotación de logs.

### Paso 5.1: Crear archivo de configuración

1. Crea o edita el archivo `daemon.json`:

```bash
sudo mkdir -p /etc/docker
sudo nano /etc/docker/daemon.json
```

2. Agrega el siguiente contenido para limitar los logs a 10MB por archivo y un máximo de 3 archivos:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

3. Guarda el archivo (`Ctrl+O`, `Enter`, `Ctrl+X` en nano)

### Paso 5.2: Aplicar cambios

Reinicia Docker para aplicar la configuración:

```bash
sudo systemctl restart docker
```

**Verificar que el servicio inició correctamente:**

```bash
sudo systemctl status docker
```

---

## 6. Configuración de SELinux

RHEL viene con SELinux habilitado por defecto. Docker es compatible con SELinux, pero debes asegurarte de que las políticas estén correctamente configuradas.

### Verificar estado de SELinux

```bash
getenforce
```

Si muestra `Enforcing`, SELinux está activo (recomendado para producción).

### Etiquetar directorios para volúmenes (si es necesario)

Si tienes problemas de permisos con volúmenes, puedes etiquetar los directorios:

```bash
# Ejemplo: permitir que Docker acceda a un directorio específico
sudo chcon -Rt svirt_sandbox_file_t /path/to/volume
```

### Deshabilitar SELinux (NO recomendado para producción)

Solo si tienes problemas graves y estás en un ambiente de desarrollo:

**Temporalmente:**

```bash
sudo setenforce 0
```

**Permanentemente** (requiere reinicio):

```bash
sudo sed -i 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
sudo reboot
```

---

## 7. Configuración de Firewall (firewalld)

Si `firewalld` está activo, debes permitir el tráfico de Docker.

### Verificar si firewalld está activo

```bash
sudo systemctl status firewalld
```

### Configurar Docker en firewalld

```bash
# Agrega Docker a la zona trusted
sudo firewall-cmd --permanent --zone=trusted --add-interface=docker0
sudo firewall-cmd --permanent --zone=trusted --add-masquerade

# Recarga la configuración
sudo firewall-cmd --reload
```

### Exponer puertos de contenedores específicos

Si necesitas exponer puertos de tus aplicaciones:

```bash
# Ejemplo: permitir puerto 8080
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

**Ver puertos abiertos:**

```bash
sudo firewall-cmd --list-all
```

---

## 8. Verificación Final

Para confirmar que todo está correctamente instalado y configurado (y que puedes ejecutar Docker sin `sudo`), ejecuta la imagen de prueba:

```bash
docker run hello-world
```

**Resultado esperado:**

Deberías ver un mensaje que comienza con:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### Verificación adicional

Ejecuta un contenedor de prueba con puerto expuesto:

```bash
docker run -d -p 8080:80 --name test-nginx nginx
curl http://localhost:8080
docker stop test-nginx
docker rm test-nginx
```

**Salida esperada de curl:** HTML de la página de bienvenida de NGINX.

---

## 9. Comandos Útiles de Docker

### Gestión del servicio

```bash
# Ver estado
sudo systemctl status docker

# Iniciar
sudo systemctl start docker

# Detener
sudo systemctl stop docker

# Reiniciar
sudo systemctl restart docker

# Ver logs del servicio
sudo journalctl -u docker
```

### Verificar versión

```bash
docker --version
docker compose version
```

### Información del sistema

```bash
docker info
docker system df
```

---

## 10. Solución de Problemas

### Error: "Cannot connect to the Docker daemon"

**Causa:** El servicio Docker no está corriendo.

**Solución:**

```bash
sudo systemctl start docker
sudo systemctl status docker
```

### Error: "permission denied while trying to connect"

**Causa:** Tu usuario no pertenece al grupo `docker`.

**Solución:**

```bash
sudo usermod -aG docker $USER
newgrp docker
# O cerrar sesión y volver a entrar
```

### Error: GPG key verification failed

**Causa:** Clave GPG no importada.

**Solución:**

```bash
sudo rpm --import https://download.docker.com/linux/centos/gpg
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Docker consume mucho espacio en disco

**Causa:** Imágenes, contenedores y volúmenes sin usar.

**Solución:**

```bash
# Ver uso de disco
docker system df

# Limpiar recursos sin usar
docker system prune -a

# Limpiar todo (incluyendo volúmenes)
docker system prune -a --volumes
```

### Los logs de contenedores llenan el disco

**Causa:** No se configuró la rotación de logs (ver sección 5).

**Solución:** Aplicar la configuración de `daemon.json` de la sección 5.

---

## 11. Desinstalación (Opcional)

Si necesitas desinstalar Docker completamente:

### Desinstalar paquetes

```bash
sudo dnf remove docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```

### Eliminar datos

```bash
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
sudo rm -rf /etc/docker
```

---

## Recursos Adicionales

- [Documentación oficial de Docker Engine - RHEL](https://docs.docker.com/engine/install/rhel/)
- [Post-installation steps for Linux](https://docs.docker.com/engine/install/linux-postinstall/)
- [Docker security best practices](https://docs.docker.com/engine/security/)
- [Docker logging configuration](https://docs.docker.com/config/containers/logging/configure/)

---

## Diferencias entre Ubuntu y RHEL

| Aspecto | Ubuntu | RHEL |
|---------|--------|------|
| Gestor de paquetes | `apt` | `dnf` / `yum` |
| Seguridad | AppArmor | SELinux |
| Firewall | ufw | firewalld |
| Repositorio Docker | docker.com/linux/ubuntu | docker.com/linux/rhel |
| Systemd | Sí | Sí |
| Paquete conflictivo | docker.io | Podman (preinstalado) |

---

**Autor:** Alejandro Fiengo
**Curso:** Docker & Kubernetes - i-Quattro
**Basado en:** Documentación oficial de Docker Engine

---

¡Instalación completa! Ahora puedes empezar a trabajar con contenedores Docker.
