# Implementación de Laboratorios Avanzados con GNS3 + Hipervisores en Windows 11

![Estado](https://img.shields.io/badge/Estado-Investigación_en_Curso-yellow)
![Plataforma](https://img.shields.io/badge/Plataforma-GNS3%20%2B%20Windows11-blue)

---

## 1. Arquitectura de Virtualización en Windows 11
<img width="602" height="408" alt="Captura de pantalla 2026-04-08 154724" src="https://github.com/user-attachments/assets/dd4c3760-c981-455b-b85f-f2f0620a98bf" />

### 1.1 Aislamiento de Núcleo y VBS

Windows 11 incorpora **Virtualization Based Security (VBS)**, que utiliza Hyper-V de forma subyacente para proteger el sistema operativo.

Esto puede generar conflictos con otros hipervisores como:

- VirtualBox
- VMware
- GNS3 VM

Cuando el **Aislamiento de Núcleo** está activado:

- Hyper-V toma control de las extensiones VT-x/AMD-V
- Otros hipervisores no pueden acceder directamente al hardware
- KVM puede aparecer como `False`

> Impacto técnico: Reduce el rendimiento o impide virtualización anidada.

---

### 1.2 Activación de VT-x / AMD-V

La virtualización debe activarse desde BIOS/UEFI.

#### Verificación en Windows:

```powershell
systeminfo
```

Debe aparecer:

```
Virtualization Enabled In Firmware: Yes
```

---

## 2. GNS3 VM: El Motor de Simulación

### 2.1 ¿Qué es KVM?

KVM (Kernel-based Virtual Machine) es un módulo del kernel de Linux que permite virtualización de alto rendimiento.

En GNS3:

- Si KVM = True → Rendimiento óptimo
- Si KVM = False → Emulación lenta

KVM es crítico porque permite ejecutar routers y appliances con aceleración por hardware.

---

### 2.2 Configuración de Recursos

Para evitar inestabilidad en Windows 11:

Recomendación técnica:

| Recurso | Recomendación |
|----------|--------------|
| CPU | 50% del total disponible |
| RAM | Máximo 60% del total |
| Disco | SSD recomendado |

Ejemplo:
Si tu PC tiene 16GB RAM → asignar 8GB a GNS3 VM.

---

## 3. Integración con VirtualBox (Local)

### 3.1 Configuración Host-Only

Pasos:

1. Ir a VirtualBox → Herramientas → Host Network Manager
2. Crear Adaptador Host-Only
3. Asignar IP estática (ej: 192.168.56.1)

Esto permite comunicación entre:

- GUI GNS3 (Windows)
- GNS3 VM

---

### 3.2 Modo Promiscuo

El modo promiscuo permite que la tarjeta virtual:

- Capture todo el tráfico
- No solo paquetes dirigidos a su MAC

Es necesario para:

- Protocolos de Capa 2
- VLANs
- STP
- ARP

Configuración:

VirtualBox → Red → Modo Promiscuo → **Permitir todo**

---

## 4. Integración con VMware ESXi (Remoto)

### 4.1 Arquitectura Cliente-Servidor

Arquitectura:

Laptop (GUI GNS3)
↓
Red Física
↓
Servidor ESXi
↓
GNS3 VM

El GUI solo controla.
El procesamiento ocurre en ESXi.

Ventaja:
- Mejor rendimiento
- Separación de recursos

---

### 4.2 Seguridad en vSwitch

En el Port Group del vSwitch se deben modificar:

- Promiscuous Mode → Accept
- MAC Address Changes → Accept
- Forged Transmits → Accept

¿Por qué?

Porque GNS3 crea dispositivos virtuales que cambian MAC dinámicamente.

Si no se permite:
- El tráfico se bloquea
- No hay conectividad entre nodos

---

## 5. Matriz de Solución de Errores (Troubleshooting)

| Error Detectado | Causa Técnica | Solución |
|-----------------|--------------|----------|
| KVM support available: False | Virtualización anidada deshabilitada | Habilitar nested virtualization |
| Sin conectividad entre VMs | Modo promiscuo desactivado | Permitir todo en adaptador |
| Error puerto 3080 | Firewall bloqueando API | Crear regla de entrada TCP |

---

### Ejemplo de comando en VirtualBox

```bash
VBoxManage modifyvm "GNS3 VM" --nested-hw-virt on
```

---

## Conclusión

La integración de GNS3 con hipervisores Tipo 1 y Tipo 2 permite construir laboratorios de nivel profesional. 

El rendimiento óptimo depende de:

- Correcta activación de VT-x/AMD-V
- Desactivación de VBS si genera conflicto
- Configuración adecuada del modo promiscuo
- Ajustes de seguridad en ESXi

Una mala configuración puede degradar significativamente el desempeño del laboratorio.
