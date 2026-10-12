# Rastro Hub

Launcher para The Isle no Windows com escolha entre Gamer Hause e Primord a cada abertura. As sessões dos dois painéis ficam separadas. A sessão Primord usada nas versões anteriores continua no mesmo perfil local, sem exigir nova conta apenas por causa da mudança de nome.

## Estado das integrações

- **Primord:** painel, minimapa, mapa grande, carcaça e guias continuam dependendo dos dados do site Primord na sessão autenticada.
- **Gamer Hause:** o launcher abre o painel oficial e preserva sua sessão Discord em um perfil separado. Os recursos de mapa, dinossauro ativo, posição e carcaça específicos desse servidor dependem das APIs que o site ainda precisa publicar. O launcher não contorna o login nem inventa dados.

Para implementar a integração Gamer Hause, consulte o [contrato de APIs do Rastro Hub](RASTRO-HUB-API.md). Ele diferencia rotas existentes de rotas propostas. Para ferramentas que rodam no mesmo PC, consulte a [API local](API.md).

## Download

As versões publicadas ficam em [Releases](https://github.com/RuanFCatarino/primord-overlay/releases). Confira o número da versão e o SHA-256 antes de executar. O executável ainda não é assinado digitalmente; Windows ou navegador podem exibir um aviso. Nunca ignore um alerta de malware confirmado.

## Segurança

Cada painel usa sua própria autenticação. Não publique cookies, tokens, localização pessoal ou credenciais em Issues ou capturas. O Rastro Hub é um projeto não oficial, sem afiliação com os responsáveis por Primord Survival ou The Isle.

## Suporte

Use a aba **Issues** e informe a versão do launcher e os passos para reproduzir, sem dados pessoais.
