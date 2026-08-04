## Fase 7 · Apartado 10 — 🏁 Auditoría y cierre

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Seguridad Avanzada (Permisos NTFS y Access-Based Enumeration)**
> 🧭 Índice de la fase: [[Fase_7]]
>
> **📍 Cuándo se lee:** **Lo último.** No pases a la fase siguiente sin repasarlo.

---

> [!caution] 🛑 Auditoría de Seguridad (RA.04)
> **Peligro:** Si no usas herencia `(OI)(CI)` al conceder el permiso NTFS, los archivos que cree un usuario del grupo `Policia` no podrán ser editados por sus compañeros del mismo grupo, generando tickets de soporte constantes — exactamente el mismo riesgo que en BoochanV3 al olvidar el flag `-d` de `setfacl`.

> [!success] 🏁 Punto de Control (Antes de seguir)
> - [ ] Comprueba con `icacls "C:\ShareData\Prueba3"` que el permiso del grupo `Policia` está aplicado con herencia `(OI)(CI)`.
> - [ ] ¿El recurso `Prueba3` muestra `FolderEnumerationMode : AccessBased` en `Get-SmbShare`?
> - [ ] En la Fase 8, cuando te conectes desde Windows, verifica que `user2` (Bomberos) no ve la carpeta `Prueba3` pero `user1` (Policia) sí.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_7.9_Preguntas]] | [[Fase_7]] | **Fase 8** |
