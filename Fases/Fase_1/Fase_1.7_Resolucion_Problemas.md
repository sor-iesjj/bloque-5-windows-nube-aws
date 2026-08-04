## Fase 1 · Apartado 7 — 🚩 Resolución de problemas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (AWS EC2 — Windows Server 2025)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Cuando algo no salga.** Búscate por el síntoma.

---

> [!bug] Tabla de Troubleshooting (¿Algo no funciona?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | El botón "Obtener contraseña de Windows" está desactivado o da error. | La instancia todavía no ha terminado de generar/cifrar la contraseña interna. | Espera 3-4 minutos tras ver el estado "En ejecución" y vuelve a intentarlo. |
> | No puedo conectar por RDP ("No se puede establecer la conexión"). | La instancia no ha terminado de arrancar/aprovisionarse. | Espera unos minutos (Windows Server tarda más que Ubuntu) y vuelve a intentarlo. |
> | RDP se conecta pero rechaza las credenciales. | Usuario o contraseña incorrectos, o Bloq Mayús activado. | Comprueba que escribes exactamente `Administrator` y la contraseña descifrada en el Paso 5, respetando mayúsculas/símbolos. |
> | Aparece "Este equipo no puede conectarse al equipo remoto". | El puerto 3389 no está abierto en el Security Group, o la regla se borró por error. | Revisa **Security Group `sg-boochan-[tunombre]`** y confirma que la regla RDP (3389) existe con origen `0.0.0.0/0`. |
> | El servidor **no responde al ping**. | El protocolo ICMP está bloqueado por defecto en el Security Group. | Es normal por seguridad. No abras el ping; usa RDP para verificar conectividad. |
> | "Host key verification failed" o la IP ha cambiado. | No se ha asignado o asociado correctamente la Elastic IP. | Revisa el Paso 4: la Elastic IP debe estar **asociada** a la instancia `WindowsServer`, no solo asignada. |
> | La pantalla de RDP se ve muy lenta o pixelada. | Configuración de calidad de la conexión demasiado alta para el ancho de banda del aula. | En el cliente RDP, antes de conectar, baja la calidad de color/experiencia en las opciones avanzadas de la conexión. |

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.6_Procedimiento]] | [[Fase_1]] | [[Fase_1.8_Punto_de_Control]] |
