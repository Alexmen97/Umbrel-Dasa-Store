# Honcho + OpenConcho on Umbrel

This package includes Honcho **3.2.2** and OpenConcho **0.16.2**, both pinned by image digest for Linux AMD64 and ARM64.

Open Honcho from the Umbrel dashboard to launch OpenConcho. The first browser visit automatically connects to the bundled API at `http://api:8000`. This is a Docker-internal address: OpenConcho proxies requests server-side, so the browser does not need Docker DNS or CORS configuration.

## Addresses

- Dashboard: `http://<umbrel-host>:8000/`
- Honcho API: `http://<umbrel-host>:8000/v3/...`
- API documentation: `http://<umbrel-host>:8000/docs`
- Honcho health: `http://<umbrel-host>:8000/health`

Existing clients can retain `http://<umbrel-host>:8000` as the Honcho base URL. Inside the stack, services use `http://api:8000`.

The dashboard's upstream allowlist permits only `api`. Adding arbitrary remote instances in OpenConcho Settings is therefore not supported by this package's default proxy configuration.

## Data and settings

Honcho retains its existing PostgreSQL, Redis, and app-data mounts. OpenConcho stores connection preferences in the browser, so it does not need a server-side data volume. Preferences are specific to that browser and origin; clearing browser storage removes them, but does not remove Honcho memories.

Provider credentials and model configuration must be supplied to both Honcho API and deriver using the [Honcho v3 configuration reference](https://github.com/plastic-labs/honcho/blob/v3.2.2/config.toml.example). They are needed for AI actions; this store does not include API keys or private provider endpoints.

The existing package runs with Honcho authentication disabled. Restrict access to a trusted network or configure authentication before exposing it publicly.

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
5. Apri `http://umbrel.local:8000/`, crea o seleziona un workspace e prova una richiesta di chat su un peer. Per provare l'elaborazione della memoria, aggiungi messaggi a una sessione e controlla le conclusioni in seguito: il deriver lavora in modo asincrono e può attendere il raggiungimento delle soglie di elaborazione.

Honcho **3.2.2** usa di default `openai / gpt-5.4-mini` per la generazione del testo e `openai / text-embedding-3-small` per gli embedding (1536 dimensioni). La chiave deve avere accesso a entrambi i modelli e credito o fatturazione API disponibili. Non serve aggiungere variabili di modello se mantieni questi valori.

Le chiavi vanno nei servizi **api e deriver**, mai in **openconcho**, PostgreSQL o Redis. Il campo token nelle impostazioni di OpenConcho riguarda l'autenticazione verso Honcho: **non è una chiave OpenAI, Anthropic o Gemini**. Con l'autenticazione Honcho disabilitata in questo pacchetto, lascialo vuoto. Mantieni `http://api:8000` come indirizzo interno della connessione alla dashboard.

### Anthropic, Gemini e modelli personalizzati

Aggiungi la credenziale in **entrambi** i servizi `api` e `deriver`:

| Provider | Nome della variabile | Valore del transport |
| --- | --- | --- |
| OpenAI | `LLM_OPENAI_API_KEY` | `openai` |
| Anthropic | `LLM_ANTHROPIC_API_KEY` | `anthropic` |
| Gemini | `LLM_GEMINI_API_KEY` | `gemini` |

**Aggiungere una chiave non cambia i modelli predefiniti.** Per cambiare il provider del testo, imposta anche le seguenti coppie di variabili in entrambi i servizi. Per ogni riga usa il transport desiderato e un identificativo di modello effettivamente disponibile presso quel provider; non usare il nome di un modello OpenAI con il transport Anthropic o Gemini.

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

Ad esempio, per usare Anthropic per l'estrazione delle conclusioni imposta `DERIVER_MODEL_CONFIG__TRANSPORT=anthropic` e `DERIVER_MODEL_CONFIG__MODEL` all'identificativo del modello scelto, oltre a `LLM_ANTHROPIC_API_KEY`. Ripeti per le altre funzioni che vuoi spostare. Le funzioni senza override continuano a usare OpenAI.

Gli embedding sono separati dai modelli di chat. Puoi mantenere OpenAI per gli embedding anche usando Anthropic o Gemini per il testo: in quel caso conserva anche `LLM_OPENAI_API_KEY`. Anthropic non fornisce un transport di embedding in Honcho. Per configurare un altro modello di embedding supportato usa `EMBEDDING_MODEL_CONFIG__TRANSPORT`, `EMBEDDING_MODEL_CONFIG__MODEL` e `EMBEDDING_VECTOR_DIMENSIONS`. Se hai già memorie salvate, pianifica la migrazione e la rigenerazione dei vettori prima di cambiare modello o dimensioni.

### Endpoint OpenAI compatibili

Per un endpoint personalizzato, usa `transport=openai`, il modello offerto dall'endpoint e un override per ciascuna funzione interessata. Esempio per il deriver:

```text
DERIVER_MODEL_CONFIG__TRANSPORT=openai
DERIVER_MODEL_CONFIG__MODEL=<identificativo del modello>
DERIVER_MODEL_CONFIG__OVERRIDES__BASE_URL=<URL base del provider>
DERIVER_MODEL_CONFIG__OVERRIDES__API_KEY_ENV=LLM_CUSTOM_API_KEY
LLM_CUSTOM_API_KEY=<chiave del provider>
```

Inserisci ogni riga come variabile personalizzata in `api` e `deriver`, sostituendo i segnaposto. Puoi applicare gli stessi suffissi `__OVERRIDES__BASE_URL` e `__OVERRIDES__API_KEY_ENV` ai prefissi di configurazione delle altre funzioni elencate sopra. Per gli embedding il prefisso è `EMBEDDING_MODEL_CONFIG`: l'endpoint e il modello scelti devono supportare realmente gli embedding, non soltanto la chat.

### Verifica e problemi comuni

- **Connected** in OpenConcho e `/health` indicano che il server risponde; non dimostrano che le chiavi dei provider funzionino.
- Errori **401/403** durante una richiesta AI: controlla chiave, autorizzazioni del modello e provider selezionato.
- Errori di quota o **429**: controlla credito API e limiti del provider.
- Modello non trovato: verifica l'identificativo esatto e il transport configurato.
- Chat funzionante ma memoria non elaborata: controlla che anche **deriver** abbia le variabili, poi verifica soglie di elaborazione e log in **Impostazioni → Avanzate → Visualizza log**.

Salva le credenziali soltanto nelle impostazioni locali di Umbrel. Non inserirle nel manifest, nel compose del repository, nei commit o in screenshot pubblici. Le richieste AI inviano i contenuti necessari al provider configurato.

Riferimenti della versione installata: [configurazione Honcho 3.2.2](https://github.com/plastic-labs/honcho/blob/v3.2.2/config.toml.example) e [nomi e valori predefiniti delle impostazioni](https://github.com/plastic-labs/honcho/blob/v3.2.2/src/config.py).

## Upgrades and verification

Store revision `3.2.2-1` adds the dashboard without changing the Honcho backend version. Back up PostgreSQL before upgrading from Honcho 2.x; the API entrypoint provisions/migrates the database before starting and OpenConcho waits for API health.

Verify on Umbrel: open the dashboard, list/create a workspace, confirm `/health` returns JSON and `/docs` opens Swagger, then check an existing API client. AI chat additionally requires valid provider configuration. Container launch, data migration, and browser behavior require runtime validation on the actual Umbrel device.
