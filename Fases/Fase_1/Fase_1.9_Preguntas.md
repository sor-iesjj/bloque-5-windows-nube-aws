## Fase 1 · Apartado 9 — ❓ Preguntas críticas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (AWS EC2 — Windows Server 2025)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Después de la instantánea.** Trabajo de mesa, en tu entrada.

---

> [!help] Preguntas Críticas (Autoevaluación del alumno)
> 1. ¿Por qué Amazon AWS es responsable del hardware pero tú eres el responsable del Sistema Operativo?
> 2. ¿Qué ocurre exactamente si dejas el puerto de administración RDP (3389) abierto a **"Cualquiera" (0.0.0.0/0)** permanentemente?
> 3. Compara los tres mecanismos de acceso inicial que has visto en el módulo: SSH directo con `.pem` (BoochanV3), usuario+contraseña elegidos (BoochanV2.1) y `.pem` para descifrar una contraseña generada por AWS (BoochanV3.1). ¿Qué ventaja de seguridad tiene este último frente a los otros dos?
> 4. ¿Por qué RDP transmite el escritorio completo mientras que SSH (usado en BoochanV3) solo transmite texto? ¿Qué ventajas e inconvenientes tiene cada enfoque?
> 5. 🔬 **Reto práctico:** Entra en el Security Group de AWS y **elimina** temporalmente la regla del puerto 3389 (y vuelve a añadirla). Intenta conectarte por RDP mientras la regla no existe. ¿Qué ocurre? ¿Qué has comprobado con este experimento?
> 6. 🔬 **Reto práctico:** Anota en el `Administrador de tareas` cuánta RAM usa el sistema base recién instalado. Guarda ese dato — lo compararás en la Fase 4, cuando el rol AD DS esté corriendo.

---

> [!danger] ⚠️ Las respuestas van en la ENTRADA, no en un documento aparte
> Estas preguntas demuestran que has **entendido** lo que has hecho, y no solo que has sabido copiar comandos. Se contestan **con tus palabras**. Una fase con el procedimiento perfecto y las preguntas en blanco está **incompleta**.

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.8_Punto_de_Control]] | [[Fase_1]] | [[Fase_1.10_Auditoria_y_Cierre]] |
