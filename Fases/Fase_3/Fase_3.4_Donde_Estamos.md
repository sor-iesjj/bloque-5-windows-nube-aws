## Fase 3 · Apartado 4 — 🎯 ¿Dónde estamos?

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Conectividad VPN (WireGuard para Windows)**
> 🧭 Índice de la fase: [[Fase_3]]
>
> **📍 Cuándo se lee:** **Antes de empezar.** De dónde vienes y a dónde llegas.

---

> [!info] Vienes de Fase 2
> Completaste la preparación inicial del servidor y le diste identidad (`WindowsServer`), con la IP privada de la VPC de AWS confirmada (rango `172.31.x.x`). Ahora tienes un servidor limpio, identificado, actualizado. Pero hay un problema crítico: está expuesto a internet público. El puerto RDP (3389) está abierto a todo el mundo desde la Fase 1 — bots intentarán conectarse miles de veces al día para adivinar contraseñas de Administrador.

> [!warning] El Problema
> Sin una VPN privada, tu servidor es vulnerable a ataques de fuerza bruta contra el Escritorio Remoto. Cualquiera en internet puede intentar adivinar credenciales de Administrador. Además, en las próximas fases necesitarás que el aula acceda al servidor desde cualquier lugar, pero solo el aula — no todo el mundo. Necesitas un túnel privado cifrado que solo tú controles.

> [!success] Objetivo de esta Fase
> Instalar **WireGuard para Windows**: una VPN ligera y moderna que crea un túnel P2P cifrado entre tu PC del aula (`10.0.0.2`) y el servidor (`10.0.0.1`). Este túnel es tu "puerta trasera" secreta — solo quien tenga las llaves criptográficas puede entrar. Cerrarás el puerto RDP directo a internet y aceptarás conexiones solo desde dentro de la VPN, tal y como quedó anunciado como tarea pendiente al final de la Fase 1.

> [!tip] Hoja de Ruta
> 1. Añadir regla entrante UDP 51820 en el Security Group de AWS (WireGuard escucha aquí)
> 2. Instalar WireGuard en el servidor
> 3. Generar pares de llaves criptográficas (servidor + tu PC)
> 4. Crear archivo de configuración `wg0.conf` en el servidor
> 5. Crear perfil VPN para tu PC del aula
> 6. Activar el túnel y verificar con `ping 10.0.0.1` desde tu PC
> 7. Cerrar el acceso RDP público en el Security Group (Zero Trust)
>
> **Resultado Final:** Servidor accesible solo a través del túnel VPN. Totalmente blindado contra ataques de internet público.
> **Siguiente:** Fase 4 (Dominio) — provisionar Active Directory Domain Services. Ahora que hay conexión VPN segura, puedes instalar servicios críticos.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_3.3_Obligaciones_Grabacion]] | [[Fase_3]] | [[Fase_3.5_Fundamento_Teorico]] |
