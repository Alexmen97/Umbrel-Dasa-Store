# Honcho on Umbrel

This package includes Honcho **3.2.2** pinned by image digest for Linux AMD64 and ARM64.

Honcho runs as an AI-native memory server, exposing its FastAPI service and interactive Swagger documentation.

## Addresses

- API documentation (Swagger UI): `http://<umbrel-host>:8000/docs`
- Honcho API: `http://<umbrel-host>:8000/v3/...`
- ReDoc documentation: `http://<umbrel-host>:8000/redoc`
- Honcho health: `http://<umbrel-host>:8000/health`

## Stack Architecture

- **`api`**: Honcho FastAPI backend (runs migrations automatically at startup, exposes port 8000).
- **`deriver`**: Background worker distilling multi-turn conversations into declarative insights and conclusions.
- **`postgres`**: PostgreSQL 15 with `pgvector` for semantic and relational vector retrieval.
- **`redis`**: Caching and session state reconciliation.

## Data and settings

Honcho retains its PostgreSQL, Redis, and app-data mounts.

Provider credentials and model configuration must be supplied to both Honcho API and deriver using the [Honcho v3 configuration reference](https://github.com/plastic-labs/honcho/blob/v3.2.2/config.toml.example). They are needed for AI actions; this store does not include API keys or private provider endpoints.

The package runs with Honcho authentication disabled (`AUTH_USE_AUTH=false`). Restrict access to a trusted network or configure authentication before exposing it publicly.

## Tutorial: configurare le credenziali dei provider

### Avvio con la configurazione predefinita OpenAI

1. Nella home di Umbrel fai **clic destro su Honcho → Impostazioni → Avanzate → Variabili d'ambiente → Aggiungi variabile personalizzata**.
2. Seleziona **Servizio dell'app: api**, **Nome: LLM_OPENAI_API_KEY**, **Valore: la tua chiave API** e premi **Aggiungi variabile**.
3. Ripeti per **deriver**, usando lo stesso nome e la stessa chiave. Le due righe devono essere:

   | Servizio dell'app | Nome | Valore |
   | --- | --- | --- |
   | `api` | `LLM_OPENAI_API_KEY` | La tua chiave API |
   | `deriver` | `LLM_OPENAI_API_KEY` | La stessa chiave API |

4. Applica le modifiche e attendi che Honcho torni pronto. Se Umbrel non riavvia automaticamente l'app, scegli **Riavvia** dal menu di Honcho.
5. Apri `http://umbrel.local:8000/docs` per esplorare e testare gli endpoint interattivi.

Honcho **3.2.2** usa di default `openai / gpt-5.4-mini` per la generazione del testo e `openai / text-embedding-3-small` per gli embedding (1536 dimensioni). La chiave deve avere accesso a entrambi i modelli e credito o fatturazione API disponibili. Non serve aggiungere variabili di modello se mantieni questi valori.

Le chiavi vanno nei servizi **api e deriver**, mai in PostgreSQL o Redis.

### Anthropic, Gemini e modelli personalizzati

Aggiungi la credenziale in **entrambi** i servizi `api` e `deriver`:

| Provider | Nome della variabile | Valore del transport |
| --- | --- | --- |
| OpenAI | `LLM_OPENAI_API_KEY` | `openai` |
| Anthropic | `LLM_ANTHROPIC_API_KEY` | `anthropic` |
| Gemini | `LLM_GEMINI_API_KEY` | `gemini` |

**Aggiungere una chiave non cambia i modelli predefiniti.** Per cambiare il provider del testo, imposta anche le seguenti coppie di variabili in entrambi i servizi:

| Funzione | Variabile transport | Variabile modello |
| --- | --- | --- |
| Estrazione delle conclusioni | `DERIVER_MODEL_CONFIG__TRANSPORT` | `DERIVER_MODEL_CONFIG__MODEL` |
| Riassunti | `SUMMARY_MODEL_CONFIG__TRANSPORT` | `SUMMARY_MODEL_CONFIG__MODEL` |
| Dream, deduzione | `DREAM_DEDUCTION_MODEL_CONFIG__TRANSPORT` | `DREAM_DEDUCTION_MODEL_CONFIG__MODEL` |
| Dream, induzione | `DREAM_INDUCTION_MODEL_CONFIG__TRANSPORT` | `DREAM_INDUCTION_MODEL_CONFIG__MODEL` |
| Chat, minimal | `DIALECTIC_LEVELS__minimal__MODEL_CONFIG__TRANSPORT` | `DIALECTIC_LEVELS__minimal__MODEL_CONFIG__MODEL` |
| Chat, low | `DIALECTIC_LEVELS__low__MODEL_CONFIG__TRANSPORT` | `DIALECTIC_LEVELS__low__MODEL_CONFIG__MODEL` |
| Chat, medium | `DIALECTIC_LEVELS__medium__MODEL_CONFIG__TRANSPORT` | `DIALECTIC_LEVELS__medium__MODEL_CONFIG__MODEL` |
| Chat, high | `DIALECTIC_LEVELS__high__MODEL_CONFIG__TRANSPORT` | `DIALECTIC_LEVELS__high__MODEL_CONFIG__MODEL` |
| Chat, max | `DIALECTIC_LEVELS__max__MODEL_CONFIG__TRANSPORT` | `DIALECTIC_LEVELS__max__MODEL_CONFIG__MODEL` |

### Endpoint OpenAI compatibili

Per un endpoint personalizzato, usa `transport=openai`, il modello offerto dall'endpoint e un override per ciascuna funzione interessata. Esempio per il deriver:

```text
DERIVER_MODEL_CONFIG__TRANSPORT=openai
DERIVER_MODEL_CONFIG__MODEL=<identificativo del modello>
DERIVER_MODEL_CONFIG__OVERRIDES__BASE_URL=<URL base del provider>
DERIVER_MODEL_CONFIG__OVERRIDES__API_KEY_ENV=LLM_CUSTOM_API_KEY
LLM_CUSTOM_API_KEY=<chiave del provider>
```

Salva le credenziali soltanto nelle impostazioni locali di Umbrel. Non inserirle nel manifest o nel compose del repository.
