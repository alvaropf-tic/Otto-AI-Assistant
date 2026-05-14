# Otto V1.4 – Arquitectura y flujo de trabajo

Esta es la primera versión estable con las principales características para las que nació Otto.

## Características
- Recibe mensajes por Telegram (texto o nota de voz).
- Transcribe audio a texto usando Groq Whisper (gratuito).
- Clasifica la intención del usuario (leer, crear, editar, borrar notas, o conversar).
- Lee y escribe notas en un vault de Obsidian (carpetas: Inbox, Proyectos, Diario, Contactos, Finanzas, Aprendizaje, Sistema_Otto).
- Mantiene un historial de conversación (últimas 30 interacciones) y lo archiva semanalmente.
- Responde por texto o convierte la respuesta a voz (usando Google TTS, no oficial) y la envía como nota de voz si el usuario original envió un audio.
- Gestiona índices automáticos (Index.md) en cada carpeta y elimina enlaces rotos al borrar notas (⚠️ la papelera no funciona, el borrado es permanente).

// Pegar foto del flujo
