## Fase 7 · Apartado 9 — ❓ Preguntas críticas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Seguridad Avanzada (Permisos NTFS y Access-Based Enumeration)**
> 🧭 Índice de la fase: [[Fase_7]]
>
> **📍 Cuándo se lee:** **Después de la instantánea.** Trabajo de mesa, en tu entrada.

---

> [!help] Preguntas Críticas (Autoevaluación)
> 1. ¿Qué diferencia fundamental hay entre un permiso de carpeta heredado por defecto y un permiso NTFS explícito con `icacls /grant`?
> 2. ¿Cómo mejora Access-Based Enumeration la privacidad de los datos en una empresa con muchos departamentos?
> 3. ¿Qué significa exactamente `(OI)(CI)` en el comando `icacls`?
> 4. 🔬 **Reto práctico:** Crea un archivo dentro de `Prueba3` desde el servidor: `New-Item -Path "C:\ShareData\Prueba3\heredado.txt" -ItemType File`. Luego ejecuta `icacls "C:\ShareData\Prueba3\heredado.txt"`. ¿Qué permisos tiene el archivo nuevo? ¿De dónde vienen esos permisos si no los has asignado explícitamente al archivo? ¿Qué habría pasado si en el Paso 1 hubieras usado `/grant` sin `(OI)(CI)`?
> 5. 🔬 **Reto práctico:** En un cliente Windows unido al dominio (o desde la propia consola del `WindowsServer` con una segunda cuenta), inicia sesión como `user2` (Bomberos) y navega a `\\WindowsServer.BOOCHAN.SPACE\` desde el Explorador de Archivos. ¿Ves la carpeta `Prueba3`? Cierra sesión e inicia como `user1` (Policia) y repite. ¿Qué diferencia hay? Haz una captura de pantalla de ambas vistas — eso es ABE trabajando en producción.

---

> [!danger] ⚠️ Las respuestas van en la ENTRADA, no en un documento aparte
> Estas preguntas demuestran que has **entendido** lo que has hecho, y no solo que has sabido copiar comandos. Se contestan **con tus palabras**. Una fase con el procedimiento perfecto y las preguntas en blanco está **incompleta**.

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_7.8_Punto_de_Control]] | [[Fase_7]] | [[Fase_7.10_Auditoria_y_Cierre]] |
