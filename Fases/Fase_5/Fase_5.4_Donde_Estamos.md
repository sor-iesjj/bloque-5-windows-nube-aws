## Fase 5 · Apartado 4 — 🎯 ¿Dónde estamos?

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Gestión de Identidades (Usuarios y Grupos en Active Directory)**
> 🧭 Índice de la fase: [[Fase_5]]
>
> **📍 Cuándo se lee:** **Antes de empezar.** De dónde vienes y a dónde llegas.

---

> [!info] Vienes de Fase 4
> Tienes un dominio Active Directory completamente provisionado sobre la instancia EC2 `WindowsServer` (AWS, `t3.large`) con el rol AD DS instalado. El dominio `BOOCHAN.SPACE` (NetBIOS `BOOCHAN`) existe, DNS es propio del controlador de dominio, y el Security Group de AWS permite el tráfico de Active Directory desde la Fase 4. Sin embargo, dentro de ese dominio todavía no existe ni un solo usuario ni grupo de "negocio" — solo las cuentas administrativas que Windows crea por defecto.

> [!warning] El Problema
> Un dominio sin usuarios ni grupos organizados es papel mojado. Si no estructuras a los empleados en grupos con un criterio claro (departamento, función, nivel de acceso), cada vez que necesites dar permisos a una carpeta tendrás que hacerlo usuario por usuario — un despropósito que no escala y que es la primera causa de "fugas de permisos" en auditorías de seguridad reales.

> [!success] Objetivo de esta Fase
> Crear una estructura organizativa dentro de Active Directory: **Unidades Organizativas (OU)** que reflejen los departamentos de la empresa ficticia, **grupos de seguridad** que agrupen a los empleados por función, y **usuarios** asignados a esos grupos. Todo con las herramientas nativas de AD DS — sin necesidad de traducir nada a otro sistema operativo.

> [!tip] Hoja de Ruta
> 1. Crear la Unidad Organizativa raíz `Departamentos` y dos sub-OUs: `Policia` y `Bomberos`
> 2. Crear dos grupos de seguridad de ámbito global: `Policia` y `Bomberos` — para demostrar después segregación de datos
> 3. Crear dos usuarios: `user1` (miembro de `Policia`) y `user2` (miembro de `Bomberos`)
> 4. Verificar con `Get-ADUser` y `Get-ADGroupMember` que la pertenencia a grupo es correcta
> 5. Comprobar el inicio de sesión de ambos usuarios (o al menos la validación de credenciales) contra el dominio
> 6. Reflexionar sobre la simplificación respecto a un entorno Samba: en AD DS nativo no existe ningún paso de "traducción" de identidades
>
> **Resultado Final:** El dominio tiene una estructura de OUs, grupos y usuarios lista para que la Fase 6 (cuotas con FSRM) y la Fase 7 (permisos NTFS + ABE) construyan sobre ella.
> **Siguiente:** Fase 6 (Almacenamiento con FSRM) — aplicarás cuotas de disco reales sobre carpetas NTFS para controlar que no llenen el servidor.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_5.3_Obligaciones_Grabacion]] | [[Fase_5]] | [[Fase_5.5_Fundamento_Teorico]] |
