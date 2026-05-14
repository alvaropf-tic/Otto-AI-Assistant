# Otto — Personal AI Assistant

<!-- GIF: Demostración del bot respondiendo por Telegram (texto y voz) -->

<p align="center">
  <img src="https://flagcdn.com/w40/es.png" width="20"> <strong>Español</strong> | 
  <a href="README.en.md"><img src="https://flagcdn.com/w40/gb.png" width="20"> English</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/n8n-workflow-orange" />
  <img src="https://img.shields.io/badge/Groq-LLaMA%203.3%2070B-blue" />
  <img src="https://img.shields.io/badge/Obsidian-vault-purple" />
  <img src="https://img.shields.io/badge/Telegram-bot-2CA5E0" />
  <img src="https://img.shields.io/badge/license-MIT-green" />
</p>

---

## <img src="https://flagcdn.com/w40/es.png" width="20"> Español

Asistente personal de IA que corre 24/7 en una Raspberry Pi, accesible desde Telegram. Capaz de guardar, editar y consultar notas en Obsidian, recordar conversaciones anteriores y responder tanto por texto como por voz.

### 🏗️ Infraestructura

<!-- IMAGEN: Foto o diagrama de la Raspberry Pi con el SSD conectado -->

| Componente | Detalle |
|---|---|
| Hardware | Raspberry Pi 4 (4GB RAM) |
| Sistema operativo | DietPi |
| Almacenamiento | SSD 256GB (montado en `/mnt/ssd`) |
| Contenedores | Docker + Portainer |
| Acceso remoto | Tailscale (VPN privada) |
| Sincronización | Syncthing (PC Windows + Android) |

### 🛠️ Tecnologías

