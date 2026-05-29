# <img src="https://raw.githubusercontent.com/KaroliniRPedrozo/Cyphers-Analytics_Frontend/main/src/assets/valorant-white-logo.png" width="70" align="center"> Cypher's Analytics — Backend

> Web API RESTful de alta performance para compilação e análise de dados do Valorant. Construída com arquitetura desacoplada, cache inteligente e segurança robusta.

![Status do Projeto](https://img.shields.io/badge/Status-Em_Desenvolvimento-yellow?style=for-the-badge)
![.NET](https://img.shields.io/badge/.NET_8-5C2D91?style=for-the-badge&logo=.net&logoColor=white)
![C#](https://img.shields.io/badge/C%23-239120?style=for-the-badge&logo=c-sharp&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)

---

## 📋 Sobre o Repositório

Este repositório contém o código-fonte do **Backend** da plataforma **Cypher's Analytics**. A API é responsável por toda a lógica de negócio: autenticação de usuários, integração com APIs externas do Valorant, persistência de dados, cálculos de estatísticas e gerenciamento de cache.

A API serve dados ao [Frontend React](https://github.com/KaroliniRPedrozo/Cyphers-Analytics_Frontend) e está devidamente versionada sob o prefixo `/api/v1/`.

---

## ✨ Funcionalidades Principais

- 🔐 **Gestão Autenticada de Usuários:** Registro, login e atualização de perfil com criptografia BCrypt.
- ⚡ **Cache Inteligente de Partidas:** Minimiza requisições a APIs externas atualizando dados apenas a cada 15 minutos, prevenindo *rate limiting*.
- 📊 **Cálculo Dinâmico de Estatísticas:** KDA, Taxa de Vitória, Melhor Mapa e Melhor Agente calculados dinamicamente no backend.
- 🗑️ **Soft Delete de Registros:** Deleção lógica de partidas com flag `DeletedAt`, permitindo auditoria e recuperação futura de dados.
- 📢 **Monitorização de Status do Jogo:** Verificação de agentes desativados ou indisponíveis em tempo real.
- 📄 **Paginação Nativa:** Suporte a `page` e `size` nos endpoints de listagem.

---

## 🚀 Tecnologias Utilizadas

| Tecnologia | Finalidade |
| :--- | :--- |
| **C# / .NET 8** | Web API RESTful com controllers assíncronos e injeção de dependência |
| **Entity Framework Core** | ORM para abstração de dados e gerenciamento de migrações |
| **PostgreSQL** | Banco relacional para credenciais, cache de jogadores e histórico de partidas |
| **BCrypt.Net** | Hashing seguro de senhas contra ataques de força bruta |
| **Asp.Versioning** | Controle de versão da API com padrão `/api/v1/` |

---

## 🗄️ Diagrama de Entidades (ER)

```
┌─────────────────────────────────┐
│          USUARIOS               │
├─────────────────────────────────┤
│ • Id (PK)                       │
│ • Email                         │
│ • SenhaHash                     │
│ • GameName                      │
│ • TagLine                       │
│ • CriadoEm                      │
└─────────┬───────────────────────┘
          │
          │ 1:N
          ▼
┌─────────────────────────────────────────────────────────┐
│          PARTIDAS                                       │
├─────────────────────────────────────────────────────────┤
│ • Id (PK)                                               │
│ • MatchId                                               │
│ • Puuid / GameName / TagLine                            │
│ • Mapa / Modo / Agente                                  │
│ • Resultado / Kills / Deaths / Assists / KDA            │
│ • Score / DataPartida                                   │
│ • DeletedAt (Soft Delete)                               │
│ • UsuarioId (FK)                                        │
└─────────┬───────────────────────────────────────────────┘
          │
          │ 1:1
          ▼
┌────────────────────────────────────────────────────────┐
│          PARTIDAESTATISTICA                            │
├────────────────────────────────────────────────────────┤
│ • Id (PK)                                              │
│ • PartidaId (FK, UNIQUE)                               │
│ • AcsScore / EconScore                                 │
│ • RoundasGanhas / RoundasPerdidas                      │
│ • HeadshotPercentual / BodyShotCount / LegShotCount    │
│ • ArmaFavorita / DanosCausados / DanosRecebidos        │
│ • CriadoEm                                             │
└────────────────────────────────────────────────────────┘
```

**Relacionamentos:**
- `1:N` **Usuario → Partidas**: Cada usuário pode ter múltiplas partidas. Deletar usuário define `UsuarioId` como NULL.
- `1:1` **Partida ↔ PartidaEstatistica**: Cada partida possui exatamente uma estatística. Deletar partida deleta a estatística (Cascade).

---

## 🛣️ Documentação da API (Endpoints)

### 🔐 Autenticação — `/api/v1/Auth`

| Método | Rota | Descrição | Payload (JSON) |
| :--- | :--- | :--- | :--- |
| `POST` | `/registrar` | Cria uma nova conta de usuário. | `{ "email", "senha", "gameName", "tagLine" }` |
| `POST` | `/login` | Autentica o usuário no sistema. | `{ "email", "senha" }` |
| `PUT` | `/perfil` | Atualiza informações de perfil, senha ou nick. | `{ "emailAtual", "senhaAtual", "novoEmail"?, "novaSenha"?, "gameName"?, "tagLine"? }` |

### 🎮 Jogadores — `/api/v1/Jogadores`

| Método | Rota | Descrição |
| :--- | :--- | :--- |
| `GET` | `/temporada` | Retorna informações da Season e Act vigente. |
| `GET` | `/{gameName}/{tagLine}` | Retorna o perfil resumido do jogador. Sincronização automática a cada 15 minutos. |
| `PUT` | `/{gameName}/{tagLine}` | Força reatualização manual do cache e importa novas partidas. |
| `GET` | `/{gameName}/{tagLine}/historico` | Retorna o histórico paginado de partidas. Query Params: `page` (padrão 1), `size` (padrão 10). |
| `GET` | `/{gameName}/{tagLine}/estatisticas` | Consolida performance por Mapa, Agente e Modo de Jogo. |

### 📝 Partidas — `/api/v1/Partidas`

| Método | Rota | Descrição |
| :--- | :--- | :--- |
| `GET` | `/` | Lista todas as partidas com paginação (`page`, `size`). |
| `GET` | `/{id}` | Recupera detalhes completos de uma partida pelo ID. |
| `POST` | `/` | Registra manualmente uma nova partida. |
| `PUT` | `/{id}` | Atualiza uma partida. KDA recalculado automaticamente se `Deaths == 0`. |
| `DELETE` | `/{id}` | Soft Delete: define `DeletedAt` sem apagar o dado fisicamente. |

### 📢 Status do Jogo — `/api/v1/Status`

| Método | Rota | Descrição |
| :--- | :--- | :--- |
| `GET` | `/agentes-desativados` | Retorna a lista de agentes desativados ou indisponíveis no momento. |

---

## 🔧 Detalhes de Implementação

### Cache de Partidas com HashSet

Para evitar inserções duplicadas, o sistema verifica os `MatchId` já persistidos usando um `HashSet`, inserindo em lote apenas partidas inéditas:

```csharp
var matchIdsExist = _context.Partidas.Select(p => p.MatchId).ToHashSet();
var partidasNovas = partidas
    .Where(p => string.IsNullOrEmpty(p.MatchId) || !matchIdsExist.Contains(p.MatchId))
    .ToList();
```

### Soft Delete

A remoção lógica preserva o dado para auditoria e recuperação futura:

```csharp
partida.DeletedAt = DateTime.UtcNow;
await _context.SaveChangesAsync();
```

---

## ⚙️ Como Executar

### Pré-requisitos

- [.NET SDK 8.0](https://dotnet.microsoft.com/download)
- [PostgreSQL](https://www.postgresql.org/)

### Configuração e Execução

```bash
# Clone o repositório
git clone https://github.com/KaroliniRPedrozo/Cyphers-Analytics_Backend.git
cd Cyphers-Analytics_Backend
```

Configure sua Connection String do PostgreSQL em `appsettings.json` ou `appsettings.Development.json`:

```json
"ConnectionStrings": {
  "DefaultConnection": "Host=localhost;Database=cyphers_analytics;Username=seu_usuario;Password=sua_senha"
}
```

```bash
# Restaure as dependências
dotnet restore

# Inicie a API
dotnet run
```

A API estará disponível em: `http://localhost:5000` ou `https://localhost:5001`

---

## 📂 Estrutura de Pastas

```
src/
├── Controllers/   # Controladores da API (Auth, Jogadores, Partidas, Status)
├── database/      # Contexto de dados do EF Core (AppDbContext)
├── models/        # Entidades de domínio (Usuario, Partida, PartidaEstatistica)
└── services/      # Serviços e integrações externas (HenrikService, AuthService)
```

---

## 🔗 Repositório Relacionado

- 🎨 **Frontend (React + Vite):** [CyphersAnalytics_Frontend](https://github.com/KaroliniRPedrozo/Cyphers-Analytics_Frontend)

---

## ✍ Autoria

Desenvolvido com dedicação por **Karolini R. Pedrozo**.
