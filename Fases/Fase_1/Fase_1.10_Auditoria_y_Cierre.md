## Fase 1 · Apartado 10 — 🏁 Auditoría y cierre

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (AWS EC2 — Windows Server 2025)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Lo último.** No pases a la fase siguiente sin repasarlo.

---

> [!caution] 🛑 Auditoría de Seguridad — Tarea pendiente tras la Fase 3
> Una vez que la VPN esté funcionando, cerrarás el acceso RDP directo desde Internet. **No lo hagas ahora**: sin VPN activa te quedarías sin acceso al servidor.
>
> **Acción — Restringir el puerto 3389 al mundo exterior en el Security Group de AWS:**
> Vuelve a **Security Group `sg-boochan-[tunombre]`** → **Reglas de entrada**. Localiza la regla `RDP` (puerto 3389, origen `0.0.0.0/0`) y cámbiala para que el **Origen** sea únicamente el rango de la red privada de la VPN, o elimínala si vas a acceder siempre a través del túnel WireGuard.
>
> Esto es aplicar seguridad "Zero Trust": nadie en Internet puede llegar al servidor por RDP; solo quien esté dentro de la VPN.

---

### ✅ Checklist Final de la Fase 1

- [ ] Key Pair `boochan-key` creado y fichero `.pem` guardado en lugar seguro.
- [ ] Security Group `sg-boochan-[tunombre]` creado con la regla RDP (3389, origen `0.0.0.0/0`).
- [ ] Instancia EC2 `WindowsServer` lanzada con AMI `Windows Server 2025 Base`.
- [ ] Tipo de instancia: `t3.large` (2 vCPU, 8 GB RAM), disco 50 GB gp3.
- [ ] Elastic IP asignada y **asociada** a la instancia.
- [ ] Contraseña de `Administrator` obtenida con "Get Windows Password" y guardada en lugar seguro.
- [ ] Conexión por RDP realizada con éxito desde tu propio ordenador.
- [ ] `ping google.com` funciona desde dentro del servidor.
- [ ] RAM base anotada desde el `Administrador de tareas` (para comparar en la Fase 4).
- [ ] Dominio del proyecto anotado: NetBIOS `BOOCHAN`, Realm `BOOCHAN.SPACE`.

> **Siguiente paso:** Fase 2 — Purga y Preparación del Entorno, donde revisaremos la configuración inicial de Windows Server (nombre de host, actualizaciones, roles innecesarios) y prepararemos el terreno para el rol AD DS.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.9_Preguntas]] | [[Fase_1]] | **Fase 2** |
