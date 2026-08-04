## Fase 2 · Apartado 4 — 🎯 ¿Dónde estamos?

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Preparación Inicial del Servidor**
> 🧭 Índice de la fase: [[Fase_2]]
>
> **📍 Cuándo se lee:** **Antes de empezar.** De dónde vienes y a dónde llegas.

---

> [!info] Vienes de Fase 1
> Creaste una instancia EC2 `t3.large` con **AMI Windows Server 2025 Base** en AWS. Está encendida, accesible por Escritorio Remoto, protegida por un Security Group que solo abre el puerto 3389. Pero viene "de fábrica": nombre genérico tipo `WIN-XXXXXXXXXXX`, sin actualizaciones aplicadas, y sin que nadie le haya confirmado cuál es su identidad definitiva dentro del proyecto.

> [!warning] El Problema
> A diferencia de Ubuntu Server (donde en BoochanV3 había que **purgar** Samba, CUPS y demonios heredados que ocupaban puertos), Windows Server no trae ningún servicio de directorio ni de archivos preinstalado que estorbe. El "problema" aquí es de otra naturaleza: el servidor tiene un nombre aleatorio que nadie reconocería en un log ni en una consulta DNS, y puede tener vulnerabilidades sin parchear desde el día en que se generó la imagen. Además, en AWS, la IP privada la asigna el DHCP de la VPC por defecto — no hace falta fijarla manualmente porque las instancias EC2 conservan su IP privada mientras no se termine la instancia, pero conviene comprobarla y anotarla antes de construir nada encima.

> [!success] Objetivo de esta Fase
> **Identidad:** Renombrar el servidor a `WindowsServer`, el nombre que usará todo el proyecto BoochanV3.1. **Red:** Verificar la IP privada que la VPC por defecto de AWS asignó al servidor (rango `172.31.x.x`) y comprobar que persiste tras reinicios. **Higiene:** Aplicar Windows Update para partir de un sistema parcheado antes de instalar ningún rol crítico como AD DS (Fase 4).

> [!tip] Hoja de Ruta
> 1. Conectarse al servidor por Escritorio Remoto (RDP) con la IP pública/Elástica de AWS
> 2. Renombrar el equipo con `Rename-Computer` a `WindowsServer`
> 3. Verificar la IP privada asignada por la VPC de AWS con `Get-NetIPConfiguration` (rango `172.31.x.x`)
> 4. Apuntar el DNS del propio adaptador a `127.0.0.1` (se explica por qué, aunque el DNS real no existirá hasta la Fase 4)
> 5. Instalar actualizaciones de Windows Update
> 6. Reiniciar y verificar identidad y red
>
> **Resultado Final:** Servidor con nombre `WindowsServer`, IP privada confirmada en el rango `172.31.x.x` de AWS, y parches de seguridad al día.
> **Siguiente:** Fase 3 (Conectividad VPN) — instalarás WireGuard para Windows para blindar el acceso remoto al servidor.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_2.3_Obligaciones_Grabacion]] | [[Fase_2]] | [[Fase_2.5_Fundamento_Teorico]] |
