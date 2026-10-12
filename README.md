# Primord Overlay

Launcher não oficial para acompanhar o painel e o mapa do Primord Survival no Windows.

## Download

Baixe a versão mais recente em [Releases](https://github.com/RuanFCatarino/primord-overlay/releases). Use o **Instalador** para instalar no Windows ou o **Portátil** para executar sem instalação. O pacote ZIP inclui as duas opções e instruções. Não é necessário instalar Electron separadamente.

## Integração com outros projetos

A API local é opcional: o launcher funciona normalmente sem conectar nenhum projeto externo. Os caminhos, autenticação, respostas e exemplos estão em [API.md](API.md). A porta padrão é `http://127.0.0.1:37621`, restrita ao próprio computador. Não exponha o token ou a porta à Internet.

## Sobre o projeto

- O launcher reúne painel, mapa, minimapa e sobreposição em jogo.
- O mapa e as informações ao vivo dependem do que o Painel Primord fornece à sessão autenticada.
- Este é um projeto não oficial, sem afiliação com os responsáveis por Primord Survival ou The Isle.

## Segurança

O executável ainda não possui assinatura digital. O Windows ou o navegador pode mostrar um aviso ao baixá-lo ou abri-lo. Verifique a origem deste repositório e o hash SHA-256 publicado junto a cada versão antes de executar. Nunca ignore um alerta que indique malware confirmado.

## Suporte

Use a aba **Issues** para relatar falhas, informando a versão do launcher e os passos para reproduzir. Não publique senhas, tokens, dados pessoais ou localização em capturas de tela e relatórios.
