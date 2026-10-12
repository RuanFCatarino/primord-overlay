# Contrato de integração — Rastro Hub ↔ Gamer Hause

Este documento é para implementar no painel `https://theislepainel.gamerhause.com.br/`. Não é uma lista de rotas já existentes. As únicas rotas confirmadas no JavaScript público atual são `GET /api/status`, `GET /api/me` e `GET/POST /api/friends`; `GET /api/status` responde `401` sem sessão Discord. O Rastro Hub nunca deve fingir que uma rota planejada já funciona.

## Sessão e privacidade

- O launcher não tem conta nem senha própria. O painel Gamer Hause é aberto na partição persistente `persist:rastro-ghs`; o Primord continua em `persist:primord-overlay`, com os cookies antigos preservados.
- Mantenha o OAuth Discord e a verificação de cargo no servidor. Se a sessão expirar, responda `401`; se o cargo faltar, `403`. O app mostra o login oficial, sem contornar o acesso.
- Para manter a conta conectada após reiniciar o app, emita uma sessão persistente `HttpOnly; Secure; SameSite=Lax` com `Expires`/`Max-Age` e política de validade/revogação definida pelo servidor. Nunca coloque token, segredo do Discord ou chave de administrador no HTML/JS público.
- Dados pessoais (dino, posição, amigos, carcaça, carteira) exigem a sessão do jogador. Dados públicos podem ter uma rota separada e limitada. Não exponha posição de outros jogadores sem autorização explícita.
- Respostas de telemetria: `Cache-Control: no-store`; sempre inclua `observed_at` UTC. O app deve tratar dados antigos como indisponíveis, não como online.
- Toda ação de escrita exige proteção CSRF compatível com sessão por cookie, rate limit e validação no servidor. O cliente nunca determina elegibilidade por conta própria.

## Rotas prioritárias

### `GET /api/overlay/v1/capabilities` — público

Informa quais módulos realmente existem, para o launcher não exibir botões falsos.

```json
{
  "version": 1,
  "features": {
    "server": true,
    "active_dino": false,
    "position": false,
    "map": false,
    "carcass": false,
    "friends": true,
    "events": false
  }
}
```

### `GET /api/overlay/v1/server` — público, somente leitura

Fonte da seleção de servidor e do contador ONLINE. `players.online` é a contagem atual, não a capacidade máxima. Não deduzir jogadores online de `max_players`.

```json
{
  "server_id": "gamer-hause-evrima",
  "name": "Gamer Hause — The Isle EVRIMA",
  "running": true,
  "players": { "online": 42, "max": 100 },
  "map_id": "gateway",
  "version": "0.x",
  "observed_at": "2026-10-12T12:00:00Z"
}
```

### `GET /api/overlay/v1/me` — sessão Discord obrigatória

Identidade para o launcher. Não retorne email, token OAuth ou dados administrativos.

```json
{
  "player_id": "opaque-stable-id",
  "display_name": "Jogador",
  "steam_linked": true,
  "in_game": true,
  "observed_at": "2026-10-12T12:00:00Z"
}
```

### `GET /api/overlay/v1/dino` — sessão obrigatória

`active` deve ser `null` quando não houver dino vivo/ativo. `id` muda ao nascer outro dino; isso impede carregar bloqueios/cooldown de uma vida anterior. Percentuais são números de `0` a `100`, ou `null` quando desconhecidos. Não use zero para valor desconhecido.

```json
{
  "active": {
    "id": "dino-life-opaque-id",
    "species": "Pteranodon",
    "diet": "carnivore",
    "growth_pct": 30,
    "hunger_pct": 22,
    "health_pct": 100,
    "thirst_pct": 90
  },
  "observed_at": "2026-10-12T12:00:00Z"
}
```

`diet` deve ser `carnivore`, `omnivore` ou `herbivore`. O servidor é a autoridade para regras da carcaça; a dieta serve apenas para visualização.

### `GET /api/overlay/v1/position` — sessão obrigatória

Retorne apenas a posição do próprio jogador. `online:false` e `position:null` quando ele sair do servidor. `map_x`/`map_y` são coordenadas normalizadas de `0` a `1`; documente a orientação dos eixos e associe a `map_id`. `observed_at` permite rejeitar posição congelada.

