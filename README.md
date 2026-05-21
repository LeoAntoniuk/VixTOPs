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

- Multi-top (sem limite) com providers extensíveis (economia ou PlaceholderAPI)
- Hook automático para **VixEconomias** (multi-economia) e **Vault** com prioridade configurável
- NPCs e hologramas 100% via packets — não precisa Citizens, HolographicDisplays ou DecentHolograms
- Holograma de NPC (acima da cabeça) **e** holograma de lista (ranking flutuante)
- Atualização event-driven: o holograma só re-renderiza quando o cache do top muda
- Cache de skin (TTL 1h) — não bate na Mojang API toda hora
- Bypass por top com permissão dinâmica: `vixtops.<top>.bypass`
- Suporte completo a **LuckPerms** (bypass funciona até pra jogadores offline)
- PlaceholderAPI integrado (em hologramas, lores e menus)
- Hex colors `&#RRGGBB` (1.16+) com fallback automático
- MaterialAdapter resolve `PLAYER_HEAD` ↔ `SKULL_ITEM:3`, `BLACK_STAINED_GLASS_PANE` ↔ `STAINED_GLASS_PANE:15`, etc.
- Save debounced (5s) — spawnar muitos NPCs não causa I/O na main thread
- Menu único e configurável com item "Sua posição" dinâmico

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
  number-format:
    enabled: true
    suffixes:
      - { from: 1000, suffix: "K" }
      - { from: 1000000, suffix: "M" }
      # ... até decilhões
  show-empty-entries: true          # mostrar slots vazios no top?
  empty-name: "&c&oNinguém"
  empty-value: "&c&o0"

holograms:
  npc-default-lines:                # linhas acima de cada NPC criado
    - "%top%"
    - "&f%player% - &d#%position%"
    - "&a%value%"
  line-spacing: 0.27                # espaço vertical entre linhas
  npc-y-offset: 2.1                 # altura inicial acima do NPC
  list-default:                     # holograma de lista padrão
    size: 10
    entry-format: "&d#%position% &f%player% &8- &b%value%"
    lines:
      - "%top%"
      - ""
      - "%entries%"
      - ""
      - "&7Atualizado a cada %update_interval%s"
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
    value-format: "&a$ {value_formatted}"
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

### `messages.yml`

Todas as mensagens enviadas pro jogador. Suporta:
- Códigos de cor clássicos: `&a`, `&l`, `&r`, etc.
- Hex colors: `&#FF5733` (1.16+)
- `%prefix%` é substituído automaticamente
- Placeholders por mensagem específica

### `menus.yml`

Configura o menu único do `/top`:
- Tamanho, título, slots dos itens de cada top
- Filler (item de fundo)
- Item dinâmico "Sua posição" (mostra a posição do jogador em **cada** top na lore)

### `npcs.yml`

**Gerado e atualizado automaticamente** pelo plugin quando você usa `/top setar` ou `/top remover`. Você pode editar manualmente as `lines`, `hologram` ou outras propriedades de NPCs/hologramas existentes — depois faça `/top reload`.

Estrutura:
```yaml
npcs:
  money:
    "1":                          # NPC da posição #1
      type: NPC
      world: world
      x: 100.5
      y: 64.0
      z: 200.5
      yaw: 0.0
      pitch: 0.0
      skin: "%player%"            # ou nome fixo (ex: Notch)
      hologram:
        - "&6&lTop &e#%position%"
        - "&f%player%"
        - "&a%value%"
    list:                         # holograma de lista
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
| `/top setar <top> <posição>` | Cria um NPC do top na posição N (ex: 1, 2, 3...) | `vixtops.command.setar` |
| `/top setar <top> holo` | Cria um holograma de **lista** (ranking flutuante completo) | `vixtops.command.setar` |
| `/top remover <top> <posição>` | Remove um NPC específico do top | `vixtops.command.remover` |
| `/top remover <top> holo` | Remove o holograma de lista do top | `vixtops.command.remover` |
| `/top remover <top>` | Atalho — remove o holograma de lista do top | `vixtops.command.remover` |
| `/top reload` | Recarrega todas as configurações | `vixtops.admin` |
| `/top help` | Lista os comandos | `vixtops.command.top` |

### Aliases do comando

`/tops`, `/vixtop`, `/vixtops` também funcionam.

### Comportamento de clique

- **Clique no item de um top no menu** → executa o `open-command` daquele top (definido em `tops.yml`)
- **Clique direito num NPC** → executa o `open-command` do top daquele NPC

O VixTOPs **não tem menu próprio de visualização do top** — quem mostra o ranking é o plugin/comando que você apontar em `open-command` (ex: `top money` do VixEconomias, ou um comando seu).

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

### Diferença entre os dois

- **NPC**: jogador físico no mundo (posição #N do top). Skin = do jogador na posição (ou skin fixa). Tem holograma acima da cabeça (configurável).
- **Holograma de lista**: ranking flutuante completo. Sem entidade visível, só texto.

### Criar um NPC

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

### PacketEvents

**Obrigatório** (`depend`). É a biblioteca de packets cross-version que faz os NPCs e hologramas funcionarem de 1.8 a 1.21 sem precisar de Citizens ou DecentHolograms.

---

## 10. Compatibilidade entre versões

Testado de **1.8.X** até **1.21.X**.

### O que se adapta automaticamente

- **Materials** no menu: `PLAYER_HEAD` ↔ `SKULL_ITEM:3`, `BLACK_STAINED_GLASS_PANE` ↔ `STAINED_GLASS_PANE:15`, etc.
- **Hex colors** (`&#RRGGBB`): renderizados em 1.16+, removidos do output em versões antigas
- **`SkullMeta.setOwningPlayer`** ↔ `setOwner(String)` via reflection
- **Packets de NPC/holograma**: PacketEvents resolve os índices de metadata por versão do cliente
- **`Bukkit.getOnlinePlayers()`** array (1.8) ↔ Collection (modernos) — funciona pelo for-each

### O que sempre funciona

- Códigos de cor clássicos: `&a`, `&l`, `&r`, etc.
- Comandos, eventos, listeners, scheduler
- Hooks de Vault, VixEconomias, PlaceholderAPI, LuckPerms
- Save assíncrono

### Limitações do cliente (não dá pra contornar no plugin)

- Cliente 1.8 não renderiza hex colors (limitação do protocolo)
- Cliente 1.8 não tem nametag invisível por padrão — o plugin usa color codes para mascarar o nome do NPC
- Skin em servidor offline-mode depende de o NICK do jogador existir na Mojang

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
