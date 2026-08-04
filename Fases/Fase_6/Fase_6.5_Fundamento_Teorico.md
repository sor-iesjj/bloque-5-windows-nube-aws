## Fase 6 · Apartado 5 — 📚 Fundamento teórico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Almacenamiento con Cuotas (FSRM)**
> 🧭 Índice de la fase: [[Fase_6]]
>
> **📍 Cuándo se lee:** **Antes de teclear.** Los conceptos que necesitas.

---

> [!abstract] 1. Evitando el Colapso por Llenado
> La gestión de cuotas de disco es vital para evitar la "Denegación de Servicio" por llenado de disco. Si un usuario (o un atacante) llena todo el volumen del sistema, los registros de eventos no podrán escribirse y los servicios del servidor pueden empezar a fallar de forma impredecible.

> [!tip] 2. Cuota Directa sobre NTFS (sin discos virtuales)
> En **BoochanV3** (Ubuntu + Samba sobre AWS), las cuotas se implementaban con **Loop Devices**: archivos `.img` que actuaban como discos duros independientes formateados en ext4, montados como si fueran particiones reales. Era un rodeo necesario porque el ext4 clásico de Linux no siempre ofrece cuotas de carpeta sencillas de administrar.
> En **Windows Server con FSRM**, ese rodeo desaparece: FSRM aplica el límite **directamente sobre la carpeta** del volumen NTFS real (el mismo disco EBS raíz de la instancia), sin necesidad de crear ningún "disco falso dentro del disco" ni de añadir un volumen de datos adicional en AWS. Sigue siendo una barrera física e infranqueable —NTFS rechaza la escritura en cuanto se alcanza el límite—, pero la implementación es mucho más simple: no hay que crear archivos `.img`, no hay que formatearlos, y no hay ninguna entrada equivalente al `fstab` que pueda romper el arranque del servidor si te equivocas.

### 📖 Diccionario de Conceptos Clave

> [!quote] Almacenamiento Avanzado
> - **FSRM (File Server Resource Manager):** El rol de Windows Server que gestiona cuotas, filtrado de archivos e informes de almacenamiento sobre carpetas NTFS.
> - **Plantilla de Cuota (Quota Template):** Una configuración reutilizable (tamaño, tipo de límite, umbrales de aviso) que se aplica a una o varias carpetas.
> - **Cuota Estricta (Hard Quota):** El límite es infranqueable — al alcanzarlo, Windows rechaza nuevas escrituras. Es el equivalente funcional al Loop Device de Samba.
> - **Cuota Flexible (Soft Quota):** Solo genera avisos, no bloquea la escritura. No la usaremos en este proyecto porque necesitamos una barrera real.
> - **Umbral (Threshold):** Un porcentaje de la cuota (por ejemplo, 85%) en el que FSRM puede disparar una notificación o un evento antes de llegar al límite.

---

### 🔓 Apertura de Puertos (Security Group de AWS)

> [!info] ℹ️ Sin cambios en el Security Group en esta fase
> Esta fase trabaja íntegramente en local sobre el servidor — instalación de un rol y gestión de carpetas NTFS en el disco de la propia instancia `WindowsServer`. No requiere añadir ninguna regla nueva en el Security Group de AWS: FSRM no expone ningún puerto de red nuevo salvo que actives las notificaciones por correo (SMTP), algo que no usaremos en este proyecto de laboratorio.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_6.4_Donde_Estamos]] | [[Fase_6]] | [[Fase_6.6_Procedimiento]] |
