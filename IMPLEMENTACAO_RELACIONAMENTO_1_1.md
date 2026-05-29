# Implementação: Relacionamento 1:1 (Partida ↔ PartidaEstatistica)

## 📋 Resumo da Implementação

Foi implementado um **relacionamento um-para-um (1:1)** entre as entidades `Partida` e `PartidaEstatistica`, separando dados básicos de desempenho de uma partida de seus dados estatísticos detalhados.

---

## 🗂️ Arquivos Criados/Modificados

### ✅ Novos Arquivos

1. **`src/api/models/PartidaEstatistica.cs`** - Novo modelo com estatísticas detalhadas
2. **`src/api/models/CriarPartidaComEstatisticasDto.cs`** - DTO para criar Partida com Estatísticas
3. **`src/api/Migrations/20260525221754_CriarPartidaEstatistica.cs`** - Migração do EF Core

### 🔄 Arquivos Modificados

1. **`src/api/models/Partida.cs`** - Adicionado navigation property `Estatistica?`
2. **`src/api/database/AppDbContext.cs`** - Configurado relacionamento 1:1 com query filters
3. **`src/api/Controllers/PartidasController.cs`** - Atualizado GET e POST com eager loading
4. **`src/api/api.http`** - Exemplos de uso da API

---

## 🔗 Estrutura do Relacionamento

```csharp
// Partida.cs
public class Partida
{
    public int Id { get; set; }
    // ... campos básicos (Mapa, Modo, Agente, KDA, Score, etc)
    
    // Navigation Property (1:1)
    public PartidaEstatistica? Estatistica { get; set; }
}

// PartidaEstatistica.cs
public class PartidaEstatistica
{
    public int Id { get; set; }
    public int PartidaId { get; set; }  // Foreign Key
    
    // Dados detalhados
    public int AcsScore { get; set; }
    public int EconScore { get; set; }
    public int RoundasGanhas { get; set; }
    public int RoundasPerdidas { get; set; }
    public double HeadshotPercentual { get; set; }
    public int BodyShotCount { get; set; }
    public int LegShotCount { get; set; }
    public string ArmaFavorita { get; set; }
    public int DanosCausados { get; set; }
    public int DanosRecebidos { get; set; }
    
    // Navigation Property (1:1 inverso)
    public Partida? Partida { get; set; }
}
```

---

## 🔧 Configuração no AppDbContext

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Soft Delete para Partidas
    modelBuilder.Entity<Partida>()
        .HasQueryFilter(p => p.DeletedAt == null);
    
    // Relacionamento 1:1 com Cascade Delete
    modelBuilder.Entity<Partida>()
        .HasOne(p => p.Estatistica)
        .WithOne(e => e.Partida)
        .HasForeignKey<PartidaEstatistica>(e => e.PartidaId)
        .OnDelete(DeleteBehavior.Cascade);
    
    // Query filter para PartidaEstatistica seguir o filtro de Partida deletada
    modelBuilder.Entity<PartidaEstatistica>()
        .HasQueryFilter(pe => pe.Partida.DeletedAt == null);
}
```

### Características

- **Foreign Key:** `PartidaId` na tabela `PartidaEstatisticas`
- **Unique Constraint:** Um índice único garante 1:1
- **Cascade Delete:** Deletar uma Partida deleta suas Estatísticas automaticamente
- **Soft Delete Compatible:** Query filters respeitam o Soft Delete de Partida

---

## 📡 Endpoints da API

### GET - Listar Partidas com Estatísticas

```http
GET /api/v1/Partidas?page=1&size=10
```

**Response (exemplo):**

```json
{
  "page": 1,
  "size": 10,
  "total": 150,
  "totalPaginas": 15,
  "partidas": [
    {
      "id": 1,
      "mapa": "Abyss",
      "modo": "Competitive",
      "agente": "Omen",
      "resultado": "Vitoria",
      "kills": 15,
      "deaths": 8,
      "assists": 6,
      "kda": 2.63,
      "dataPartida": "2026-05-25T10:30:00Z",
      "estatistica": {
        "id": 1,
        "acsScore": 284,
        "headshotPercentual": 42.5,
        "rundasGanhas": 13,
        "rundasPerdidas": 7,
        "danosCausados": 3200,
        "danosRecebidos": 1850
      }
    }
  ]
}
```

---

### GET - Buscar Partida Específica com Estatísticas

```http
GET /api/v1/Partidas/1
```

**Response:**

```json
{
  "id": 1,
  "matchId": "match-123-abc",
  "mapa": "Abyss",
  "modo": "Competitive",
  "agente": "Omen",
  "resultado": "Vitoria",
  "kills": 15,
  "deaths": 8,
  "assists": 6,
  "kda": 2.63,
  "score": 2840,
  "dataPartida": "2026-05-25T10:30:00Z",
  "deletedAt": null,
  "estatistica": {
    "id": 1,
    "partidaId": 1,
    "acsScore": 284,
    "econScore": 18500,
    "rundasGanhas": 13,
    "rundasPerdidas": 7,
    "headshotPercentual": 42.5,
    "bodyShotCount": 18,
    "legShotCount": 5,
    "armaFavorita": "Vandal",
    "danosCausados": 3200,
    "danosRecebidos": 1850,
    "criadoEm": "2026-05-25T10:30:00Z"
  }
}
```

---

### POST - Criar Partida COM Estatísticas (Novo DTO)

```http
POST /api/v1/Partidas
Content-Type: application/json

