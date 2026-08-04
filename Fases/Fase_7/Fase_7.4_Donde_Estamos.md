## Fase 7 · Apartado 4 — 🎯 ¿Dónde estamos?

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Seguridad Avanzada (Permisos NTFS y Access-Based Enumeration)**
> 🧭 Índice de la fase: [[Fase_7]]
>
> **📍 Cuándo se lee:** **Antes de empezar.** De dónde vienes y a dónde llegas.

---

> [!info] Vienes de Fase 6
> Tienes dos carpetas con cuotas físicas de 5 GB gestionadas por FSRM en `C:\ShareData\Prueba1` y `C:\ShareData\Prueba3`. Los usuarios pueden crear archivos dentro de ellas, pero cualquiera que llegue a la carpeta puede verla, aunque no tenga permiso real para entrar. No hay ningún control granular por grupo todavía.

> [!warning] El Problema
> Con solo los permisos por defecto de una carpeta nueva (heredados del disco), no puedes crear un modelo seguro para múltiples departamentos. Si necesitas que el grupo `Policia` tenga acceso total a `Prueba3` pero `Bomberos` no vea ni que existe, no basta con compartir la carpeta por SMB "a secas". Necesitas dos capas: (1) una física (permisos NTFS) que controle quién realmente accede, y (2) una visual (Access-Based Enumeration) que oculte la carpeta a quien no tiene permiso.

> [!success] Objetivo de esta Fase
> Implementar **dos capas de seguridad profesional**: los **permisos NTFS** (listas de control de acceso granulares, nativas del sistema de archivos) que otorgan permisos reales a grupos específicos, y **Access-Based Enumeration (ABE)** —nativa del recurso compartido SMB de Windows— que oculta visualmente las carpetas a las que el usuario no tiene acceso. El resultado: `user2` (Bomberos) simplemente no ve la carpeta `Prueba3` en el Explorador de Archivos.

> [!tip] Hoja de Ruta
> 1. Romper la herencia de permisos en `C:\ShareData\Prueba3` y dejar acceso exclusivo al grupo `Policia`
> 2. Conceder al grupo `Policia` el permiso NTFS `Modify` (Modificar) con herencia hacia subcarpetas y archivos futuros
> 3. Crear los recursos compartidos SMB `Prueba1` (sin ABE) y `Prueba3` (con ABE activado) con `New-SmbShare`
> 4. Verificar que `FolderEnumerationMode` está en `AccessBased` para `Prueba3`
> 5. Desde un cliente Windows: iniciar sesión como `user1` (Policia) y verificar que ve `Prueba3`
> 6. Desde un cliente Windows: iniciar sesión como `user2` (Bomberos) y verificar que NO ve `Prueba3`
>
> **Resultado Final:** Carpeta `Prueba3` completamente protegida — invisible para quienes no tienen permiso, accesible solo para el grupo `Policia`. Los archivos nuevos heredan automáticamente los permisos del grupo.
> **Siguiente:** Fase 8 (Integración del Cliente) — unirás un equipo Windows 11 al dominio y probarás el acceso desde el aula.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_7.3_Obligaciones_Grabacion]] | [[Fase_7]] | [[Fase_7.5_Fundamento_Teorico]] |
