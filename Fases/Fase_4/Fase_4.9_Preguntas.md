## Fase 4 · Apartado 9 — ❓ Preguntas críticas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Aprovisionamiento del Dominio (AD DS nativo)**
> 🧭 Índice de la fase: [[Fase_4]]
>
> **📍 Cuándo se lee:** **Después de la instantánea.** Trabajo de mesa, en tu entrada.

---

> [!help] Preguntas Críticas (Autoevaluación)
> 1. ¿Por qué es fundamental que el servidor DNS del dominio sea el propio servidor?
> 2. ¿Qué es un "ticket" de Kerberos y por qué evita enviar contraseñas por la red constantemente?
> 3. Compara el proceso de aprovisionamiento de BoochanV3 (script `provision_boochan.sh`) con el de BoochanV3.1 (`Install-ADDSForest`). ¿Qué ventajas y qué desventajas tiene cada enfoque?
> 4. ¿Cuál es la diferencia entre el Realm (`BOOCHAN.SPACE`) y el nombre NetBIOS (`BOOCHAN`) del dominio? ¿Cuándo se usa cada uno?
> 5. 🔬 **Reto práctico:** Ejecuta `Resolve-DnsName _kerberos._tcp.BOOCHAN.SPACE` en el servidor. Si el dominio está bien provisionado, ¿qué debería devolver? Si falla, ¿qué componente del sistema está fallando?
> 6. 🔬 **Reto práctico:** Ejecuta `Get-ADUser -Filter *` en el servidor. ¿Qué usuarios ves, siendo que tú no has creado ninguno todavía? Localiza el usuario que empieza por `krbtgt` — busca en internet para qué sirve ese usuario en Kerberos y explícalo con tus palabras. Compara además la RAM libre actual (`Get-Counter '\Memory\Available MBytes'`) con la que anotaste al final de la Fase 2.

---

> [!danger] ⚠️ Las respuestas van en la ENTRADA, no en un documento aparte
> Estas preguntas demuestran que has **entendido** lo que has hecho, y no solo que has sabido copiar comandos. Se contestan **con tus palabras**. Una fase con el procedimiento perfecto y las preguntas en blanco está **incompleta**.

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_4.8_Punto_de_Control]] | [[Fase_4]] | [[Fase_4.10_Auditoria_y_Cierre]] |
