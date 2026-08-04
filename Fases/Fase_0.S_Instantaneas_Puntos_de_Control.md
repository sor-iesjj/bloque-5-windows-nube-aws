## 💾 Puntos de control: cómo volver atrás

### El seguro que te permite experimentar sin miedo

> **[Módulo: SOR — Sistemas Operativos en Red]**
> **Profesor:** Pedro Navarro Miralles · IES Jorge Juan (ALICANTE)

---

> [!warning] 📖 Cómo se usa este documento
> **Esto no es una fase y no se entrega.** Es una técnica que aplicarás **al terminar cada fase**, y una instrucción que verás repetida en el apartado 8 de todas ellas.
>
> Léelo **una vez, antes de empezar la Fase 1**.

---

### 🎯 El problema que resuelve

> [!danger] La situación que te vas a encontrar
> Estás en la Fase 4, montando el dominio. Algo sale mal, intentas arreglarlo y lo empeoras. Al cabo de una hora tu servidor está en un estado que no entiendes y que no se parece a nada de lo que describe el manual.
>
> **Sin punto de control, solo tienes una opción: empezar de cero.** Con uno, vuelves al último estado bueno en minutos y repites solo la fase que se torció.

> [!success] Y sirve para algo más: para experimentar
> Un punto de control no es solo un seguro contra catástrofes. Es lo que te permite **romper cosas a propósito** para ver qué pasa.
>
> ¿Qué ocurre si te saltas un paso? ¿Si pones la máscara mal? Hazlo, míralo, aprende, y vuelve atrás.
>
> **Un administrador que puede volver atrás experimenta. Uno que no, obedece instrucciones por miedo a romper algo.**

---

### 🛠️ Cómo se hace en AWS

> [!warning] ⚠️ En la nube NO hay "instantáneas de VirtualBox"
> El concepto es el mismo, pero el mecanismo cambia: aquí se hace una **AMI** (imagen de la instancia) o una **instantánea EBS** del volumen.

> [!example] Crear una AMI (lo más práctico)
> 1. **Detén la instancia** para que el volumen quede consistente.
> 2. Consola EC2 → tu instancia → **`Acciones` → `Imagen y plantillas` → `Crear imagen`**.
> 3. Nómbrala **`Fase N terminada`**.
>
> Por CLI:
> ```bash
> aws ec2 create-image --instance-id i-TUINSTANCIA --name "Fase-N-terminada" --no-reboot
> ```

> [!example] Volver atrás
> Se **lanza una instancia nueva** a partir de la AMI. No se "restaura" la existente: en AWS las máquinas son desechables y se reemplazan, no se reparan. Es una filosofía distinta de la de un servidor local, y conviene entenderla.
>
> ⚠️ Si lanzas una instancia nueva, **su IP privada puede cambiar** — y el dominio quedaría apuntando a la vieja.

> [!danger] 💰 Las AMI y los snapshots CUESTAN dinero
> Ocupan almacenamiento EBS facturable. En el **Learner Lab** además consumen tu crédito. Borra las que no necesites, y al terminar el proyecto elimina instancias, volúmenes e imágenes.
---

> [!summary] 🎓 Qué has aprendido
> Que **antes de tocar algo importante, se guarda el estado al que poder volver**. Cambia el nombre según dónde estés —instantánea, punto de control, snapshot, imagen— pero la idea es la misma en todas partes, y en el código se llama `commit`.
>
> **No avances sin poder retroceder.**
