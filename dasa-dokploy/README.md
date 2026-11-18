# Dokploy for Umbrel

Questo pacchetto esegue Dokploy con Traefik v3, PostgreSQL 16 e Redis 7 su architettura Docker Swarm con rete overlay isolata `dokploy-network`.

## Porte

| Porta | Scopo |
|---:|---|
| `8528` | Dashboard web di Dokploy tramite Umbrel App Proxy |
| `8529` | Entry point HTTP per le applicazioni e i servizi deployati da Dokploy |
| `8530` | Entry point HTTPS e HTTP/3 per le applicazioni e i servizi deployati da Dokploy |

Per instradare traffico pubblico verso le applicazioni create in Dokploy, punta il tuo dominio o reverse proxy verso le porte `8529` e `8530`.

## Architettura

- **Routing integrato**: Traefik gestisce i certificati SSL/TLS e il reverse proxy automatico verso i container connessi a `dokploy-network`.
- **Database & Cache**: istanze dedicate di PostgreSQL 16 e Redis 7 con script di migrazione automatica delle password.
- **Hook pre-start**: inizializza automaticamente il cluster locale Docker Swarm e la rete overlay attachable `dokploy-network`.
