# Instalación de WSL2 + AlmaLinux/Ubuntu en Windows

Guía para configurar el entorno de desarrollo en Windows usando WSL2 (Windows Subsystem for Linux) con AlmaLinux 10 o Ubuntu 24.04 LTS.

**Configuración recomendada para el curso:** WSL2 + AlmaLinux 10 (sin GUI)

---

## Requisitos previos

- Windows 10 versión 2004 o superior (Build 19041 o superior) o Windows 11
- Permisos de administrador en tu equipo

---

## Opción A: Instalar AlmaLinux 10 en WSL2 (Recomendado)

### Paso 1: Habilitar WSL

Abre **PowerShell** o **Windows Terminal** como **Administrador** y ejecuta:

```powershell
wsl --install --no-distribution
```

Este comando habilita WSL2 sin instalar una distribución por defecto.

### Paso 2: Reiniciar el equipo

Después de la instalación, reinicia tu computadora para aplicar los cambios.

### Paso 3: Descargar e importar AlmaLinux 10

**Recomendación para el curso:** Usar **AlmaLinux 10** (versión estable y recomendada).

1. Descarga la imagen de AlmaLinux 10 para WSL desde:
   - Repositorio oficial: https://github.com/AlmaLinux/wsl-images/releases
   - Busca la última versión de AlmaLinux 10

2. Opción más sencilla - importar directamente:

```powershell
# Crear directorio para AlmaLinux
mkdir C:\WSL\AlmaLinux

# Descargar e importar imagen base de AlmaLinux 10
# Nota: Verifica la URL actual en https://github.com/AlmaLinux/wsl-images/releases
wsl --import AlmaLinux10 C:\WSL\AlmaLinux https://github.com/AlmaLinux/wsl-images/releases/download/10.0/AlmaLinux-10-x86_64.tar.gz
```

**Alternativa manual:**

```powershell
# Descargar el archivo tar.gz manualmente desde:
# https://github.com/AlmaLinux/wsl-images/releases

# Luego importar:
wsl --import AlmaLinux10 C:\WSL\AlmaLinux C:\Users\TU_USUARIO\Downloads\AlmaLinux-10-x86_64.tar.gz
```

### Paso 4: Configurar usuario en AlmaLinux

```powershell
# Iniciar AlmaLinux
wsl -d AlmaLinux10

# Dentro de AlmaLinux, crear usuario
useradd -m -G wheel tu_usuario
passwd tu_usuario

# Configurar como usuario predeterminado
exit

# En PowerShell
cd C:\WSL\AlmaLinux
.\AlmaLinux10.exe config --default-user tu_usuario
```

### Paso 5: Configurar AlmaLinux como distribución predeterminada

```powershell
wsl --set-default AlmaLinux10
```

---

## Opción B: Instalar Ubuntu 24.04 LTS en WSL2

### Paso 1: Instalar Ubuntu

```powershell
wsl --install -d Ubuntu-24.04
```

### Paso 2: Reiniciar el equipo

Después de la instalación, reinicia tu computadora.

### Paso 3: Configurar Ubuntu

Al iniciar Ubuntu por primera vez, se te pedirá crear un usuario y contraseña:

```bash
Enter new UNIX username: tu_usuario
New password: ********
Retype new password: ********
```

> **Nota:** La contraseña no se mostrará mientras la escribes.

---

## Paso 4: Actualizar el sistema

**Para AlmaLinux:**

```bash
sudo dnf update -y
sudo dnf upgrade -y
```

**Para Ubuntu:**

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Paso 5: Verificar la instalación

Verifica que estás usando WSL2:

```powershell
wsl --list --verbose
```

Deberías ver algo como:

```
  NAME            STATE           VERSION
* AlmaLinux9      Running         2
```

o

```
  NAME            STATE           VERSION
* Ubuntu-24.04    Running         2
```

Si aparece VERSION 1, actualiza a WSL2:

```powershell
# Para AlmaLinux
wsl --set-version AlmaLinux9 2

# Para Ubuntu
wsl --set-version Ubuntu-24.04 2
```

---

## Configuración adicional recomendada

### Establecer Ubuntu como distribución predeterminada

```powershell
wsl --set-default Ubuntu-24.04
```

### Integración con Windows Terminal

Windows Terminal detecta automáticamente WSL. Puedes configurarlo como perfil predeterminado en:

`Configuración → Perfil predeterminado → Ubuntu-24.04`

### Acceso a archivos de Windows desde WSL

Los archivos de Windows están disponibles en:

```bash
cd /mnt/c/Users/tu_usuario/
```

### Acceso a archivos de WSL desde Windows

Abre el Explorador de Windows y escribe en la barra de direcciones:

```
\\wsl$\Ubuntu-24.04\home\tu_usuario
```

---

## Solución de problemas comunes

### Error: "WSL 2 requires an update to its kernel component"

Descarga e instala el paquete de actualización del kernel:
https://aka.ms/wsl2kernel

### Ver logs de WSL

```powershell
wsl --status
```

### Reiniciar WSL

```powershell
wsl --shutdown
```

---

## Recursos adicionales

- [Documentación oficial de WSL](https://docs.microsoft.com/es-es/windows/wsl/)
- [Guía de instalación de Docker en WSL2](https://docs.docker.com/desktop/wsl/)

---

[← Volver al README principal](README.md)
