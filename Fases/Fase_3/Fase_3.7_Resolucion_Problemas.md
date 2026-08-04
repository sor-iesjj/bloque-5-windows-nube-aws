## Fase 3 · Apartado 7 — 🚩 Resolución de problemas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Conectividad VPN (WireGuard para Windows)**
> 🧭 Índice de la fase: [[Fase_3]]
>
> **📍 Cuándo se lee:** **Cuando algo no salga.** Búscate por el síntoma.

---

> [!bug] Troubleshooting (¿No hay conexión?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | `wireguard.exe /installtunnelservice` falla con "Address already in use". | Ya hay otra interfaz VPN activa con esa IP. | Desinstala el servicio con `wireguard.exe /uninstalltunnelservice wg0` antes de volver a instalarlo. |
> | No hay ping entre `10.0.0.1` y `10.0.0.2`. | El puerto 51820 UDP está cerrado en el Security Group de AWS. | Abre el puerto 51820 **UDP** (no TCP) en el Security Group de AWS. |
> | WireGuard no conecta pero el puerto está abierto en AWS. | Las llaves públicas están intercambiadas incorrectamente, o el Firewall de Windows Defender bloquea el puerto 51820/UDP. | Verifica las llaves públicas cruzadas. Comprueba también `Get-NetFirewallRule -DisplayName "WireGuard VPN"` en el servidor. |
> | El cliente no encuentra el `Endpoint`. | Escribiste mal la IP Elástica de AWS, o la instancia no está encendida. | Comprueba la IP Elástica en la consola de EC2 y confirma que el servicio `WireGuardTunnel$wg0` está activo (`Get-Service`). |

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_3.6_Procedimiento]] | [[Fase_3]] | [[Fase_3.8_Punto_de_Control]] |
