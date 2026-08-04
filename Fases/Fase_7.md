## 🕵️ Fase 7: Seguridad Avanzada (Permisos NTFS y Access-Based Enumeration)

### Infraestructura de Servidores Cloud (AWS)

> **[Módulo: SOR — Sistemas Operativos en Red]**
> **[U.T. 5 y 8: Gestión avanzada de permisos / Compartición SMB]**
>
> **Profesor:** Pedro Navarro Miralles  
> **Correo:** p.navarromiralles2@edu.gva.es  
> **Centro:** IES Jorge Juan (ALICANTE)
>
> **⏱️ Tiempo estimado:** ~1,5 horas (teoría + práctica + retos + troubleshooting)  
> **Requisitos:** Carpetas con cuota FSRM operativas | Rol de Servidor de Archivos disponible | Instancia EC2 `WindowsServer` (`t3.large`)
>
> **📦 Entrega:** una entrada de apuntes + un vídeo + la instantánea `Fase 7 terminada`

---

## 🧭 Índice de la fase

> [!warning] 📖 Esta fase va en diez documentos, no en uno
> Cada apartado es un fichero aparte, dentro de la carpeta `Fase_7/`. **Se leen en orden**, pero puedes volver a cualquiera: al final de cada uno tienes la navegación.
>
> **La fase completa es UNA sola entrega:** una entrada de apuntes y un vídeo, no diez.

| # | Apartado | Cuándo se lee |
| :--- | :--- | :--- |
| **1** | [[Fase_7.1_Que_Se_Evalua]] | Antes de encender la VM — qué se te evalúa |
| **2** | [[Fase_7.2_Entregables]] | Antes de encender la VM — qué debes producir |
| **3** | [[Fase_7.3_Obligaciones_Grabacion]] | Antes de arrancar OBS — cómo se graba y se entrega |
| **4** | [[Fase_7.4_Donde_Estamos]] | Antes de empezar — de dónde vienes y a dónde llegas |
| **5** | [[Fase_7.5_Fundamento_Teorico]] | Antes de teclear — los conceptos |
| **6** | [[Fase_7.6_Procedimiento]] | **Con la VM delante — aquí está el trabajo** |
| **7** | [[Fase_7.7_Resolucion_Problemas]] | Cuando algo no salga — búscate por el síntoma |
| **8** | [[Fase_7.8_Punto_de_Control]] | Al terminar, con la grabación aún en marcha |
| **9** | [[Fase_7.9_Preguntas]] | Después de la instantánea — trabajo de mesa |
| **10** | [[Fase_7.10_Auditoria_y_Cierre]] | Lo último — la checklist antes de seguir |

> [!tip] 💡 Cómo se recorre
> - Los apartados **1, 2 y 3** se leen **antes de encender nada**: son las reglas del juego.
> - El **4 y el 5** te preparan: contexto y conceptos.
> - El **6 es el trabajo**. El **7** solo si algo falla.
> - Los apartados **8, 9 y 10** cierran, **en ese orden**: primero aseguras la máquina, luego escribes, luego compruebas.

---

> [!abstract] 📋 Qué se te evalúa (resumen)
> **RA.04** — CE.04.a · CE.04.b · CE.04.c · CE.04.f
>
> El detalle: [[Fase_7.1_Que_Se_Evalua]]

**Siguiente al terminar los diez apartados:** Fase 8.