{
  "matchId": "match-123-abc",
  "puuid": "player-puuid-123",
  "gameName": "SeuNome",
  "tagLine": "BR",
  "mapa": "Abyss",
  "modo": "Competitive",
  "agente": "Omen",
  "resultado": "Vitoria",
  "kills": 15,
  "deaths": 8,
  "assists": 6,
  "kda": 2.63,
  "score": 2840,
  "dataPartida": "2026-05-25T10:30:00Z",
  "estatistica": {
    "acsScore": 284,
    "econScore": 18500,
    "rundasGanhas": 13,
    "rundasPerdidas": 7,
    "headshotPercentual": 42.5,
    "bodyShotCount": 18,
    "legShotCount": 5,
    "armaFavorita": "Vandal",
    "danosCausados": 3200,
    "danosRecebidos": 1850
  }
}
```

### POST - Criar Partida SEM Estatísticas (Opcional)

```http
POST /api/v1/Partidas
Content-Type: application/json

{
  "matchId": "match-456-def",
  "puuid": "player-puuid-456",
  "gameName": "OutroJogador",
  "tagLine": "BR",
  "mapa": "Haven",
  "modo": "Deathmatch",
  "agente": "Jett",
  "resultado": "Derrota",
  "kills": 8,
  "deaths": 12,
  "assists": 3,
  "kda": 0.92,
  "score": 1650,
  "dataPartida": "2026-05-25T14:15:00Z"
}
```

---

## 💡 Vantagens da Implementação

✅ **Separação de Responsabilidades:** Dados básicos vs. estatísticas detalhadas  
✅ **Melhor Performance:** Queries mais limpas e otimizadas com eager loading  
✅ **Integridade Referencial:** Cascade Delete garante consistência  
✅ **Extensibilidade:** Fácil adicionar novos campos de estatística sem quebrar Partida  
✅ **API Limpa:** DTO unifica criação de ambas as entidades  
✅ **Soft Delete Compatible:** Estatísticas são filtradas quando Partida é deletada  

---

## 📊 Diagrama ER

```
┌─────────────────────────────┐
│        Partida              │
├─────────────────────────────┤
│ Id (PK)                     │
│ MatchId                     │
│ Puuid                       │
│ GameName                    │
│ TagLine                     │
│ Mapa                        │
│ Modo                        │
│ Agente                      │
│ Resultado                   │
│ Kills, Deaths, Assists      │
│ Kda, Score                  │
│ DataPartida                 │
│ DeletedAt (Soft Delete)     │
│ ↓ (1:1)                     │
└─────────────────────────────┘
           |
           | FK_PartidaId (Unique)
           ↓
┌─────────────────────────────┐
│   PartidaEstatistica        │
├─────────────────────────────┤
│ Id (PK)                     │
│ PartidaId (FK, Unique)      │
│ AcsScore                    │
│ EconScore                   │
│ RoundasGanhas               │
│ RoundasPerdidas             │
│ HeadshotPercentual          │
│ BodyShotCount               │
│ LegShotCount                │
│ ArmaFavorita                │
│ DanosCausados               │
│ DanosRecebidos              │
│ CriadoEm                    │
└─────────────────────────────┘
```

---

## ✅ Checklist do Projeto Atualizado

- [x] **Relacionamento 1:1 implementado** ✨ Partida ↔ PartidaEstatistica
- [x] **Migração do EF Core criada e aplicada**
- [x] **Controllers atualizados com eager loading**
- [x] **DTO para criar ambas as entidades em uma requisição**
- [x] **Documentação e exemplos de API**
- [x] **Soft Delete compatible**

---

## 🚀 Próximos Passos (Opcional)

Se precisar implementar também o **relacionamento 1:N (Usuario → Partidas)**, basta adicionar uma Foreign Key em Partida apontando para Usuario e configurar similar ao feito aqui.
