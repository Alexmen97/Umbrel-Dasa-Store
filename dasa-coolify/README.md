# Coolify for Umbrel

Questo pacchetto esegue Coolify con un demone Docker dedicato e isolato in modalità Docker-in-Docker. Coolify non riceve accesso al socket Docker principale di Umbrel e non può interferire con i container delle altre applicazioni.

## Porte

| Porta | Scopo |
|---:|---|
| `8527` | Dashboard web di Coolify tramite Umbrel App Proxy |
| `6001` | Eventi realtime e aggiornamenti di deployment |
| `6002` | WebSocket del terminale da browser |
| `8531` | Entry point HTTP per le applicazioni deployate da Coolify |
| `8532` | Entry point HTTPS e HTTP/3 per le applicazioni deployate da Coolify |

Per esporre pubblicamente le applicazioni create all'interno di Coolify, inoltra le porte esterne desiderate verso `8531` e `8532`, oppure punta un reverse proxy esterno verso queste porte.

## Architettura

Coolify si connette via SSH al servizio `coolify-testing-host`. La CLI Docker utilizza un socket Unix appartenente esclusivamente al demone `coolify-docker` (dind). I database PostgreSQL e Redis e il servizio realtime utilizzano nomi DNS dedicati per prevenire conflitti con altre app su Umbrel. Lo stato persistente viene salvato in `${APP_DATA_DIR}/data/`.

## Permessi

Nessun comando `chmod` o `chown` manuale è necessario: un servizio di inizializzazione one-shot (`coolify-permissions`) crea la struttura delle cartelle necessarie e imposta i permessi per l'utente UID `9999` prima dell'avvio dei servizi.
