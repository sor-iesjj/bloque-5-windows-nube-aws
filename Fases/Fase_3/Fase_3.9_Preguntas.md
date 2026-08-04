## Fase 3 · Apartado 9 — ❓ Preguntas críticas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Conectividad VPN (WireGuard para Windows)**
> 🧭 Índice de la fase: [[Fase_3]]
>
> **📍 Cuándo se lee:** **Después de la instantánea.** Trabajo de mesa, en tu entrada.

---

> [!help] Preguntas Críticas (Autoevaluación)
> 1. ¿Por qué la llave privada **NUNCA** debe salir de tu servidor ni enviarse por correo?
> 2. ¿Qué diferencia hay entre instalar WireGuard "solo activado en la app" y hacerlo como Tunnel Service con `/installtunnelservice`?
> 3. ¿Para qué sirve el parámetro `AllowedIPs` en la configuración del Peer?
> 4. 🔬 **Reto práctico:** Con el túnel activo, ejecuta `wg show` en el servidor y localiza la línea `latest handshake`. ¿Hace cuántos segundos fue el último intercambio? Ahora desactiva el túnel desde tu PC del aula y vuelve a ejecutar el comando 30 segundos después. ¿Qué cambió en esa línea? ¿Qué te dice eso sobre el estado de la conexión?
> 5. 🔬 **Reto práctico:** Con el túnel WireGuard **desactivado** en tu PC, intenta conectarte al servidor por RDP usando la IP pública/Elástica de AWS (no la `10.0.0.1`). ¿Puedes entrar? ¿Por qué sí o por qué no? Razona tu respuesta mirando la regla RDP del Security Group tras el cambio de esta fase — ¿sigue existiendo la regla que lo permitía desde `0.0.0.0/0`?

---

> [!danger] ⚠️ Las respuestas van en la ENTRADA, no en un documento aparte
> Estas preguntas demuestran que has **entendido** lo que has hecho, y no solo que has sabido copiar comandos. Se contestan **con tus palabras**. Una fase con el procedimiento perfecto y las preguntas en blanco está **incompleta**.

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_3.8_Punto_de_Control]] | [[Fase_3]] | [[Fase_3.10_Auditoria_y_Cierre]] |
