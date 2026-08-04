## Fase 1 · Apartado 4 — 🎯 ¿Dónde estamos?

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (AWS EC2 — Windows Server 2025)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Antes de empezar.** De dónde vienes y a dónde llegas.

---

> [!info] El Punto de Partida
> No vienes de una fase anterior — esta es la base. Necesitas un servidor que esté **disponible 24/7**, que no dependa de tu ordenador personal, que sea escalable, profesional y seguro. En BoochanV3 (Ubuntu + Samba) ese servidor vivía en AWS con Linux; en **BoochanV3.1** vamos a construir el mismo tipo de servidor cloud, en la misma nube de Amazon, pero con **Windows Server 2025** y su rol nativo de directorio (AD DS), sin Samba de por medio.

> [!warning] El Problema
> Instalar un servidor físico en la clase es caro, requiere mantenimiento constante y no es escalable. Además, Windows Server con interfaz gráfica y Active Directory consume bastantes más recursos que un Ubuntu Server headless: no vale con "la instancia más barata que encuentres", hay que dimensionarla con cabeza — y en AWS Academy eso también significa vigilar el consumo de crédito. La nube resuelve la disponibilidad; nosotros resolvemos el dimensionado.

> [!success] Objetivo de esta Fase
> Crear una **instancia EC2 en Amazon Web Services** llamada `WindowsServer`, con la AMI **Windows Server 2025 Base**, dimensionada en `t3.large` (2 vCPU, 8 GB RAM). Este servidor será, en las próximas fases, tu controlador de dominio Active Directory nativo. Lo protegerás con un **Security Group (firewall cloud)** que bloquea internet y abre solo el puerto imprescindible por ahora: RDP.

> [!tip] Hoja de Ruta
> 1. Crear el Key Pair (`boochan-key`) — aquí no sirve para SSH, sirve para descifrar la contraseña de Windows
> 2. Crear el Security Group con la regla RDP (3389)
> 3. Lanzar la instancia EC2 `WindowsServer` (Windows Server 2025 Base, `t3.large`)
> 4. Asignar una Elastic IP para tener una IP pública fija
> 5. Obtener la contraseña de Administrador con "Get Windows Password" / "Obtener contraseña de Windows"
> 6. Conectarte por **RDP** desde tu PC con el usuario `Administrator`
> 7. Verificar acceso a internet y medir la RAM base con el `Administrador de tareas`
> 8. Anotar el dominio de todo el proyecto: `BOOCHAN.SPACE`
>
> **Resultado Final:** Un servidor Windows Server 2025 en AWS, listo, accesible por RDP, y protegido perimetralmente.
> **Siguiente:** Fase 2 (Purga y Preparación del Entorno) — limpiaremos el servidor de roles y configuración innecesaria y prepararemos el terreno para el rol AD DS.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.3_Obligaciones_Grabacion]] | [[Fase_1]] | [[Fase_1.5_Fundamento_Teorico]] |
