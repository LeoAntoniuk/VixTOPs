# VixTOPs — Documentação

Plugin de tops configuráveis para Bukkit/Spigot/Paper, com NPCs e Hologramas via packets (sem dependência de Citizens ou DecentHolograms), suporte a múltiplos plugins de economia e refresh event-driven. Compatível com Minecraft **1.8.X** a **1.21.X**.

---

## Sumário

1. [Visão geral](#1-visão-geral)
2. [Instalação](#2-instalação)
3. [Arquivos de configuração](#3-arquivos-de-configuração)
4. [Comandos](#4-comandos)
5. [Permissões](#5-permissões)
6. [Placeholders internos](#6-placeholders-internos)
7. [Sistema de tops](#7-sistema-de-tops)
8. [NPCs e Hologramas](#8-npcs-e-hologramas)
9. [Integrações](#9-integrações)
10. [Compatibilidade entre versões](#10-compatibilidade-entre-versões)
11. [Solução de problemas](#11-solução-de-problemas)

---

## 1. Visão geral

VixTOPs permite criar **quantos tops quiser** (dinheiro, gems, kills, playtime, qualquer coisa que vire um número), cada um com seu próprio refresh, formatação, ícone no menu, NPC físico no mundo e holograma. Tudo configurável em YAML.

### Características principais

- Multi-top (sem limite) com providers extensíveis (economia, PlaceholderAPI, VixTempoOnline)
- Hook automático para **VixEconomias** (multi-economia), **Vault** e **VixTempoOnline** (tempo online) — todos funcionam simultaneamente
- **4 tipos de display**: NPC (player entity), ARMORSTAND (com equipamento), HEAD (cabeça flutuante) e HOLOGRAM (lista de ranking)
- NPCs e hologramas 100% via packets — não precisa Citizens, HolographicDisplays ou DecentHolograms
- Atualização event-driven: o holograma só re-renderiza quando o cache do top muda
- Cache de skin (TTL 1h) — não bate na Mojang API toda hora
- Bypass por top com permissão dinâmica: `vixtops.<top>.bypass`
- Suporte completo a **LuckPerms** (bypass funciona até pra jogadores offline)
- PlaceholderAPI integrado (em hologramas, lores e menus)
- **Suporte completo a `&`, `&#RRGGBB` (hex) e MiniMessage** (`<gradient:#FF0000:#0000FF>...</gradient>`) — formatação **per-viewer** version-aware (1.16+ vê hex/gradient real, 1.8-1.15 cai pra cor legacy mais próxima)
- MaterialAdapter resolve `PLAYER_HEAD` ↔ `SKULL_ITEM:3`, `BLACK_STAINED_GLASS_PANE` ↔ `STAINED_GLASS_PANE:15`, etc.
- Formato de valor configurável por-top (`LETTER` ou `DECIMAL`), com sufixos e separadores em `config.yml`
- Per-NPC: `y-offset` e `line-spacing` do holograma podem ser sobrescritos individualmente
- **Aliases de subcomandos configuráveis** em `commands.yml` (traduzir, abreviar, customizar)
- Save debounced (5s) — spawnar muitos NPCs não causa I/O na main thread
- Menu único e configurável com item "Sua posição" dinâmico (cabeça do player + lore com posição em cada top)

---

## 2. Instalação

A instalação é feita **via Vix Loader** com base na licença do cliente. Não é necessário fazer upload manual do `.jar` no servidor — o loader baixa o plugin sob demanda em memória.

### 2.1 Fluxo geral

```
┌───────────────────┐         ┌────────────────────┐         ┌─────────────────┐
│  Servidor Bukkit  │         │ www.vixplugins.com │         │   Sua licença   │
│                   │         │                    │         │                 │
│  + Vix-API        │ ───1──► │                    │         │                 │
│    plugin         │ ◄───2── │   Envia o plugin   │ ◄──3─── │  Valida quais   │
│                   │         │      VixTOPs       │         │  plugins libera │
│  Carrega em RAM   │         │                    │         │                 │
└───────────────────┘         └────────────────────┘         └─────────────────┘

1. API autentica com a licença
2. Servidor recebe o plugin
```

### 2.2 Pré-requisitos no servidor

1. **Vix-API** instalado em `plugins/` (baixa uma vez)
2. **Licença** configurada (adicionar o IP da sua máquina no painel de plugins)
3. **PacketEvents** instalado em `plugins/` (download em <https://modrinth.com/plugin/packetevents>)
4. **Java 8+** (recomendado Java 17/21 pra servidores modernos)
5. **VixEconomias** ou **Vault** — pra tops de economia (opcional, mas necessário para o top `money` padrão)
6. **PlaceholderAPI** — opcional, só se for usar placeholders externos
7. **LuckPerms** — opcional, mas recomendado pra que `vixtops.<top>.bypass` funcione com jogadores offline

### 2.3 Ativar o VixTOPs na sua conta

1. Acesse o painel da Vix Plugins
2. Faça login com sua conta licenciada
3. Localize o plugin **VixTOPs** na sua lista
4. Ative-o pro servidor/licença desejado

### 2.4 Primeira execução

Quando o servidor iniciar com o loader configurado:

1. O loader contata o website e verifica a licença
2. O `VixTOPs` é baixado em memória
3. O `onEnable()` do VixTOPs roda normalmente
4. Os arquivos de configuração são gerados em `plugins/VixTOPs/`:
   - `config.yml`
   - `messages.yml`
   - `tops.yml`
   - `menus.yml`
   - `commands.yml`
   - `npcs.yml` (atualizado em runtime quando você criar NPCs)
5. Console exibe: `[VixTOPs] Enabling VixTOPs v1.0.0`

### 2.5 Atualizar o plugin

Não é necessário substituir o jar manualmente. Quando uma nova versão é publicada no website:
- Reinicie o servidor (a API baixa a versão mais recente)
- Ou use `/top reload` se a mudança não exigir restart

As configs em `plugins/VixTOPs/` **não** são sobrescritas em updates — só são adicionados campos novos.

### 2.6 Desinstalar

- No painel Vix, desative o VixTOPs pra essa licença
- Reinicie o servidor
- (Opcional) Apague `plugins/VixTOPs/` se não quiser manter configs

### 2.7 Verificar que está rodando

Use `/top` no console ou in-game. Se exibir o menu com os tops configurados, o plugin foi carregado com sucesso.

Outra forma: `/plugins` mostra o `VixTOPs` na lista (em verde quando carregado).

### 2.8 Solução de problemas comuns na instalação

| Sintoma no console | Causa provável | Solução |
|---|---|---|
| `License invalid` ou `Unauthorized` | Licença expirada ou sem acesso ao plugin | Cheque o painel da Vix, renove/ative |
| `Failed to download` | Sem conectividade ao website | Verifique firewall / DNS / proxy do servidor |
| `Missing depend: packetevents` | PacketEvents não está instalado | Baixe e coloque em `plugins/` |
| `Nenhum plugin de economia encontrado!` | Sem Vault e sem VixEconomias | Instale um deles ou remova os tops de economia em `tops.yml` |
| `Skin não encontrada para NPC` | Servidor offline-mode ou rate-limit Mojang | Esperado em servidores cracked; tente skin por nome em vez de `%player%` |

---

## 3. Arquivos de configuração

### `config.yml`

Configurações gerais: prefix das mensagens, intervalo default de atualização dos tops, distância de render dos NPCs, prioridade dos plugins de economia, formato de números (K/M/B/T...), entradas vazias do top, e defaults dos hologramas (linhas padrão, espaçamento).

```yaml
settings:
  prefix: "&5&lVix&f&lTOPs &7"
  default-update-interval: 60       # segundos
  npc-render-distance: 48.0         # blocos
  economy-priority:                 # ordem de detecção
    - VixEconomias
    - Vault
  show-empty-entries: true          # mostrar slots vazios no top?
  empty-name: "&c&oNinguém"
  empty-value: "&c&o0"

# Formatação por LETTER: índice 0 = < 1000, 1 = 10^3 (K), 2 = 10^6 (M), 3 = 10^9 (B)...
# Você pode adicionar quantos níveis quiser.
letters:
  - ""    # 0 — < 1.000
  - "K"   # 1 — 10^3
  - "M"   # 2 — 10^6
  - "B"   # 3 — 10^9
  - "T"   # 4 — 10^12
  - "Q"   # 5 — 10^15
  # ... continue indefinidamente

# Padrão para formato DECIMAL e separadores
decimal-pattern: "#,##0.00"
decimal:
  thousands-separator: "."
  decimal-separator: ","

holograms:
  npc-default-lines:                # linhas acima de cada NPC criado
    - "%top%"
    - "&f%player% - &d#%position%"
    - "&a%value%"
  line-spacing: 0.27                # espaço vertical entre linhas (override por-NPC possível)
  npc-y-offset: 2.1                 # altura inicial acima do NPC (override por-NPC possível)
  list-default:                     # holograma de lista padrão
    size: 10
    entry-format: "&d#%position% &f%player% &8- &b%value%"
    lines:
      - "%top%"
      - ""
      - "%entries%"
      - ""
      - "&7Atualizado a cada %update_interval%s"

# Defaults dos NPCs de armor stand (criados com /top setar <top> armor <pos>)
armorstand-default:
  small: false
  arms: true
  base-plate: false
  equipment:
    helmet: "%player%"              # cabeça do jogador da posição (ou material fixo)
    chestplate: ""                  # vazio = sem item
    leggings: ""
    boots: ""
    main-hand: ""
    off-hand: ""

# Defaults dos NPCs de cabeça flutuante (criados com /top setar <top> head <pos>)
head-default:
  small: false
  equipment:
    helmet: "%player%"              # cabeça do jogador da posição
    # outros slots vazios por padrão — adicione se quiser "armadura flutuante"
```

### `tops.yml`

Define cada top configurado. Estrutura:

```yaml
tops:
  money:
    display-name: "&6&lTop &e&lDinheiro"
    provider: economy           # economy | placeholder
    economy: auto               # auto | VixEconomias | Vault
    vix-economy: coins          # id da economia no VixEconomias
    update-interval: 60         # segundos (sobrescreve default global)
    size: 10                    # quantos jogadores no top
    open-command: "money top"   # comando executado ao clicar no item / interagir com NPC
    format: LETTER              # LETTER (1,5M) | DECIMAL (1.500.000,00) — default LETTER
    value-format: "&a$ {value}"
    ascending: false            # ordenar crescente? (default false = decrescente)
    icon:
      material: GOLD_INGOT
      name: "&6&lTop &e&lDinheiro"
      lore:
        - "&7Os jogadores mais ricos:"
        - ""
        - "%entries%"           # expande no preview do ranking
        - ""
        - "&eClique para abrir!"
      preview:
        size: 5
        format: "&7#%position% &f%player% &8- &a$ %value%"
        empty-format: "&7#%position% &8&oVazio"
        show-empty: false
```

Tipos de `provider`:

- **`economy`**: usa o hook de economia (`Vault` ou `VixEconomias`).
  - Para VixEconomias multi-economia, defina `vix-economy: coins` (ou `gems`, `tokens`, etc.)
- **`placeholder`**: pega o valor de um placeholder do PlaceholderAPI.
  - Use `placeholder: "%statistic_player_kills%"`. Precisa PAPI instalado.
- **`tempoonline`** (aliases: `playtime`, `tempo`): pega o top de tempo online do **VixTempoOnline**.
  - Valor é em **segundos** internamente. O `format:` é forçado para `TIME` automaticamente — `{value}` sai como `"1 ano, 2 meses e 3 dias"`, `"3 horas e 25 minutos"`, `"5 minutos e 30 segundos"`, etc. (zeros omitidos).
  - Requer VixTempoOnline instalado; se ausente, retorna lista vazia silenciosamente.

Tipos de `format` (por-top):

- **`LETTER`** (default): formato curto com sufixos. Ex: `1,5K`, `2,3M`, `999B`.
- **`DECIMAL`**: formato completo com separadores. Ex: `1.500,00`, `1.234.567,89`.
- **`TIME`**: trata o valor como **segundos** e formata como duração em português. Ex: `1 ano, 2 meses, 3 semanas e 4 dias`, ou `3 horas, 15 minutos e 20 segundos`. Componentes: anos, meses, semanas, dias, horas, minutos, segundos — zeros omitidos automaticamente.

Você pode misturar livremente — cada top configura o seu. Os sufixos (K, M, B, T...) e separadores são globais e definidos em [config.yml](#3-arquivos-de-configuração) (`letters`, `decimal-pattern`, `decimal.*`).

> **Observação**: quando `provider: tempoonline`, o `format:` é **automaticamente forçado para TIME** — independente do que o admin colocou. Não faz sentido formatar segundos como `1,5K`.

Tokens disponíveis em `value-format`:

| Token | Significado |
|---|---|
| `{value}` | Valor renderizado pelo `format` do top (LETTER ou DECIMAL) |
| `{value_letter}` | Sempre LETTER (`1,5M`) — ignora o `format` do top |
| `{value_decimal}` | Sempre DECIMAL (`1.500.000,00`) — ignora o `format` do top |
| `{value_short}` | Alias de `{value_letter}` |
| `{value_plain}` | Alias de `{value_decimal}` |
| `{value_formatted}` | Alias de `{value_letter}` (compatibilidade) |

Use `{value_decimal}` + `{value_letter}` juntos pra mostrar os dois ao mesmo tempo:
```yaml
value-format: "&a$ {value_decimal} &8(&7{value_letter}&8)"
# → "$1.500.000,00 (1,5M)"
```

### `messages.yml`

Todas as mensagens enviadas pro jogador. Suporta **três formatos misturáveis** em qualquer string do plugin (messages.yml, tops.yml, menus.yml, npcs.yml, holograms — tudo):

- **Códigos legacy**: `&a`, `&l`, `&r`, `&n`, `&o`, etc.
- **Hex colors**: `&#FF5733` (renderizado em clientes 1.16+, aproximado em legacy)
- **MiniMessage**: `<red>texto</red>`, `<gradient:#FF0000:#0000FF>arco-íris</gradient>`, `<bold><italic>...</italic></bold>`, etc.

Você pode misturar à vontade. Exemplo:
```yaml
no-permission: "&c<bold>Sem permissão!</bold> <gradient:#FFAA00:#FF0000>Você não pode fazer isso</gradient>"
```

Tags úteis do MiniMessage:
| Tag | Função |
|---|---|
| `<red>`, `<green>`, ..., `<#RRGGBB>` | Cores |
| `<bold>`, `<italic>`, `<underlined>`, `<strikethrough>`, `<obfuscated>` | Decorações |
| `<gradient:#XXXXXX:#YYYYYY>...</gradient>` | Gradient entre cores |
| `<rainbow>...</rainbow>` | Arco-íris |
| `<reset>` | Reseta formatação |

Documentação completa do MiniMessage: <https://docs.advntr.dev/minimessage/format.html>

Outros tokens:
- `%prefix%` é substituído automaticamente em messages.yml
- Placeholders por mensagem específica (documentados em cada bloco)
- PlaceholderAPI (`%vixeconomias_coins_amount%`, etc.) funciona junto com qualquer formato acima

### `menus.yml`

Configura o **único** menu do plugin (`/top`):

```yaml
main-menu:
  title: "&5Jogadores mais ricos"
  size: 27                                # múltiplo de 9 (9, 18, 27, 36, 45, 54)
  fill:
    enabled: false                        # item de fundo nos slots vazios
    material: BLACK_STAINED_GLASS_PANE
    name: " "

  # Slots fixos para cada top (chave = key em tops.yml).
  # Se o top não tem slot definido aqui, vai pro próximo slot livre.
  slots:
    money: 13
    # gems: 11

  # Item dinâmico "Sua posição" — mostra a posição do viewer em CADA top configurado.
  player-position-item:
    enabled: true
    slot: 4
    material: PLAYER_HEAD                 # PLAYER_HEAD = cabeça do jogador que abriu o menu
    name: "&d&lSua posição"
    lore:
      - "&7Suas posições nos tops:"
      - ""
      - "%positions%"                     # ← placeholder: gera 1 linha por top
      - ""
      - "&8Clique em um top para abrir."
    position-line: "&8• &f%top% &8- &e#%position% &7(&a%value%&7)"
    not-in-top: "&8• &f%top% &8- &7&ofora do top"
```

**Clicar num item de top no menu** dispara o `open-command` daquele top (definido em [tops.yml](#3-arquivos-de-configuração)). O VixTOPs não exibe o ranking — quem mostra é o comando que você configurar.

### `commands.yml`

Configura **aliases dos subcomandos** e **dos tipos do `/top setar`**. Útil pra traduzir para sua língua ou simplesmente abreviar.

> Observação: aliases do **comando principal** (`/top`, `/tops`, `/vixtop`, `/vixtops`) são lidos do `plugin.yml` pelo Bukkit no startup e não podem ser alterados em runtime.

```yaml
subcommands:
  setar:
    aliases: [definir, set, criar]
  remover:
    aliases: [remove, deletar, delete, apagar]
  reload:
    aliases: [rl, recarregar]
  help:
    aliases: ["?", ajuda]

setar-types:
  armor:
    aliases: [armorstand, as]
  head:
    aliases: [cabeca, cabeça, skull]
  holo:
    aliases: [hologram, holograma, lista, list]
```

A **chave** da entrada (`setar`, `remover`, etc.) também é tratada como alias — você não precisa repetir. Aliases são **case-insensitive** e funcionam tanto no comando quanto no tab-complete. Use `/top reload` pra aplicar mudanças sem restart.

### `npcs.yml`

**Gerado e atualizado automaticamente** pelo plugin quando você usa `/top setar` ou `/top remover`. Você pode editar manualmente as `lines`, `hologram` ou outras propriedades de NPCs/hologramas existentes — depois faça `/top reload`.

**Campos opcionais aplicáveis a qualquer tipo:**
- `y-offset: 2.5` — sobrescreve `holograms.npc-y-offset` (altura do holograma acima desse NPC específico)
- `line-spacing: 0.30` — sobrescreve `holograms.line-spacing` (espaço vertical entre linhas desse holograma)

Os defaults globais vêm de [config.yml](#3-arquivos-de-configuração). Adicione esses campos só se quiser customizar **esse** NPC específico.

**Exemplo de NPC (player entity):**
```yaml
npcs:
  money:
    "1":                          # posição #1 do top
      type: NPC
      world: world
      x: 100.5
      y: 64.0
      z: 200.5
      yaw: 0.0
      pitch: 0.0
      skin: "%player%"            # ou nome fixo (ex: Notch)
      y-offset: 2.5               # opcional
      line-spacing: 0.30          # opcional
      hologram:
        - "&6&lTop &e#%position%"
        - "&f%player%"
        - "&a%value%"
```

**Exemplo de ARMORSTAND (criado com `/top setar <top> armor <pos>`):**
```yaml
npcs:
  money:
    "1":
      type: ARMORSTAND
      world: world
      x: 100.5
      y: 64.0
      z: 200.5
      yaw: 0.0
      pitch: 0.0
      small: false
      arms: true
      base-plate: false
      equipment:
        helmet: "%player%"
        chestplate: DIAMOND_CHESTPLATE
        leggings: ""
        boots: ""
        main-hand: DIAMOND_SWORD
        off-hand: SHIELD
      hologram:
        - "&6&lTop &e#%position%"
        - "&f%player%"
        - "&a%value%"
```

**Exemplo de HEAD (criado com `/top setar <top> head <pos>`):**
```yaml
npcs:
  money:
    "1":
      type: HEAD
      world: world
      x: 100.5
      y: 64.0
      z: 200.5
      yaw: 0.0
      pitch: 0.0
      small: false
      equipment:
        helmet: "%player%"        # cabeça do jogador (o resto fica vazio)
      hologram:
        - "&6&lTop &e#%position%"
        - "&f%player%"
        - "&a%value%"
```

**Exemplo de HOLOGRAM lista (criado com `/top setar <top> holo`):**
```yaml
npcs:
  money:
    list:
      type: HOLOGRAM
      world: world
      x: 100.5
      y: 64.0
      z: 200.5
      size: 10
      entry-format: "&e#%position% &f%player% &8- &a%value%"
      lines:
        - "&6&lTop Dinheiro"
        - ""
        - "%entries%"
        - ""
        - "&7Atualizado a cada %update_interval%s"
```

---

## 4. Comandos

| Comando | Descrição | Permissão |
|---|---|---|
| `/top` | Abre o menu principal com todos os tops | `vixtops.command.top` |
| `/top <topkey>` | Atalho — executa direto o `open-command` daquele top | `vixtops.command.top` |
| `/top setar <top> <posição>` | Cria um NPC (player entity) na posição N | `vixtops.command.setar` |
| `/top setar <top> armor <posição> [small]` | Cria um NPC de **armor stand** na posição N | `vixtops.command.setar` |
| `/top setar <top> head <posição> [small]` | Cria um **floating head** (cabeça flutuante) na posição N | `vixtops.command.setar` |
| `/top setar <top> holo` | Cria um holograma de **lista** (ranking flutuante completo) | `vixtops.command.setar` |
| `/top remover <top> <posição>` | Remove um NPC/armor stand específico do top | `vixtops.command.remover` |
| `/top remover <top> holo` | Remove o holograma de lista do top | `vixtops.command.remover` |
| `/top remover <top>` | Atalho — remove o holograma de lista do top | `vixtops.command.remover` |
| `/top reload` | Recarrega todas as configurações | `vixtops.admin` |
| `/top help` | Lista os comandos | `vixtops.command.top` |

### Aliases do comando

- Comando principal: `/tops`, `/vixtop`, `/vixtops` (além de `/top`) — fixos via plugin.yml
- Subcomandos e tipos do `setar` — **configuráveis** em [commands.yml](#3-arquivos-de-configuração). Defaults:
  - `setar` aceita: `setar`, `definir`, `set`, `criar`
  - `remover` aceita: `remover`, `remove`, `deletar`, `delete`, `apagar`
  - `reload` aceita: `reload`, `rl`, `recarregar`
  - `help` aceita: `help`, `?`, `ajuda`
  - `armor` aceita: `armor`, `armorstand`, `as`
  - `head` aceita: `head`, `cabeca`, `cabeça`, `skull`
  - `holo` aceita: `holo`, `hologram`, `holograma`, `lista`, `list`

### Comportamento de clique

- **Clique no item de um top no menu** → executa o `open-command` daquele top (definido em `tops.yml`)
- **Clique direito num NPC, ArmorStand ou HEAD** → executa o `open-command` do top daquele NPC

O VixTOPs **não tem menu próprio de visualização do top** — quem mostra o ranking é o plugin/comando que você apontar em `open-command` (ex: `money top` do VixEconomias, ou um comando seu).

---

## 5. Permissões

### Permissões fixas

| Permissão | Default | Descrição |
|---|---|---|
| `vixtops.admin` | `op` | Acesso a todos os comandos administrativos |
| `vixtops.command.top` | `true` | Usar `/top` |
| `vixtops.command.setar` | `op` | Usar `/top setar` |
| `vixtops.command.remover` | `op` | Usar `/top remover` |

### Permissões dinâmicas (uma por top)

| Permissão | Default | Descrição |
|---|---|---|
| `vixtops.<topkey>.bypass` | `false` | Oculta o jogador desse top específico |

Exemplos:
- `vixtops.money.bypass` — esconde do top dinheiro
- `vixtops.kills.bypass` — esconde do top kills
- `vixtops.gems.bypass` — esconde do top gems

Quem tiver a permissão **não aparece** no `/top`, nem nos NPCs, nem nos hologramas.

> **Importante**: para que o bypass funcione com jogadores **offline**, instale **LuckPerms**. Sem ele, o plugin só consegue checar permissões de jogadores online. Aviso disso é logado no console no startup.

---

## 6. Placeholders internos

Disponíveis em hologramas, lores de itens, mensagens e `preview.format`/`entry-format` dos hologramas:

| Placeholder | Significado |
|---|---|
| `%position%` | Posição no top (1, 2, 3...) |
| `%player%` | Nome do jogador na posição |
| `%value%` | Valor formatado pelo `value-format` do top |
| `%value_formatted%` | Valor com sufixos curtos (15K, 1.2M) |
| `%value_raw%` | Valor cru sem formatação |
| `%top%` | Display-name do top (com cores) |
| `%top_key%` | ID interno do top (ex: `money`) |
| `%size%` | Quantidade configurada no top |
| `%update_interval%` | Intervalo de atualização em segundos |
| `%entries%` | (apenas em hologramas/lores) Expande nas N linhas do ranking |
| `%positions%` | (apenas no item "Sua posição") Lista posição do viewer em cada top |

Placeholders do **PlaceholderAPI** também funcionam em todos esses lugares se o plugin estiver instalado.

---

## 7. Sistema de tops

### Atualização (event-driven)

Cada top tem seu próprio `update-interval` em segundos (default global em `config.yml > settings.default-update-interval`). A cada intervalo:

1. Provider coleta os dados (async): `EconomyTopProvider` chama `VixEconomiasAPI.getTop()` (eficiente, já vem ordenado) ou itera `Bukkit.getOfflinePlayers()` (fallback Vault)
2. Permissão `vixtops.<top>.bypass` é aplicada (LuckPerms se disponível, senão online-only)
3. Lista é ordenada e cortada para o `size` configurado
4. Cache é atualizado e a **version** do top é incrementada
5. NPCs e hologramas detectam a mudança no próximo tick (0.5s) e re-renderizam

### Cache

Os tops mantêm cache em memória — não há banco de dados próprio. Os dados sempre vêm das APIs (VixEconomias, Vault, PlaceholderAPI) quando o cache atualiza.

### Bypass

A permissão `vixtops.<top>.bypass` é checada **no momento do refresh** (não em cada render). Com LuckPerms, funciona pra jogadores offline. Sem LuckPerms, só pra online — fica avisado no console.

---

## 8. NPCs e Hologramas

### Tipos disponíveis

- **NPC** (player entity): jogador físico no mundo (posição #N do top). Skin = do jogador na posição (ou skin fixa). Tem holograma acima da cabeça.
- **ARMORSTAND**: armor stand visível com armaduras e itens configuráveis. Por padrão usa a cabeça do jogador da posição. Tem holograma acima da cabeça também.
- **HEAD**: cabeça flutuante (armor stand invisível com só o helmet = cabeça do jogador). Visualmente é só uma cabeça no ar. Tem holograma acima.
- **HOLOGRAM (lista)**: ranking flutuante completo. Sem entidade visível, só texto.

### Criar um NPC (player entity)

```
/top setar money 1
```
Spawna o NPC da posição #1 do top `money` na sua localização atual. A skin é puxada automaticamente da Mojang API (cacheada 1h). Se o servidor é offline-mode, o plugin faz fallback de UUID → nome.

Defaults usados (configuráveis em `config.yml > holograms.npc-default-lines`):
```yaml
- "%top%"
- "&f%player% - &d#%position%"
- "&a%value%"
```

### Criar uma Cabeça Flutuante (HEAD)

```
/top setar money head 1
/top setar money head 1 small
```
Spawna uma cabeça flutuando no ar (armor stand invisível com helmet = skull do jogador da posição). Configurável em `config.yml > head-default`:

```yaml
head-default:
  small: false
  equipment:
    helmet: "%player%"   # cabeça do jogador da posição (atualiza com o top)
```

Você pode opcionalmente equipar outros slots — eles aparecem como armadura flutuante (corpo continua invisível).

### Criar um NPC de ArmorStand

```
/top setar money armor 1
/top setar money armor 1 small
```
Spawna um armor stand na sua localização. Configurável em [npcs.yml](#3-arquivos-de-configuração) (gerado em runtime) e defaults em `config.yml > armorstand-default`:

```yaml
armorstand-default:
  small: false           # use "small" no comando ou true aqui para versão pequena
  arms: true             # mostrar braços (necessário para itens nas mãos)
  base-plate: false      # mostrar a base (placa)
  equipment:
    helmet: "%player%"   # cabeça do jogador da posição (ou material como DIAMOND_HELMET)
    chestplate: ""       # vazio = sem item
    leggings: ""
    boots: ""
    main-hand: ""        # ex: DIAMOND_SWORD
    off-hand: ""         # ex: SHIELD
```

**Sobre tamanhos:** Minecraft só tem 2 (`small` e normal). "Grande/médio" não existem nativamente — esses são os únicos.

**Equipamento:**
- Use `%player%` no helmet para a cabeça do jogador da posição (atualizada automaticamente quando o top muda).
- Use o nome do material para itens fixos (ex: `DIAMOND_HELMET`, `IRON_CHESTPLATE`, `SHIELD`, `DIAMOND_SWORD`).
- Deixe vazio (`""`) para não colocar item naquele slot.

O armor stand tem o mesmo sistema de holograma acima da cabeça que o NPC tradicional.

### Criar um holograma de lista

```
/top setar money holo
```
Spawna um holograma completo do ranking. Defaults em `config.yml > holograms.list-default`.

### Editar um NPC/holograma

Edite `plugins/VixTOPs/npcs.yml` manualmente e execute `/top reload`. As mudanças globais em `config.yml > holograms` só se aplicam a **novos** NPCs (pra não sobrescrever customizações por-NPC).

### Renderização

- NPCs/hologramas só aparecem dentro de `settings.npc-render-distance` blocos
- O **mesmo NPC** aparece pra todos os jogadores próximos (não é "pessoal")
- Re-render acontece **somente quando o top atualiza** (não há polling fixo)
- Cache do `UserProfile` por-NPC: a skin é fetched uma vez por refresh, não a cada spawn

### Skin

- Por default, `skin: "%player%"` no `npcs.yml` → usa a skin do jogador da posição
- Pode trocar para um nome fixo: `skin: "Notch"`
- Cache de 1h por nome/UUID
- Em **servidor offline-mode**: tenta UUID (vai falhar) e cai pro fetch por nome (funciona)

---

## 9. Integrações

### VixEconomias

Detectado automaticamente. Suporta multi-economia: defina `vix-economy: <id>` em cada top. Sem isso, usa a primeira economia registrada.

O VixTOPs usa o `getTop()` nativo do VixEconomias — não itera jogadores manualmente. Isso é **muito** mais eficiente.

### Vault

Detectado automaticamente. Single-economy (sem `vix-economy`). Top é montado iterando `Bukkit.getOfflinePlayers()`.

### Prioridade

`config.yml > settings.economy-priority` define qual hook é usado quando `economy: auto`:
```yaml
economy-priority:
  - VixEconomias
  - Vault
```

### PlaceholderAPI

Detectado automaticamente. Quando presente:
- Placeholders externos (`%vixeconomias_coins_amount%`, `%statistic_player_kills%`, etc.) funcionam em hologramas, lores e mensagens
- Tops com `provider: placeholder` ficam disponíveis

### LuckPerms

Detectado automaticamente. Quando presente, permite checar a permissão `vixtops.<top>.bypass` para jogadores **offline**. Sem ele, fallback para Bukkit (só online).

### VixTempoOnline

Detectado automaticamente via reflection (sem dependência de compile-time). Quando presente:
- Top com `provider: tempoonline` fica disponível
- Usa o `fetchTop()` nativo da API do VixTempoOnline (eficiente, vai direto no banco com cache)
- Funciona simultaneamente com VixEconomias e Vault — um top de dinheiro e outro de tempo online no mesmo menu

### PacketEvents

**Obrigatório** (`depend`). É a biblioteca de packets cross-version que faz os NPCs e hologramas funcionarem de 1.8 a 1.21 sem precisar de Citizens ou DecentHolograms.

---

## 10. Compatibilidade entre versões

Testado de **1.8.X** até **1.21.X**.

### O que se adapta automaticamente

- **Materials** no menu: `PLAYER_HEAD` ↔ `SKULL_ITEM:3`, `BLACK_STAINED_GLASS_PANE` ↔ `STAINED_GLASS_PANE:15`, etc.
- **Hex colors e gradients** (`&#RRGGBB` e MiniMessage): renderizados exatos em clientes 1.16+; em 1.8-1.15 são automaticamente **downgraded para a cor legacy mais próxima** (não some, vira `§c` ou `§e` aproximado)
- **MiniMessage**: detectado e parseado automaticamente; serializa pra legacy com a flavor certa por versão do cliente
- **`SkullMeta.setOwningPlayer`** ↔ `setOwner(String)` via reflection
- **Packets de NPC/holograma**: PacketEvents resolve os índices de metadata por versão do cliente
- **`Bukkit.getOnlinePlayers()`** array (1.8) ↔ Collection (modernos) — funciona pelo for-each
- **`WrapperPlayServerEntityEquipment`**: PE serializa per-slot em 1.8-1.15 e em batch em 1.16+

### O que sempre funciona

- Códigos de cor clássicos: `&a`, `&l`, `&r`, etc.
- Comandos, eventos, listeners, scheduler
- Hooks de Vault, VixEconomias, PlaceholderAPI, LuckPerms
- Save assíncrono

### Limitações do cliente (não dá pra contornar no plugin)

- Cliente 1.8 não renderiza hex colors (limitação do protocolo) — o plugin faz downgrade automático pra cor legacy mais próxima
- Cliente 1.8 não tem nametag invisível por padrão — o plugin usa color codes pra mascarar o nome do NPC (player entity)
- Skin em servidor offline-mode depende de o NICK do jogador existir na Mojang
- Click/hover events do MiniMessage (`<click:...>`, `<hover:...>`) são perdidos no downgrade pra legacy (Bukkit `sendMessage` legacy não suporta)

---

## 11. Solução de problemas

### Em caso de erro, verifique o `console` do servidor.

As mensagens importantes começam com `[VixTOPs]`. Erros comuns:

| Sintoma | Causa | Solução |
|---|---|---|
| NPC aparece com skin Steve | Servidor offline-mode + nick não existe na Mojang | Use `skin: "<nick_real>"` no `npcs.yml` em vez de `%player%` |
| `[Skin] Mojang retornou 429 (rate-limit)` | Muitos NPCs spawnando ao mesmo tempo | Espere alguns minutos; o cache evita re-fetch |
| Holograma vazio | Top sem dados (zero jogadores ou todos com bypass) | Cheque cache do top via `/top reload`, valide o hook de economia |
| `Top 'X' não existe em tops.yml mas há NPCs em npcs.yml` | Você removeu o top mas os NPCs ainda referenciam ele | Remova manualmente do `npcs.yml` ou use `/top remover` |
| Bypass não funciona para player offline | LuckPerms não instalado | Instale LuckPerms |
| Hologram mostra "ArmorStand" | Não deveria acontecer com versão atual; report bug | Verifique no console se está rodando a versão correta |
| `Skin não encontrada para NPC` | Servidor offline + nick não-Mojang OU Mojang API fora do ar | Use skin fixa em `npcs.yml` |
| NPCs não atualizam | Top está parado | `/top reload`, verifique se o `update-interval` está razoável |
| NPC com tag flutuante de nick aleatório | Bug — não deveria acontecer | Report; o plugin usa color codes para mascarar |

### Logs úteis pra debug

- `[VixTOPs] Hook de economia ativo: VixEconomias` — confirma o hook
- `[VixTOPs] LuckPerms detectado — checagem de bypass para jogadores offline habilitada.`
- `[VixTOPs] Tops carregados: N`
- `[VixTOPs] NPCs/Hologramas carregados: N`

### Performance

O plugin foi otimizado pra rodar bem em servidores grandes:
- Refresh dos hologramas é **event-driven** (só quando o cache do top muda)
- NPCs/hologramas indexados por mundo (só itera os do mundo do player)
- Configs cacheados em memória (sem reads de YAML em loops)
- Save de `npcs.yml` debounced e async
- Cache de `UserProfile` por NPC (não re-fetch skin a cada spawn)
- TTL no cache de skin (1h)

Se ainda assim notar lag, suspeite primeiro de plugins externos (Citizens, DecentHolograms, HolographicDisplays — você **não** precisa de nenhum desses).

---

## Suporte

- Painel: <https://www.vixplugins.com>
- Logs sempre começam com `[VixTOPs]` no console
- Cole o log completo ao reportar problema (não só a primeira linha)
