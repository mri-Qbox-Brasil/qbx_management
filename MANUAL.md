# qbx_management — Manual

Menu de chefe para jobs e gangues: contratar, promover, rebaixar e demitir membros, com zonas de acesso e itens de menu extensíveis por outros recursos.

---

## Sumário

1. [Dependências](#dependências)
2. [Instalação](#instalação)
3. [Configuração](#configuração)
4. [Comandos](#comandos)
5. [Menu do chefe](#menu-do-chefe)
6. [Zonas](#zonas)
7. [Integrações](#integrações)
8. [Entrypoints para outros recursos](#entrypoints-para-outros-recursos)
9. [Localização](#localização)
10. [Estrutura de arquivos](#estrutura-de-arquivos)

---

## Dependências

| Recurso | Obrigatório | Observação |
|---|---|---|
| `qbx_core` | Sim | Versão 1.7.0 ou superior. Jobs, gangues, jogadores online e offline, notificações e o módulo `playerdata` |
| `ox_lib` | Sim | Versão 3.13.0 ou superior. Context menus, zonas, callbacks, locale |
| `oxmysql` | Sim | Lê a tabela `player_groups` para listar membros, inclusive offline |
| `mri_Qjobsystem` | Sim | O menu chama `CheckPlayerIsbossByJobSystemData` e `CheckPlayerIrecruiterByJobSystemData` para decidir o que mostrar. Sem ele, o menu não abre |
| `ox_target` | Não | Só quando `useTarget = true`; troca o "pressione E" pela interação de target |
| Recurso de emotes (`dpemotes`/`rpemotes`) | Não | Só quando `holdTablet = true`; o menu executa `e tablet` ao abrir e `e c` ao fechar |

A tabela `player_groups` já é criada pelo `qbx_core` — este recurso não traz SQL próprio.

---

## Instalação

1. Copie a pasta `qbx_management` para `resources/`.
2. Adicione ao `server.cfg`, depois do `qbx_core`:
   ```
   ensure qbx_management
   ```
3. Nenhum SQL adicional: o recurso consulta a tabela `player_groups` do `qbx_core`.
4. **Conflitos** — não rode junto com o `qb-management`. Ambos servem o mesmo papel de menu de chefe.

---

## Configuração

### `config/client.lua`

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `useTarget` | bool | Sim | `true` cria as zonas com `ox_target`; `false` usa `lib.zones.box` com TextUI e tecla `E` |
| `debugPoly` | bool | Sim | Desenha as zonas na tela para depuração |
| `holdTablet` | bool | Sim | Toca a emote `tablet` ao abrir o menu e `c` (cancelar) ao fechar |

### `config/server.lua`

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `commandName` | string | Sim | Nome do comando que abre o menu (padrão `tablet`) |
| `commandHelp` | string | Sim | Texto de ajuda do comando |
| `discordWebhook` | string ou `nil` | Não | Webhook para os logs de contratação, promoção e demissão. Com `nil`, o log vai pelo `ox_lib` |
| `menus.toggle` | bool | Sim | `false` desativa todas as zonas declaradas abaixo dele |
| `menus.<grupo>` | tabela | Não | Zona de acesso do grupo. Campos: `coords` (vec3), `size` (vec3), `rotation` (número), `type` (`job` ou `gang`) |

As zonas do config continuam funcionando, mas o caminho recomendado é registrar em runtime com o export `RegisterBossMenu`.

---

## Comandos

| Comando | Permissão | Descrição |
|---|---|---|
| `/tablet` | Chefe ou recrutador de um job/gangue | Abre o menu. Se o jogador for chefe de um job **e** de uma gangue, pergunta qual grupo abrir |
| `/+tablet:job` | Chefe ou recrutador do job | Abre o menu direto no job. Feito para ser mapeado em uma tecla |
| `/+tablet:gang` | Chefe ou recrutador da gangue | Abre o menu direto na gangue. Feito para ser mapeado em uma tecla |

O nome do primeiro comando vem de `config.commandName`.

---

## Menu do chefe

O que aparece depende do papel do jogador, resolvido pelo `mri_Qjobsystem`:

| Papel | Opções |
|---|---|
| Chefe (`isboss`) | Gerenciar membros (listar, promover, rebaixar, demitir), contratar, e todos os itens dinâmicos registrados por outros recursos para aquele tipo de grupo |
| Recrutador (`isrecruiter`) | Apenas contratar |

- **Gerenciar membros** — lista quem está no grupo, online (marcado) e offline, ordenado por grade. Ao escolher alguém, o menu oferece cada grade do job/gangue para promoção ou rebaixamento, e a opção de demitir/expulsar.
- **Contratar** — lista os jogadores num raio de 10 metros que ainda não pertencem ao grupo, com nome, citizenid e ID de servidor.

Todas as ações são revalidadas no servidor: os callbacks conferem `PlayerData[groupType].isboss` antes de executar. Contratações, promoções e demissões geram log via `qbx_core.modules.logger` (webhook de `discordWebhook`, quando definido).

---

## Zonas

Cada zona registrada (pelo config ou pelo export) vira um ponto de acesso ao menu.

- Com `useTarget = false`: uma `lib.zones.box`. Ao entrar, aparece um TextUI; a tecla `E` abre o menu. Só funciona para quem é chefe daquele grupo.
- Com `useTarget = true`: uma box zone do `ox_target` com a opção "Menu do chefe" / "Menu da gangue", visível só para o chefe do grupo correspondente.
- `size` e `rotation` são opcionais; sem eles, a zona usa `vec3(1.5, 1.5, 1.5)` e rotação `0.0`.

---

## Integrações

### mri_Qjobsystem

Define quem é chefe e quem é recrutador. `OpenBossMenu` chama `CheckPlayerIsbossByJobSystemData(groupType, QBX.PlayerData)` e `CheckPlayerIrecruiterByJobSystemData(groupType, QBX.PlayerData)`. É esse recurso que permite o papel de recrutador, que só pode contratar.

### ox_target

Com `useTarget = true`, as zonas viram box zones do `ox_target` em vez do fluxo TextUI + tecla `E`.

### Emotes

Com `holdTablet = true`, o recurso roda `ExecuteCommand('e tablet')` ao abrir o menu e `ExecuteCommand('e c')` ao fechar, o que exige um recurso de emotes que aceite esses comandos.

---

## Entrypoints para outros recursos

### Registrar uma zona de menu em runtime (servidor)

```lua
exports.qbx_management:RegisterBossMenu({
    groupName = 'police',
    type = 'job',              -- 'job' ou 'gang'
    coords = vec3(441.7, -978.9, 30.6),
    size = vec3(1.5, 1.5, 1.5), -- opcional
    rotation = 0.0,             -- opcional
})
```

A zona é criada imediatamente em todos os clientes conectados via `qbx_management:client:bossMenuRegistered`.

### Adicionar itens ao menu (cliente)

```lua
-- Retorna o id do item, usado para removê-lo depois
local id = exports.qbx_management:AddBossMenuItem({
    title = 'Cofre da empresa',
    description = 'Abrir o cofre',
    icon = 'vault',
    args = { type = 'job' },   -- obrigatório: 'job' ou 'gang'
    onSelect = function()
        -- ...
    end,
})

exports.qbx_management:RemoveBossMenuItem(id)
```

`AddGangMenuItem` e `RemoveGangMenuItem` são apelidos das mesmas funções — o que decide em qual menu o item aparece é o `args.type`. Os itens dinâmicos só são exibidos para quem é chefe.

### Abrir o menu (cliente)

```lua
exports.qbx_management:OpenBossMenu('job')   -- ou 'gang'
```

### Eventos

```lua
-- Servidor -> cliente: abre o menu. Sem o segundo argumento, pergunta o grupo quando há mais de um
TriggerClientEvent('qbx_management:client:OpenBossMenu', source, 'job')

-- Servidor -> todos: nova zona registrada
TriggerClientEvent('qbx_management:client:bossMenuRegistered', -1, menuInfo)
```

### Callbacks (`lib.callback`)

Todos exigem que o chamador seja chefe do grupo.

```lua
lib.callback.await('qbx_management:server:getEmployees', false, groupName, groupType)
lib.callback.await('qbx_management:server:hireEmployee', false, targetSource, groupType)
lib.callback.await('qbx_management:server:fireEmployee', false, citizenid, groupType)
lib.callback.await('qbx_management:server:updateGrade', false, citizenid, gradeAtual, novoGrade, groupType)
lib.callback.await('qbx_management:server:getPlayers', false, closePlayers)
lib.callback.await('qbx_management:server:getBossMenus', false)
```

---

## Localização

Strings via `ox_lib` locale. Arquivos em `locales/`:

- `da.json`, `en.json`, `pt-br.json`, `pt.json`

Idioma ativo pela convar no `server.cfg`:

```
setr ox:locale "pt-br"
```

---

## Estrutura de arquivos

```
qbx_management/
├── client/
│   └── main.lua       — menu do chefe, itens dinâmicos, zonas (ox_target ou lib.zones)
├── server/
│   ├── main.lua       — callbacks de contratar/promover/demitir, RegisterBossMenu, comandos
│   └── storage.lua    — consulta dos membros do grupo na tabela player_groups
├── config/
│   ├── client.lua     — useTarget, debugPoly, holdTablet
│   └── server.lua     — comando, webhook e zonas de menu
├── locales/           — da, en, pt-br, pt
└── fxmanifest.lua
```
