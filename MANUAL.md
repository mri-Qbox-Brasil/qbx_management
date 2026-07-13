# Manual do qbx_management

Menu de gerenciamento de empresas e gangues — sistema abrangente para gerenciamento de funcionários, promoções, contratações e demissões.

## Funcionalidades Principais

### Gerenciamento de Empresa
- **Lista de Funcionários**: Visualizar todos os funcionários atuais com status online/offline
- **Contratar**: Contratar novos funcionários entre jogadores próximos
- **Demitir**: Remover funcionários da organização
- **Promover/Rebaixar**: Alterar graus e cargos dos funcionários
- **Menu do Chefe**: Acesso rápido a todas as funções de gerenciamento

### Gerenciamento de Gangue
- Funcionalidades idênticas ao gerenciamento de empresa
- Suporte completo para líderes de gangue
- Gerenciamento de membros e hierarquia

### Sistema de Permissões
- **Chefe**: Pode contratar, demitir, promover e rebaixar
- **Recrutador**: Pode apenas contratar novos membros
- Permissões gerenciadas via mri_Qjobsystem

### Interface
- UI moderna para gerenciamento
- Indicadores visuais: 🟢 online, ❌ offline
- Navegação intuitiva entre abas

### Acesso
- **Comando**: `/bossmenu` (configurável)
- **Zonas**: Use ox_target ou text UI em locais configurados
- **Atalhos**: `/+tablet:job` e `/+tablet:gang`

### Logging
- Registro de ações de contratação/demissão
- Webhook do Discord para auditoria
- Log de promoções e alterações de cargo

## Comandos

| Comando | Permissão | Descrição |
|----------|-------------|-------------|
| `/bossmenu` | Configurável | Abrir menu de gerenciamento do chefe |
| `/+tablet:job` | - | Abrir rapidamente menu do chefe do trabalho |
| `/+tablet:gang` | - | Abrir rapidamente menu do chefe da gangue |

## Configuração

### config/server.lua
```lua
config = {
    commandName = 'bossmenu',      -- Comando para abrir menu
    commandHelp = 'Open boss menu', -- Texto de ajuda
    discordWebhook = '',            -- URL do webhook Discord
    menus = {                       -- Zonas de menu do chefe
        {
            groupName = 'police',
            type = 'job',
            coords = vector3(440.84, -981.34, 30.69),
            size = vec3(2.0, 2.0, 2.0),
            rotation = 0.0
        },
        -- Adicione mais menus conforme necessário
    }
}
```

### config/client.lua
```lua
config = {
    useTarget = true,       -- Usar ox_target (true) ou text UI (false)
    holdTablet = true,      -- Animação de tablet ao abrir menu
    debugPoly = false,      -- Mostrar polígonos de debug
}
```

## Exports (API)

### Client Exports

| Export | Parâmetros | Retorno | Descrição |
|--------|------------|--------|-------------|
| `OpenBossMenu` | `groupType?` | - | Abrir menu do chefe ('job' ou 'gang') |
| `AddBossMenuItem` | `menuItem` | `number` | Adicionar item personalizado ao menu |
| `AddGangMenuItem` | `menuItem` | `number` | Adicionar item ao menu da gangue |
| `RemoveBossMenuItem` | `id` | - | Remover item do menu |
| `RemoveGangMenuItem` | `id` | - | Remover item do menu |

### Server Exports

| Export | Parâmetros | Descrição |
|--------|------------|-------------|
| `RegisterBossMenu` | `menuInfo` | Registrar zona de menu do chefe |

### Estrutura do MenuInfo
```lua
---@class MenuInfo
---@field groupName string  # Nome do grupo (trabalho/gangue)
---@field type GroupType      # 'job' ou 'gang'
---@field coords vector3      # Coordenadas da zona
---@field size? vector3       # Tamanho da zona (padrão: vec3(1.5, 1.5, 1.5))
---@field rotation? number    # Rotação da zona (padrão: 0.0)
```

## Callbacks do Servidor

| Callback | Parâmetros | Retorno | Descrição |
|----------|------------|--------|-------------|
| `qbx_management:server:getEmployees` | `groupName, groupType` | `table[]` | Lista de funcionários |
| `qbx_management:server:updateGrade` | `citizenId, oldGrade, newGrade, groupType` | - | Atualizar grau |
| `qbx_management:server:hireEmployee` | `employee, groupType` | - | Contratar funcionário |
| `qbx_management:server:fireEmployee` | `employee, groupType` | - | Demitir funcionário |
| `qbx_management:server:getPlayers` | `closePlayers` | `table[]` | Jogadores próximos |
| `qbx_management:server:getBossMenus` | - | `table[]` | Menus registrados |

## Eventos

### Client Events

| Evento | Payload | Descrição |
|-------|----------|-------------|
| `qbx_management:client:OpenBossMenu` | `menuType?` | Abrir menu do chefe |
| `qbx_management:client:bossMenuRegistered` | `menuInfo` | Nova zona registrada |

## Exemplos de Uso

### Abrindo o Menu do Chefe
```lua
-- Via export
exports.qbx_management:OpenBossMenu('job')
exports.qbx_management:OpenBossMenu('gang')

-- Via evento
TriggerClientEvent('qbx_management:client:OpenBossMenu', source, 'job')
```

### Adicionando Itens Personalizados
```lua
local menuId = exports.qbx_management:AddBossMenuItem({
    title = 'Ação Personalizada',
    description = 'Fazer algo personalizado',
    icon = 'wrench',
    args = {
        type = 'job' -- ou 'gang'
    },
    onSelect = function()
        -- Sua lógica aqui
    end
})
```

### Registrando Zonas (Servidor)
```lua
exports.qbx_management:RegisterBossMenu({
    groupName = 'police',
    type = 'job',
    coords = vector3(440.84, -981.34, 30.69),
    size = vec3(2.0, 2.0, 2.0),
    rotation = 0.0
})
```

## Estrutura de Arquivos

```
qbx_management/
├── client/
│   └── main.lua           # UI do menu, gerenciamento, zonas
├── server/
│   ├── main.lua           # Funcionários, contratação, logging
│   └── storage.lua         # Banco de dados
├── config/
│   ├── client.lua         # Config do client
│   └── server.lua         # Config do servidor
└── locales/               # Traduções
```

## Dependências

| Dependência | Versão Mínima | Obrigatória |
|------------|-------------------|----------|
| ox_lib | 3.13.0 | ✅ |
| qbx_core | 1.7.0 | ✅ |
| mri_Qjobsystem | - | ✅ (Específico MRI) |

## Permissões

- **Chefe**: Acesso total ao menu, pode contratar, demitir, promover, rebaixar
- **Recrutador**: Pode apenas contratar novos funcionários
- Permissões definidas no mri_Qjobsystem

## Solução de Problemas

### Menu não abre
- Verifique se o jogador tem permissão de chefe/recrutador
- Confirme que o mri_Qjobsystem está rodando
- Verifique se o grupo existe em qbx_core

### Zona não funciona
- Verifique a configuração em `config/server.lua`
- Confirme se `useTarget` está configurado corretamente
- Verifique se as coordenadas estão corretas

### Funcionário não aparece na lista
- Recarregue o menu após contratação
- Verifique se o jogador está no banco de dados
- Confirme que o grupo foi atribuído corretamente

### Webhook não envia logs
- Verifique se a URL do webhook está correta
- Confirme que o Discord aceita a conexão
- Verifique os logs do servidor para erros
