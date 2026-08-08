## Fase 1 · Apartado 3 — 📹 Obligaciones de grabación

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (AWS EC2 — Windows Server 2025)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Antes de arrancar OBS.** Cómo se graba y se entrega.

---

> [!important] 📹 Obligaciones de grabación (LÉEME — es igual en TODAS las fases)
> Esta práctica se **graba entera con OBS**, de principio a fin. No es un repaso al final: quiero ver **cómo lo haces tú**.
> 1. **Prepárate primero (sin grabar):** comprueba lo necesario, **léete el procedimiento entero** y **crea la entrada de apuntes de esta fase** en Obsidian: fichero `v3-1-fase-1-infraestructura-cloud-aws-ec2-windows-se.md` dentro de `00_Apuntes/Trimestre_N/B5_Windows_Nube/`, con la estructura del **Bloque 0 · Fase 0.1.b** y **vacía**. Rellenarla es cosa tuya, después.
> 2. **Arranca OBS y PRESÉNTATE:** *"Hola, me llamo [Nombre], 2.º SMR, y en este vídeo voy a explicar la Fase 1 de Boochan V3.1 — Infraestructura Cloud (AWS EC2 — Windows Server 2025)."* Y **muestra algo que demuestre que eres tú** (tu perfil de GitHub, tu Teams o tu correo `@alu.edu.gva.es`). Di qué vas a hacer.
> 3. **Graba TODO el procedimiento**, explicando cada paso en voz alta mientras lo haces.
> 4. **Timestamps SIEMPRE** en la descripción: `00:00 Presentación` + uno por cada paso.
> 5. **Al terminar:** nombra el vídeo `V3.1 · Fase 1 — Infraestructura Cloud (AWS EC2 — Windows Server 2025)`, súbelo a tu playlist de YouTube **`B5_Windows_Nube`** (No listado) y **copia su enlace**.
> 6. **~8-10 min.** Esta fase es más larga que las de prerrequisitos: ve al grano, pero no te saltes pasos. Si se te va mucho, **pártela en dos vídeos** y ponlos los dos en la entrada.
> 7. **El enlace del vídeo va DENTRO de tu entrada de apuntes**, en el apartado `Enlace al vídeo explicativo`. Ahí, no en un papel.
> 8. **La entrega va por la TAREA de Teams.** Abriré una tarea que cubrirá **esta fase y otras**; te llegará notificación con fecha límite.

---

### 🎯 Objetivos de la fase

Al terminar esta fase serás capaz de:

- [ ] Explicar qué es IaaS y por qué usamos AWS en lugar de un servidor físico en el aula.
- [ ] Crear una instancia **EC2** con **Windows Server 2025** dimensionada de forma realista para un futuro controlador de dominio con interfaz gráfica.
- [ ] Configurar un **Security Group** que abre solo el puerto RDP (3389), entendiendo que el resto de puertos se irán abriendo fase a fase.
- [ ] Generar un **Key Pair (.pem)** y usarlo para **descifrar la contraseña inicial** del Administrador — un mecanismo distinto al login SSH de BoochanV3 y a la contraseña elegida en Azure de BoochanV2.1.
- [ ] Asignar una **Elastic IP** para tener una dirección fija.
- [ ] Conectarte al servidor por **RDP (Escritorio Remoto)** desde tu propio ordenador.
- [ ] Conocer el nombre de dominio (`BOOCHAN.SPACE`) que usará todo el proyecto BoochanV3.1.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.2_Entregables]] | [[Fase_1]] | [[Fase_1.4_Donde_Estamos]] |
