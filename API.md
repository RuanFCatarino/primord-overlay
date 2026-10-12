# API local do Primord Overlay (v1)

Base: `http://127.0.0.1:37621`. A API só atende conexões no mesmo computador e é **opcional**: o launcher funciona sem qualquer integração. Ela não é uma API oficial do Painel Primord e não oferece acesso a arquivos, memória do jogo ou ações do painel.

## Autenticação

Na primeira abertura, o launcher cria um token aleatório em `%APPDATA%\primord-overlay\integration-token.txt`. Mantenha-o privado. Envie-o nas rotas `/v1/*` pelo cabeçalho `Authorization: Bearer <token>`. Não há PIN universal nem rota para revelar o token.

Para trocar o token, feche o launcher, remova **somente** `integration-token.txt` e abra o app novamente. Integrações que usavam o token antigo precisarão ser atualizadas.

## Rotas

| Método | Caminho | Autenticação | Resposta |
| --- | --- | --- | --- |
| GET | `/health` | Não | `200`: `{ "service": "primord-overlay", "apiVersion": 1, "authentication": "installation-token" }` |
| GET | `/v1/capabilities` | Sim | `200`: versão, funções `status`, `open-map`, `open-guides` e restrições |
| GET | `/v1/status` | Sim | `200`: estado atual do launcher (campos abaixo) |
| POST | `/v1/commands/open-map` | Sim | `202`: `{ "accepted": true }`; `409`: `{ "accepted": false }` se indisponível |
| POST | `/v1/commands/open-guides` | Sim | `202`: `{ "accepted": true }`; `409`: `{ "accepted": false }` se indisponível |

`/v1/status` contém `gameMode` (booleano), `serverConnected` (booleano), `activeDinoPresent` (booleano ou `null` enquanto não confirmado), `activePosition` (posição ou `null`), `mapVisible`, `minimapVisible`, `guidesVisible` (booleanos) e `activeSpecies` (identificador ou `null`). Dados de dinossauro e posição dependem do que o painel fornece; `null` não significa necessariamente servidor offline.

Os comandos são aceitos apenas quando a sessão do painel está autenticada. Eles abrem janelas existentes do launcher; não controlam o jogo nem executam ações no Painel Primord. Consulte `/v1/capabilities` antes de depender de funções novas.

Erros: `401` para token ausente/incorreto, `403` para origem web não autorizada, `404` para rota inexistente. Todas as respostas são JSON e usam `Cache-Control: no-store`. Se a porta 37621 estiver ocupada, o launcher continua funcional, mas a API não sobe.

## Exemplo: ferramenta local

No PowerShell, substitua o caminho do token pelo perfil Windows que executa o launcher:

```powershell
$tokenPath = Join-Path $env:APPDATA 'primord-overlay\integration-token.txt'
$token = (Get-Content -LiteralPath $tokenPath -Raw).Trim()
$headers = @{ Authorization = "Bearer $token" }
Invoke-RestMethod 'http://127.0.0.1:37621/health'
Invoke-RestMethod 'http://127.0.0.1:37621/v1/capabilities' -Headers $headers
Invoke-RestMethod 'http://127.0.0.1:37621/v1/status' -Headers $headers
Invoke-RestMethod 'http://127.0.0.1:37621/v1/commands/open-map' -Method Post -Headers $headers
```

Não copie o valor do token para logs, repositórios, páginas públicas ou código distribuído. Uma ferramenta instalada no computador pode ler o arquivo do próprio usuário. Um site público **não** deve embutir esse token no JavaScript.

## Integração com site

Por padrão, nenhuma origem de navegador é permitida. Para uma origem HTTPS específica, crie `%APPDATA%\primord-overlay\integration-origins.json` com, por exemplo:

```json
{"allowedOrigins":["https://seu-site.example"]}
```

Reinicie o launcher. A origem deve coincidir exatamente, incluindo esquema e porta. A API responde a `OPTIONS` e permite `Authorization` e `Content-Type` para origens autorizadas. A autorização por origem **não substitui** o token. Para um site hospedado, use um componente local de confiança ou pareamento seguro; o servidor remoto não alcança diretamente `127.0.0.1` do usuário.