| Tecnología | Uso |
|---|---|
| [n8n](https://n8n.io/) | Orquestación del flujo de trabajo |
| [Groq](https://groq.com/) | LLaMA 3.3 70B (texto) + Whisper (voz a texto) |
| [Telegram Bot API](https://core.telegram.org/bots/api) | Interfaz de mensajería |
| [Obsidian](https://obsidian.md/) | Base de conocimiento (vault en markdown) |
| [Syncthing](https://syncthing.net/) | Sincronización del vault entre dispositivos |
| Google Translate TTS | Texto a voz (endpoint no oficial, gratuito) |

### 🗂️ Estructura del vault

<!-- IMAGEN: Captura del Graph View de Obsidian mostrando las notas conectadas -->

```
Otto Vault/
├── Home.md                  ← Dashboard principal
├── Inbox/                   ← Notas rápidas sin clasificar
├── Proyectos/               ← Cosas con principio y fin
├── Diario/                  ← Entradas diarias personales
├── Contactos/               ← Información de personas
├── Finanzas/                ← Gastos, ingresos, presupuestos
├── Aprendizaje/             ← Libros, cursos, recursos
├── Sistema_Otto/            ← Configuración y memoria de Otto
├── Historiales/             ← Historial semanal archivado
└── Papelera/                ← Notas eliminadas
```

Cada carpeta tiene su propio `Index.md` que enlaza todas sus notas. El `Home.md` enlaza todos los índices, creando un grafo de conocimiento navegable en Obsidian.

### 📋 Requisitos

- Raspberry Pi 4 (recomendado 4GB RAM) o tu homelab
- Docker y Portainer instalados
- Cuenta en [Groq](https://console.groq.com/) (gratuita)
- Bot de Telegram creado con [@BotFather](https://t.me/BotFather)
- Obsidian instalado en PC y/o Android
- Syncthing en todos los dispositivos

### 🔧 Instalación rápida

1. Clona el repositorio
2. Importa el workflow JSON en n8n
3. Configura las credenciales (Groq API key + Telegram Bot Token)
4. Ajusta las variables de entorno
5. Activa el workflow

## 🚨 AVISO

Es muy probable que necesites modificar las rutas de las constantes o variables en los nodos "Código" y "Agente IA". Además, de necesitar habilitar ciertas funciones en el archivo .yaml para que funcionen de algunas características.

---

## 📦 Versiones

### V1.0 — Estructura funcional inicial

**Características principales:**
- Clasificación de carpetas mediante LLM (AI Agent 1)
- Lectura de notas relevantes desde el vault de Obsidian
- Creación y edición básica de notas en markdown
- Respuesta solo por texto vía Telegram

**Flujo:**
```
Telegram Trigger
  → AI Agent 1 (elige carpetas)
  → Code (lee notas)
  → AI Agent 2 (genera respuesta)
  → Code parsear JSON
  → If is_note
      ├── True  → Escribe nota → Telegram
      └── False → Telegram
```

<!-- IMAGEN: Captura del workflow V1.0 en n8n -->

---

### V1.1 — Contexto y ciclo de vida de notas

**Nuevas características:**
- Historial de conversación persistente en `.historial.json`
- Soporte completo para editar y borrar notas
- Lógica de bifurcación `is_note` mejorada
- El asistente recuerda interacciones anteriores

**Flujo añadido:**
```
→ Code Lee Historial (antes del AI Agent 2)
→ Code Escribe Historial True/False (después de responder)
```

**Punto débil conocido:** el borrado era permanente, sin papelera.

<!-- IMAGEN: Captura del workflow V1.1 en n8n -->

---

### V1.2 — Papelera e índices

**Nuevas características:**
- **Papelera** — las notas eliminadas se mueven a `Papelera/` en lugar de borrarse ⚠️
- **Índices automáticos** — cada carpeta tiene un `Index.md` que se actualiza solo al crear/borrar notas
- **Home.md** — dashboard central que enlaza todos los índices
- **Graph View** — las notas aparecen conectadas visualmente en Obsidian

**⚠️ Issue conocido:** la funcionalidad de papelera no funciona actualmente. Se intentara corregir en próximas versiones.

<!-- IMAGEN: Captura del Graph View de Obsidian con los grupos de carpetas -->

---

### V1.3 — Limpieza y robustez

**Nuevas características:**
- **Eliminación de enlaces rotos** — al borrar una nota, se eliminan todas las referencias `[[nota]]` en el resto del vault mediante `removeLinksFromVault()`
- **Parseo de JSON mejorado** — extracción robusta del JSON aunque el LLM incluya texto extra alrededor
- **Rotación automática del historial** — cada semana se archiva `historial.json` en `Historiales/YYYY-MM-DD.json` y se inicia uno nuevo (Schedule Trigger los domingos a las 3am, configurable al dia y hora de tu elección)
- **Optimización de tokens** — el AI Agent 1 ahora detecta la intención (`list`, `read`, `create`, `edit`, `delete`, `chat`) y el Code solo envía el contexto necesario.
**Detalles técnicos:**
- `Code Títulos` añadido antes del AI Agent 1 para proporcionar el listado completo del vault
- Structured Output Parser con JSON Schema flexible para soportar múltiples `note_title` separados por coma
- La papelera funciona correctamente a partir de esta versión

<!-- IMAGEN: Captura del workflow V1.3 en n8n mostrando el flujo completo -->

---

### V1.4 — Experiencia multimodal (voz)

**Nuevas características:**
- **Entrada de voz** — los audios de Telegram se transcriben con Groq Whisper
- **Salida de voz** — la respuesta se convierte a audio con Google TTS y se envía como nota de voz
- **Flujo de audio separado** — un nodo Switch detecta si el mensaje es texto o audio
- **Bandera `isAudio`** — controla el formato de respuesta en todo el flujo
- **Limpieza de caracteres especiales** — evita errores de parseo en Telegram Markdown

**Subflujo de audio:**
```
Switch (texto/audio)
  └── Audio → Get File → HTTP Descarga → HTTP Groq Whisper
            → Code Transcripción (inyecta texto en flujo normal)
            → [flujo normal]
            → If isAudio
                ├── True  → HTTP Google TTS → Send Audio
                └── False → Send Text
```

<!-- GIF: Demostración enviando un audio y recibiendo respuesta de voz -->

---

## 🚀 Mejoras planificadas (V1.5 / V2.0)

| Mejora | Descripción | En desarrollo |
|---|---|---|
| Mejora prompt| Mejora del prompt de Agent AI 2 para textos muy largos que rompian el json | ✅ |
| Búsqueda web | Integración con Tavily para consultas de información actual (1000 creditos al mes) | ✅ |
| Fix historial Agent 2 | El historial ahora llega correctamente al prompt del Agent 2 | ✅ |
| Fix Historial audio | Los mensajes de voz se guardan transcritos en el historial | ✅ |
| Personalidad Otto | Tono directo, sin frases de relleno, con carácter propio | ✅ |
| Confirmaciones | Respuestas como "sí" o "dale" crean la nota sin volver a preguntar | ✅ |
| Modularización | Sub-workflows para Historial Manager, Vault Reader y TTS Service |  |
| Manejo de errores | Error Trigger para fallos en Groq, Telegram o I/O de archivos | 🔜 |
| Variables de entorno | Externalizar rutas hardcodeadas como `OBSIDIAN_VAULT_PATH` |  |
| Campo `confidence` | El AI Agent 1 pide confirmación antes de acciones destructivas si la confianza es baja |  |
| Few-shot en prompts | Ejemplos en el prompt del AI Agent 1 para mejorar precisión |  |
| Búsqueda semántica | Embeddings con ChromaDB para encontrar notas sin usar el título exacto |  |
| Resumen diario | Schedule Trigger que envía cada mañana las tareas pendientes |  |
| Comandos slash | `/stats`, `/search`, `/export` via `botCommand` en Telegram | 🔜 |
| Inline Keyboard Markup | Para poner el audio en auto, no, si| 🔜 |
| PDFs | Subir PDFs y que Otto los use como contexto para responder preguntas |  |
| Base de datos | Migración a PostgreSQL para historial y metadatos cuando el volumen lo requiera |  |
| Spotify | Abrir en navegador de PC personal la ventana de Spotify con música |  |
| Gmail | Que te diga si tienes algun correo sin leer y de quien es (Prohibido que Otto coneste emails/mensajes) | 🔜 |
| Reproduccion de audios anteriores en Telegram| Pasar de enviar con nodo "Send an audio file" a "Send a voice message" | 🔜 |
| --- | --- | --- |
| IA | Cambiar el modelo de IA (V2.0)|  |
| BD | Cambio de base de datos, pasar de guardar en Obsidian a otro tipo de BD (V2.0) |  |
| Fisico | Hacer que Otto tenga presencia fisica, un ESP32 con micrófono, pantalla y altavoz (V2.0) |  |

---

## 📄 Licencia

MIT — siéntete libre de usar, adaptar y aprender de este proyecto.