```json
{
  "online": true,
  "map_id": "gateway",
  "position": {
    "world_x": 12345.6,
    "world_y": -9876.5,
    "map_x": 0.576,
    "map_y": 0.360,
    "heading_deg": 145
  },
  "observed_at": "2026-10-12T12:00:00Z"
}
```

### `GET /api/overlay/v1/map` — público, somente metadados estáticos

O raster deve ter URL HTTPS/cacheável e versão ou hash para atualizar somente quando mudar. `width` e `height` são pixels da imagem original. Uma rota separada para marcadores dinâmicos evita invalidar a imagem inteira.

```json
{
  "map_id": "gateway",
  "image_url": "https://theislepainel.gamerhause.com.br/static/gateway-v1.webp",
  "image_version": "sha256-or-version",
  "width": 4096,
  "height": 4096,
  "available_layers": ["sanctuary", "migration", "patrol", "poi"]
}
```

### `GET /api/overlay/v1/map/markers` — conforme a privacidade da camada

Cada marcador precisa de `id`, `type`, `map_x`, `map_y`, `label`, `updated_at`. Camadas públicas podem ser anônimas. Marcadores de amigos exigem autorização e devem respeitar a visibilidade escolhida por cada amigo. Não exponha a posição de todos os jogadores por padrão.

### `GET /api/overlay/v1/carcass` — sessão obrigatória

Retorne o estado autoritativo para o dino atual. `eligible` não deve ser inferido apenas de fome/crescimento. `cooldown_until` é UTC e `null` quando não existe cooldown. `hide_after` é opcional: o app pode ocultar o cartão após um estado terminal, mas nunca apagar o mapa.

```json
{
  "dino_id": "dino-life-opaque-id",
  "state": "available",
  "eligible": true,
  "reasons": [],
  "hunger_pct": 22,
  "growth_pct": 30,
  "carcass_type": "Beipiaosaurus",
  "cooldown_until": null,
  "observed_at": "2026-10-12T12:00:00Z"
}
```

Estados sugeridos: `available`, `cooldown`, `blocked`, `expired`, `unknown`. Motivos estruturados: `hunger_too_high`, `growth_limit`, `herbivore`, `not_in_game`, `cooldown`. Inclua limites em campos próprios se puderem mudar por espécie/servidor.

### `POST /api/overlay/v1/carcass` — sessão obrigatória, ação explícita

Chamado somente depois de o jogador apertar o botão/atalho e confirmar. Envie `Idempotency-Key` único por tentativa e o `dino_id` atual; o servidor valida sessão, dino, posição, regras e cooldown, sem confiar em `eligible` vindo do app. Responda `200/201` com estado atualizado, `409` se já solicitado, `422` se bloqueado, `429` se limitado.

```json
{ "dino_id": "dino-life-opaque-id" }
```

## Tempo real

Preferência: `GET /api/overlay/v1/events` por Server-Sent Events autenticado, emitindo `server`, `dino`, `position`, `map_markers` e `carcass` com `event_id` crescente e horário UTC. O app reconecta e consulta os snapshots acima para recuperar eventos perdidos. Alternativa: polling de `/server` a cada 10–15 s e `/dino`, `/position`, `/carcass` a cada 2–5 s enquanto o Modo Jogo está ativo. Use `ETag` se o volume crescer.

## Módulos opcionais, depois do núcleo

- `GET /api/overlay/v1/friends`: amigos aceitos e visíveis; pode reaproveitar o `/api/friends` atual se o esquema e a permissão forem suficientes.
- `GET /api/overlay/v1/wallet`: saldo somente do jogador autenticado.
- `GET /api/overlay/v1/storage`: slots e dinos guardados; ações de guardar/retirar apenas com confirmação e validação no servidor.
- `GET /api/overlay/v1/notifications`: eventos pessoais com `id`, `type`, `created_at`, `read_at`.

Não inclua APIs administrativas, RCON, chaves de Steam/Discord, credenciais do servidor ou ações destrutivas no launcher público. Se um dado não existe no backend, marque a capability como `false` e o Rastro Hub mostrará “não disponível” em vez de inventar valores.

