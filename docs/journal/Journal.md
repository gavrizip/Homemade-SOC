
## 2026-07-06

- **Done:** SVG network achitecture diagram, IRP flow diagram, lab architecture diagram, made references folder with each
- **Problems:** SVG problems
- **Next:** Continue creating new entries in "docs/reference/" folder


## 2026-07-07

- **Done:** fiinshed the "docs/reference/" folder, started with readteam folder
- **Problems:** This could be so much for this project, maybe it could be better if I explore this field in a side project related to this one.
- **Next:** Start

## 2026-07-10 

Layer: architecture + Ollama setup
### Done
- Definida la arquitectura del nuevo apartado de IA del proyecto (detección ML, automatización/triage con LLM, seguridad de IA, meta-documentación), como carpeta raíz independiente `/ai/`.
- Estructura de carpetas propuesta y creada (`detection/`, `automation/`, `llm-security/`, `meta/`) vía comando zsh.
- Diagrama de arquitectura de la capa IA (`ai-architecture.svg`) diseñado y **rediseñado dos veces** tras feedback de legibilidad — versión final en fila horizontal única (pipeline izquierda→derecha) en vez de paneles con conectores diagonales.
- Decisión de arquitectura clave: el **LLM Triage Assistant será self-hosted (Ollama)**, no una API externa — ningún dato de alertas sale de VLAN 20.
- Estructura interna de `ai/automation/` definida (`setup/`, `prompts/`, `docs/`).
- Creado `ai/automation/setup/01-ollama-server-setup.md`: instalación de drivers NVIDIA, Ollama, y selección de modelo ajustada a la GPU real disponible (GTX 1060, 3GB VRAM) — recomendado `llama3.2:3b` como base, con benchmarking de tiempo de respuesta incluido.

### Problemas / bloqueos
- La decisión local vs. API para el LLM no estaba tomada y bloqueaba avanzar con el diseño del pipeline — resuelto: local.
- 3GB de VRAM es una limitación real: la mayoría de modelos 7B no entran en 4-bit sin offload parcial a CPU (con pérdida notable de velocidad). 
- **`02-network-integration.md` bloqueo**: el servidor Ollama es la primera máquina física del proyecto (todo lo demás es VMware), y para que entre en VLAN 20 necesito confirmar si puedo disponer de un switch gestionado con soporte 802.1Q - pendiente de resolver.

### Next
- Responder si hay switch gestionado con 802.1Q para poder cerrar `02-network-integration.md`.
- `03-shuffle-integration.md` — cómo Shuffle llama al endpoint local de Ollama.
- `ai/automation/docs/ollama.md` — doc de referencia con el template estándar (qué es / rol / justificación / limitaciones / integración).
- Resto de `/ai/` aún sin contenido: `detection/`, `llm-security/`, `meta/`.






___