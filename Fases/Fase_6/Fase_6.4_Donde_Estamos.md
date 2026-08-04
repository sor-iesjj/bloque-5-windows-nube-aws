## Fase 6 · Apartado 4 — 🎯 ¿Dónde estamos?

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Almacenamiento con Cuotas (FSRM)**
> 🧭 Índice de la fase: [[Fase_6]]
>
> **📍 Cuándo se lee:** **Antes de empezar.** De dónde vienes y a dónde llegas.

---

> [!info] Vienes de Fase 5
> Tienes usuarios y grupos del dominio completamente estructurados: `user1` en el grupo `Policia`, `user2` en el grupo `Bomberos`, organizados en sus OUs correspondientes dentro de `BOOCHAN.SPACE`. El servidor reconoce a ambos como ciudadanos de pleno derecho de Active Directory. Sin embargo, no existe ningún límite sobre cuánto espacio pueden usar en el disco del servidor.

> [!warning] El Problema
> Sin cuotas de disco, un usuario malintencionado (o simplemente desprevenido) podría llenar el volumen del sistema completamente. Cuando un disco de Windows se llena, el sistema se vuelve inestable: los registros de eventos dejan de escribirse, los servicios pueden fallar al no poder escribir archivos temporales, y la administración remota se complica. En empresas, esto es una "Denegación de Servicio" (DoS) involuntaria pero devastadora — y en una instancia EC2 de AWS, además, un disco lleno puede impedir la propia conexión RDP de auxilio.

> [!success] Objetivo de esta Fase
> Instalar el rol **FSRM (File Server Resource Manager)** y aplicar **cuotas físicas** de 5 GB sobre dos carpetas del proyecto (`Prueba1` y `Prueba3`), directamente sobre el sistema de archivos NTFS real del disco del sistema operativo — sin crear ningún disco virtual adicional ni depender de un volumen EBS de datos separado en AWS. Si una carpeta se llena, solo ese departamento se ve afectado; el resto del servidor sigue funcionando con normalidad.

> [!tip] Hoja de Ruta
> 1. Instalar el rol `FS-Resource-Manager` (File Server Resource Manager)
> 2. Crear las carpetas del proyecto: `C:\ShareData\Prueba1` y `C:\ShareData\Prueba3`
> 3. Crear una **plantilla de cuota** de 5 GB con `New-FsrmQuotaTemplate`
> 4. Aplicar esa plantilla a ambas carpetas con `New-FsrmQuota`
> 5. Verificar con `Get-FsrmQuota` que el límite está activo
> 6. Intentar llenar una carpeta para comprobar que la barrera es infranqueable, y observar cómo se libera el espacio al borrar
>
> **Resultado Final:** Dos carpetas del proyecto, `C:\ShareData\Prueba1` y `C:\ShareData\Prueba3`, cada una con un límite físico de 5 GB gestionado por FSRM. El servidor está protegido contra el llenado descontrolado de disco.
> **Siguiente:** Fase 7 (Seguridad Avanzada) — aplicarás permisos NTFS y Access-Based Enumeration para controlar quién ve y accede a cada carpeta.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_6.3_Obligaciones_Grabacion]] | [[Fase_6]] | [[Fase_6.5_Fundamento_Teorico]] |
