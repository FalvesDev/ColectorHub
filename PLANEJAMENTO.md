# ColectorHub — Planejamento Completo do App (v1.0)

> App mobile para colecionadores de jogos físicos com scan inteligente de coleção.
> Plataforma inicial: **PlayStation 1 (PS1/PSX)**
> Stack: **Flutter + Supabase + Google ML Kit / Gemini Vision**

---

## Sumário

1. [Visão Geral](#1-visão-geral)
2. [Escopo da v1.0](#2-escopo-da-v10)
3. [Arquitetura Técnica](#3-arquitetura-técnica)
4. [Autenticação e Contas](#4-autenticação-e-contas)
5. [Algoritmo de Scan (OCR + IA)](#5-algoritmo-de-scan-ocr--ia)
6. [Integração com APIs de Jogos](#6-integração-com-apis-de-jogos)
7. [Banco de Dados](#7-banco-de-dados)
8. [Telas e Fluxo do Usuário](#8-telas-e-fluxo-do-usuário)
9. [Estrutura de Pastas (Flutter)](#9-estrutura-de-pastas-flutter)
10. [Fases de Desenvolvimento](#10-fases-de-desenvolvimento)
11. [Dependências e Pacotes](#11-dependências-e-pacotes)
12. [Requisitos Não-Funcionais](#12-requisitos-não-funcionais)
13. [Roadmap Futuro (pós v1.0)](#13-roadmap-futuro-pós-v10)

---

## 1. Visão Geral

**ColectorHub** é um aplicativo mobile para colecionadores de jogos físicos. O usuário cadastra sua coleção tirando fotos das prateleiras ou lombadas dos jogos. O app utiliza inteligência artificial para identificar automaticamente os títulos a partir das imagens e registrá-los na coleção digital do usuário, organizada por console.

### Proposta de Valor

- Scan automático de lombadas e capas via foto
- Catálogo digital organizado por console
- Metadados ricos: capa oficial, gênero, ano, developer, publisher
- Perfil público de coleção (futuro)
- Sem necessidade de digitar nome por nome

---

## 2. Escopo da v1.0

### O que ENTRA na v1.0

| Funcionalidade | Descrição |
|---|---|
| Autenticação | Cadastro e login com e-mail/senha e Google |
| Perfil do usuário | Nome, foto, bio curta |
| Coleção PS1 | Registro e visualização da coleção de PS1 |
| Scan por foto | Captura de foto e identificação dos jogos via OCR + IA |
| Adição manual | Busca e adição manual por nome do jogo |
| Catálogo de referência | Lista de jogos do PS1 via API externa |
| Detalhes do jogo | Tela com capa, descrição, gênero, ano, publisher |
| Wishlist | Lista de jogos que o usuário quer mas não tem |

### O que NÃO entra na v1.0

- Outros consoles (PS2, PS3, SNES, etc.)
- Avaliação/review de jogos
- Funcionalidades sociais (seguir usuários, feed)
- Valor de mercado / integração com eBay ou Mercado Livre
- Modo offline completo
- Notificações push
- Colocar zerados, em jogo, desejados
- Adicionar jogos de outras plataformas
- Roleta para jogo do dia
- Metodo de slots de jogos para ajudar a zerar mais jogos
- Sistema atual de platina e conquistas.
---

## 3. Arquitetura Técnica

```
┌─────────────────────────────────────────────────────────────┐
│                     FLUTTER APP (Mobile)                    │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  ┌──────────┐    │
│  │  Auth    │  │Collection│  │  Scanner  │  │Catalog   │    │
│  │  Module  │  │  Module  │  │  Module   │  │  Module  │    │
│  └──────────┘  └──────────┘  └───────────┘  └──────────     │
│                         │                                   │
│              ┌───────────────────┐                          │
│              │  Riverpod (State) │                          │
│              └───────────────────┘                          │
└───────────────────────┬─────────────────────────────────────┘
                        │ HTTP / SDK
        ┌───────────────┼───────────────────────┐
        │               │                       │
        ▼               ▼                       ▼
┌──────────────┐ ┌─────────────────┐ ┌──────────────────────┐
│   Supabase   │ │  Google ML Kit  │ │    IGDB API          │
│              │ │  (OCR on-device)│ │ (Metadados de Jogos) │
│ - Auth       │ └─────────────────┘ └──────────────────────┘
│ - PostgreSQL │         │                       │
│ - Storage    │         ▼                       ▼
│ - Edge Func  │ ┌─────────────────┐ ┌──────────────────────┐
└──────────────┘ │  Gemini Vision  │ │   TheGamesDB API     │
                 │  (identificação │ │   (Box Art / Capas)  │
                 │   de lombadas)  │ └──────────────────────┘
                 └─────────────────┘
```

### Padrão de Arquitetura: Feature-First + Clean Architecture lite

- **Presentation**: Widgets, Telas, Controllers (Riverpod)
- **Domain**: Models, UseCases, Interfaces de Repositório
- **Data**: Implementações de Repositório, Data Sources (Supabase, APIs)

---

## 4. Autenticação e Contas

### Provedores suportados (v1.0)

- E-mail + Senha (Supabase Auth)
- Google Sign-In (OAuth via Supabase)

### Fluxo de Cadastro

```
Splash → Onboarding (3 slides) → Login/Cadastro
         ↓
    Cadastro com e-mail
         ↓
    Verificação de e-mail (Supabase envia automaticamente)
         ↓
    Tela de configurar perfil (nome + foto)
         ↓
    Home (coleção vazia com CTA de primeiro scan)
```

### Dados do Perfil (Supabase: tabela `profiles`)

```sql
-- Tabela profiles (vinculada ao auth.users do Supabase)
create table profiles (
  id          uuid primary key references auth.users(id) on delete cascade,
  display_name text not null,
  photo_url   text,
  bio         text check (char_length(bio) <= 160),
  created_at  timestamptz default now(),
  total_games integer default 0
);
```

---

## 5. Algoritmo de Scan (OCR + IA)

Este é o coração do app. O objetivo é: **o usuário tira uma foto de lombadas/capas de jogos e o app identifica e registra automaticamente os títulos.**

### Pipeline de Identificação

```
┌─────────────────────────────────────────────────────────────┐
│                    PIPELINE DE SCAN                         │
│                                                             │
│  1. CAPTURA                                                 │
│     Câmera do dispositivo (camera package)                  │
│     ↓ Suporte a foto da galeria também                      │
│                                                             │
│  2. PRÉ-PROCESSAMENTO (on-device)                           │
│     - Redimensionar imagem (max 1920x1080)                  │
│     - Aumentar contraste e nitidez                          │
│     - Normalizar brilho                                     │
│     ↓ Pacote: image (dart)                                  │
│                                                             │
│  3. OCR — EXTRAÇÃO DE TEXTO (on-device)                     │
│     - Google ML Kit Text Recognition v2                     │
│     - Identifica blocos de texto na imagem                  │
│     - Extrai texto de lombadas, capas e contra-capas        │
│     ↓ Resultado: lista de strings com textos detectados     │
│                                                             │
│  4. FILTRO E NORMALIZAÇÃO DE TEXTO                          │
│     - Remove ruído (textos muito curtos < 3 chars)          │
│     - Remove textos genéricos ("PlayStation", "SONY", etc.) │
│     - Normaliza encoding (acentos, caracteres especiais)    │
│     ↓ Resultado: candidatos a nomes de jogos                │
│                                                             │
│  5. MATCHING COM CATÁLOGO LOCAL (cache Hive/SQLite)         │
│     - Fuzzy search contra lista de jogos do PS1             │
│     - Algoritmo: Levenshtein Distance + Jaro-Winkler        │
│     - Score de confiança por candidato                      │
│     ↓ Resultado: matches com score > 0.7                    │
│                                                             │
│  6. CONFIRMAÇÃO VIA API (para baixa confiança)              │
│     - Se score < 0.7: chama IGDB API para busca             │
│     - Fallback: Gemini Vision API (se OCR falhar)           │
│     ↓ Resultado: metadados completos do jogo                │
│                                                             │
│  7. TELA DE CONFIRMAÇÃO                                     │
│     - Mostra jogos identificados com foto/capa              │
│     - Usuário confirma, edita ou descarta cada item         │
│     ↓                                                       │
│  8. REGISTRO NA COLEÇÃO (Supabase PostgreSQL)               │
│     - Salva jogo confirmado na coleção do usuário           │
└─────────────────────────────────────────────────────────────┘
```

### Estratégia de Fallback (Gemini Vision)

Quando o OCR falha (lombada ilegível, foto torta, reflexo), o app pode enviar a imagem para o **Gemini Vision API** com o prompt:

```
"Esta é uma foto de jogos físicos de PlayStation 1.
Identifique os títulos dos jogos visíveis na imagem,
lendo lombadas e capas. Retorne apenas um JSON array
com os nomes dos jogos identificados, em ordem de confiança."
```

> **Custo:** Gemini 1.5 Flash é gratuito até 15 req/min — suficiente para v1.0.

### Tratamento de Casos Especiais (PS1)

- Jogos japoneses com kanji: usar ML Kit com modelo japonês
- Jogos com capa em degradê escura: ajuste de contraste antes do OCR
- Caixas sem lombada visível: analisar capa frontal inteira

---

## 6. Integração com APIs de Jogos

### API Principal: IGDB (Internet Game Database)

| Campo | Valor |
|---|---|
| URL | `https://api.igdb.com/v4` |
| Autenticação | Client ID + Bearer Token (via Twitch Developer) |
| Custo | Gratuito (até 4 req/segundo) |
| Plataforma PS1 | ID `7` |

#### Endpoints usados

```
POST /games
Body (Apicalypse):
  fields name, summary, cover.url, genres.name,
         first_release_date, involved_companies.company.name,
         involved_companies.developer, involved_companies.publisher,
         rating, aggregated_rating;
  where platforms = (7) & name ~ *"{{NOME_DO_JOGO}}"*;
  limit 5;

POST /covers
Body:
  fields url, game;
  where game = {{GAME_ID}};

POST /platforms
Body:
  fields id, name, abbreviation;
  where id = 7;
```

#### Cache Local (Hive)

Para evitar exceder o limite de rate e melhorar performance:
- Cache da lista completa de jogos PS1 (nome + ID) carregado na primeira abertura
- Cache de metadados individuais por 30 dias
- Atualização em background semanalmente

### API Secundária: TheGamesDB (Box Art Regional)

| Campo | Valor |
|---|---|
| URL | `https://api.thegamesdb.net/v1` |
| Autenticação | API Key gratuita |
| Custo | Gratuito (3.000 req/mês) |
| Plataforma PS1 | ID `10` |

Usado especificamente para buscar box art em diferentes regiões (NTSC-U, PAL, NTSC-J) quando o usuário quiser a capa regional do jogo que possui fisicamente.

### Fluxo de Busca no Catálogo

```
Usuário digita nome (busca manual)
          ↓
  Verifica cache local (Hive)
          ↓ (cache miss)
  Chama IGDB API /games
          ↓
  Salva resultado no cache
          ↓
  Exibe lista de resultados com capa
          ↓
  Usuário seleciona jogo correto
          ↓
  Adiciona à coleção (Supabase)
```

---

## 7. Banco de Dados

### Supabase (PostgreSQL) — Schema Completo

```sql
-- =============================================
-- EXTENSÕES NECESSÁRIAS
-- =============================================
create extension if not exists "pg_trgm";   -- busca fuzzy no catálogo
create extension if not exists "uuid-ossp"; -- geração de UUIDs

-- =============================================
-- TABELA: profiles
-- Dados públicos/extras do usuário
-- =============================================
create table profiles (
  id            uuid primary key references auth.users(id) on delete cascade,
  display_name  text not null,
  photo_url     text,
  bio           text check (char_length(bio) <= 160),
  total_games   integer default 0,
  created_at    timestamptz default now()
);

-- =============================================
-- TABELA: games_cache
-- Cache global de jogos do IGDB (compartilhado entre todos os usuários)
-- =============================================
create table games_cache (
  id            uuid primary key default uuid_generate_v4(),
  igdb_id       integer unique not null,
  title         text not null,
  platform      text not null,          -- 'ps1', 'ps2', etc.
  cover_url     text,
  summary       text,
  genres        text[],                 -- array de strings
  release_year  integer,
  developer     text,
  publisher     text,
  igdb_rating   numeric(4,2),
  cached_at     timestamptz default now()
);

-- Índice para busca fuzzy por título (usa pg_trgm)
create index games_cache_title_trgm_idx
  on games_cache using gin (title gin_trgm_ops);

-- Índice para filtrar por plataforma
create index games_cache_platform_idx on games_cache (platform);

-- =============================================
-- TABELA: collection
-- Jogos na coleção do usuário
-- =============================================
create table collection (
  id            uuid primary key default uuid_generate_v4(),
  user_id       uuid not null references auth.users(id) on delete cascade,
  igdb_id       integer not null,
  title         text not null,
  platform      text not null default 'ps1',
  cover_url     text,
  condition     text check (condition in ('mint','good','fair','poor')) default 'good',
  region        text check (region in ('NTSC-U','PAL','NTSC-J','other')) default 'NTSC-U',
  has_box       boolean default true,
  has_manual    boolean default true,
  added_via     text check (added_via in ('scan','manual')) default 'manual',
  user_photo_url text,                  -- foto do usuário no Supabase Storage
  added_at      timestamptz default now(),
  unique (user_id, igdb_id)             -- usuário não duplica o mesmo jogo
);

create index collection_user_id_idx on collection (user_id);
create index collection_platform_idx on collection (platform);

-- =============================================
-- TABELA: wishlist
-- Jogos que o usuário quer mas não tem
-- =============================================
create table wishlist (
  id         uuid primary key default uuid_generate_v4(),
  user_id    uuid not null references auth.users(id) on delete cascade,
  igdb_id    integer not null,
  title      text not null,
  platform   text not null default 'ps1',
  cover_url  text,
  added_at   timestamptz default now(),
  unique (user_id, igdb_id)
);

-- =============================================
-- ROW LEVEL SECURITY (RLS)
-- Cada usuário só acessa seus próprios dados
-- =============================================
alter table profiles   enable row level security;
alter table collection enable row level security;
alter table wishlist   enable row level security;
alter table games_cache enable row level security;

-- Policies: profiles
create policy "Usuário vê seu perfil"
  on profiles for select using (auth.uid() = id);
create policy "Usuário atualiza seu perfil"
  on profiles for update using (auth.uid() = id);

-- Policies: collection
create policy "Usuário gerencia sua coleção"
  on collection for all using (auth.uid() = user_id);

-- Policies: wishlist
create policy "Usuário gerencia sua wishlist"
  on wishlist for all using (auth.uid() = user_id);

-- Policies: games_cache (leitura pública para autenticados)
create policy "Qualquer usuário lê o cache"
  on games_cache for select using (auth.role() = 'authenticated');
create policy "Somente service_role escreve no cache"
  on games_cache for insert with check (auth.role() = 'service_role');

-- =============================================
-- TRIGGER: atualiza total_games automaticamente
-- =============================================
create or replace function update_total_games()
returns trigger language plpgsql as $$
begin
  update profiles
  set total_games = (
    select count(*) from collection where user_id = new.user_id
  )
  where id = new.user_id;
  return new;
end;
$$;

create trigger trg_update_total_games
  after insert or delete on collection
  for each row execute function update_total_games();
```

### Supabase Storage — Buckets

```
Bucket: avatars       (público)
  └── {user_id}/avatar.jpg

Bucket: game-photos   (privado, acessado via signed URL)
  └── {user_id}/{collection_item_id}.jpg
```

### Cache Local (Hive — on device)

```dart
// Boxes Hive — só para dados que precisam funcionar offline
HiveBox<GameModel>  'ps1_catalog'    // lista completa de jogos PS1 (nomes + igdb_id)
HiveBox<GameModel>  'game_details'   // detalhes individuais por igdb_id (30 dias)
HiveBox<String>     'api_tokens'     // token IGDB (renovado via Edge Function)
```

---

## 8. Telas e Fluxo do Usuário

### Mapa de Telas

```
SplashScreen
    ↓
OnboardingScreen (3 slides: scan, organize, explore)
    ↓
AuthScreen
  ├── LoginScreen
  └── RegisterScreen
        ↓
    ProfileSetupScreen
        ↓
    HomeScreen (BottomNav)
    ├── Tab 1: CollectionScreen
    │     ├── ConsoleTabBar (PS1, PS2... - só PS1 na v1.0)
    │     ├── GameGridView / GameListView
    │     └── GameDetailScreen
    │           ├── UserPhotoSection
    │           ├── MetadataSection (capa IGDB, gênero, ano, etc.)
    │           ├── ConditionSection (conservação, tem caixa/manual)
    │           └── EditGameScreen
    │
    ├── Tab 2: ScanScreen
    │     ├── CameraView (câmera ao vivo)
    │     ├── ScanResultsScreen
    │     │     └── ConfirmGameCard × N (jogos identificados)
    │     └── GameConfirmScreen (confirmar/editar antes de salvar)
    │
    ├── Tab 3: CatalogScreen
    │     ├── SearchBar
    │     ├── FilterChips (gênero, ano, region)
    │     ├── GameListView (todos os jogos do PS1)
    │     └── GameDetailScreen (modo catálogo, sem dados do usuário)
    │
    └── Tab 4: ProfileScreen
          ├── StatsCard (total jogos, wishlist, etc.)
          ├── WishlistSection
          └── SettingsScreen
```

### Wireframes Simplificados

#### HomeScreen / CollectionScreen
```
┌─────────────────────────────────┐
│  ColectorHub         [@] [🔔]   │
├─────────────────────────────────┤
│  [PS1] [PS2] [PS3]  ← tabs     │
│         (só PS1 na v1.0)       │
├─────────────────────────────────┤
│  123 jogos       [≡] [⊞]       │
│  ┌───────┐ ┌───────┐ ┌───────┐ │
│  │ CAPA  │ │ CAPA  │ │ CAPA  │ │
│  │       │ │       │ │       │ │
│  │FF VII │ │ GT    │ │ MGS   │ │
│  └───────┘ └───────┘ └───────┘ │
│  ┌───────┐ ┌───────┐ ┌───────┐ │
│  │ CAPA  │ │ CAPA  │ │ CAPA  │ │
│  └───────┘ └───────┘ └───────┘ │
├─────────────────────────────────┤
│  [Coleção] [Scan] [Catálogo] [Perfil] │
└─────────────────────────────────┘
```

#### ScanScreen
```
┌─────────────────────────────────┐
│  ← Escanear Jogos               │
├─────────────────────────────────┤
│                                 │
│   ┌─────────────────────────┐   │
│   │                         │   │
│   │    [VISOR DA CÂMERA]    │   │
│   │                         │   │
│   │   Mire para as          │   │
│   │   lombadas ou capas     │   │
│   │                         │   │
│   └─────────────────────────┘   │
│                                 │
│   ┌─────────────────────────┐   │
│   │  Dica: Boa iluminação   │   │
│   │  melhora o resultado    │   │
│   └─────────────────────────┘   │
│                                 │
│   [  GALERIA  ]  [  CAPTURAR  ] │
└─────────────────────────────────┘
```

#### ScanResultsScreen
```
┌─────────────────────────────────┐
│  ← Jogos Identificados (4)      │
├─────────────────────────────────┤
│  ┌─────────────────────────┐    │
│  │ [CAPA] Final Fantasy VII│[✓] │
│  │        PS1 | RPG | 1997 │    │
│  └─────────────────────────┘    │
│  ┌─────────────────────────┐    │
│  │ [CAPA] Metal Gear Solid │[✓] │
│  │        PS1 | Ação | 1998│    │
│  └─────────────────────────┘    │
│  ┌─────────────────────────┐    │
│  │ [?]  Não identificado   │[✗] │
│  │      "GRAN TURISM..."   │    │
│  └─────────────────────────┘    │
│  ┌─────────────────────────┐    │
│  │ [CAPA] Castlevania: SotN│[✓] │
│  │        PS1 | RPG | 1997 │    │
│  └─────────────────────────┘    │
│                                 │
│        [ADICIONAR 3 JOGOS]      │
└─────────────────────────────────┘
```

---

## 9. Estrutura de Pastas (Flutter)

```
lib/
├── main.dart
├── app.dart                          # MaterialApp, rotas, tema
│
├── core/
│   ├── constants/
│   │   ├── app_colors.dart
│   │   ├── app_strings.dart
│   │   └── api_constants.dart        # IGDB base URL, plataforma IDs
│   ├── errors/
│   │   ├── failures.dart
│   │   └── exceptions.dart
│   ├── network/
│   │   ├── api_client.dart           # Dio instance + interceptors
│   │   └── igdb_auth_interceptor.dart
│   └── utils/
│       ├── fuzzy_matcher.dart        # Levenshtein + Jaro-Winkler
│       ├── image_preprocessor.dart   # Ajuste contraste/brilho
│       └── text_filter.dart          # Remove ruído de OCR
│
├── features/
│   ├── auth/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   │   └── auth_remote_datasource.dart
│   │   │   └── repositories/
│   │   │       └── auth_repository_impl.dart
│   │   ├── domain/
│   │   │   ├── models/
│   │   │   │   └── user_model.dart
│   │   │   ├── repositories/
│   │   │   │   └── auth_repository.dart
│   │   │   └── usecases/
│   │   │       ├── sign_in_with_google.dart
│   │   │       ├── sign_in_with_email.dart
│   │   │       └── sign_out.dart
│   │   └── presentation/
│   │       ├── controllers/
│   │       │   └── auth_controller.dart
│   │       └── screens/
│   │           ├── login_screen.dart
│   │           ├── register_screen.dart
│   │           └── profile_setup_screen.dart
│   │
│   ├── collection/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   │   ├── collection_remote_datasource.dart   # Supabase
│   │   │   │   └── collection_local_datasource.dart    # Hive
│   │   │   └── repositories/
│   │   │       └── collection_repository_impl.dart
│   │   ├── domain/
│   │   │   ├── models/
│   │   │   │   └── collection_item_model.dart
│   │   │   ├── repositories/
│   │   │   │   └── collection_repository.dart
│   │   │   └── usecases/
│   │   │       ├── add_game_to_collection.dart
│   │   │       ├── remove_game_from_collection.dart
│   │   │       ├── get_user_collection.dart
│   │   │       └── update_game_details.dart
│   │   └── presentation/
│   │       ├── controllers/
│   │       │   └── collection_controller.dart
│   │       └── screens/
│   │           ├── collection_screen.dart
│   │           ├── game_detail_screen.dart
│   │           └── edit_game_screen.dart
│   │
│   ├── scanner/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   │   ├── ocr_datasource.dart          # ML Kit
│   │   │   │   └── vision_ai_datasource.dart    # Gemini Vision (fallback)
│   │   │   └── repositories/
│   │   │       └── scanner_repository_impl.dart
│   │   ├── domain/
│   │   │   ├── models/
│   │   │   │   └── scan_result_model.dart
│   │   │   ├── repositories/
│   │   │   │   └── scanner_repository.dart
│   │   │   └── usecases/
│   │   │       ├── scan_image.dart
│   │   │       └── confirm_scan_results.dart
│   │   └── presentation/
│   │       ├── controllers/
│   │       │   └── scanner_controller.dart
│   │       └── screens/
│   │           ├── scan_screen.dart
│   │           ├── scan_results_screen.dart
│   │           └── widgets/
│   │               └── scan_result_card.dart
│   │
│   └── catalog/
│       ├── data/
│       │   ├── datasources/
│       │   │   ├── igdb_datasource.dart
│       │   │   ├── thegamesdb_datasource.dart
│       │   │   └── catalog_cache_datasource.dart  # Hive
│       │   └── repositories/
│       │       └── catalog_repository_impl.dart
│       ├── domain/
│       │   ├── models/
│       │   │   └── game_model.dart
│       │   ├── repositories/
│       │   │   └── catalog_repository.dart
│       │   └── usecases/
│       │       ├── search_games.dart
│       │       ├── get_game_details.dart
│       │       └── get_ps1_catalog.dart
│       └── presentation/
│           ├── controllers/
│           │   └── catalog_controller.dart
│           └── screens/
│               ├── catalog_screen.dart
│               └── game_catalog_detail_screen.dart
│
└── shared/
    ├── widgets/
    │   ├── game_card.dart
    │   ├── game_cover_image.dart
    │   ├── loading_overlay.dart
    │   ├── empty_state_widget.dart
    │   └── error_widget.dart
    └── providers/
        └── providers.dart            # todos os Riverpod providers
```

---

## 10. Fases de Desenvolvimento

### Fase 1 — Fundação (Semanas 1-2)
- [ ] Setup do projeto Flutter (flutter create, estrutura de pastas)
- [ ] Setup Supabase (projeto, tabelas SQL, buckets, RLS)
- [ ] Configurar Riverpod
- [ ] Implementar tema visual (cores, tipografia, dark mode)
- [ ] Tela de Splash + Onboarding
- [ ] Autenticação: login e cadastro com e-mail (Supabase Auth)
- [ ] Autenticação: Google Sign-In (OAuth Supabase)
- [ ] Tela de configuração de perfil (insert em `profiles`)
- [ ] Roteamento base (go_router)

### Fase 2 — Catálogo e Coleção Manual (Semanas 3-4)
- [ ] Integração com IGDB API (client + auth + cache token)
- [ ] Carregar catálogo de jogos PS1 no Hive (cache local)
- [ ] Tela de catálogo com busca
- [ ] Detalhes do jogo (metadados + capa)
- [ ] Adicionar jogo manualmente à coleção
- [ ] Tela de coleção (grid/lista)
- [ ] Tela de detalhes do item na coleção
- [ ] Editar item (condição, tem caixa, foto)
- [ ] Remover item da coleção
- [ ] Wishlist básica

### Fase 3 — Scanner (Semanas 5-6)
- [ ] Implementar CameraView com câmera live
- [ ] Integrar Google ML Kit Text Recognition
- [ ] Implementar pré-processamento de imagem (contraste/brilho)
- [ ] Implementar filtro de ruído de OCR
- [ ] Implementar fuzzy matching (Levenshtein + Jaro-Winkler)
- [ ] Tela de resultados de scan com confirmação
- [ ] Salvar foto do usuário no Supabase Storage (bucket game-photos)
- [ ] Fallback com Gemini Vision API
- [ ] Suporte a foto da galeria além da câmera

### Fase 4 — Polimento e QA (Semana 7)
- [ ] Tela de perfil com estatísticas
- [ ] Animações e transições
- [ ] Tratamento de erros com feedback ao usuário
- [ ] Loading states e skeletons
- [ ] Testes de integração básicos
- [ ] Testes manuais do scanner (diversas condições de iluminação)
- [ ] Ajustes de UX baseados em testes
- [ ] Build de produção e checklist de publicação

---

## 11. Dependências e Pacotes

```yaml
# pubspec.yaml

dependencies:
  flutter:
    sdk: flutter

  # Estado
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5

  # Navegação
  go_router: ^14.2.0

  # Supabase (substitui firebase_core + firebase_auth + cloud_firestore + firebase_storage)
  supabase_flutter: ^2.5.0

  # Câmera e Imagem
  camera: ^0.11.0
  image_picker: ^1.1.2
  image: ^4.2.0                     # pré-processamento
  flutter_image_compress: ^2.3.0

  # OCR e IA
  google_mlkit_text_recognition: ^0.13.1
  google_generative_ai: ^0.4.4      # Gemini Vision (fallback)

  # HTTP
  dio: ^5.6.0

  # Cache local
  hive_flutter: ^1.1.0

  # Fuzzy Search
  string_similarity: ^2.0.0         # Jaro-Winkler
  # ou: fuzzywuzzy (alternativa com Levenshtein)

  # UI
  cached_network_image: ^3.3.1
  shimmer: ^3.0.0
  lottie: ^3.1.0                    # animações
  flutter_svg: ^2.0.10

  # Utilitários
  intl: ^0.19.0
  equatable: ^2.0.5
  logger: ^2.4.0
  package_info_plus: ^8.0.2

dev_dependencies:
  flutter_test:
    sdk: flutter
  riverpod_generator: ^2.4.3
  build_runner: ^2.4.12
  hive_generator: ^2.0.1
  flutter_lints: ^4.0.0
  mockito: ^5.4.4
```

---

## 12. Requisitos Não-Funcionais

### Performance
- Scan deve retornar resultados em no máximo **5 segundos** (condições normais)
- Catálogo deve carregar com busca em tempo real (debounce de 300ms)
- Cache local deve permitir navegação offline na coleção já carregada

### Segurança
- Token IGDB nunca exposto no client — renovação via **Supabase Edge Function** (Deno)
- **Row Level Security (RLS)** no PostgreSQL: usuário só acessa seus próprios registros
- Imagens armazenadas em bucket privado `game-photos`, acessadas via signed URL temporária
- Chaves do Supabase: `anon key` no app (segura com RLS), `service_role key` apenas nas Edge Functions

```sql
-- Exemplo da RLS em ação: mesmo que o usuário tente burlar o client,
-- o PostgreSQL bloqueia no nível do banco de dados
-- policy "Usuário gerencia sua coleção":
create policy "Usuário gerencia sua coleção"
  on collection for all
  using (auth.uid() = user_id);
-- auth.uid() é resolvido automaticamente pelo Supabase com o JWT do usuário
```

### UX
- Dark mode por padrão (estética retrô gaming)
- Suporte a acessibilidade (semântica, contraste mínimo WCAG AA)
- Feedback visual em todas as ações async (loading, sucesso, erro)

---

## 13. Roadmap Futuro (pós v1.0)

| Versão | Feature |
|---|---|
| v1.1 | Suporte a PS2 |
| v1.2 | Suporte a PS3 e PSP |
| v2.0 | Multi-fabricante: Nintendo (SNES, N64, GBA) |
| v2.1 | Valor de mercado estimado (integração eBay/Mercado Livre) |
| v2.2 | Perfil público da coleção com link compartilhável |
| v3.0 | Social: seguir colecionadores, feed de aquisições |
| v3.1 | Marketplace interno: trocar ou vender entre usuários |
| v4.0 | Scan por código de barras / QR Code |

---

## Referências e Links

| Recurso | URL |
|---|---|
| IGDB API Docs | https://api-docs.igdb.com |
| IGDB via Twitch Dev | https://dev.twitch.tv/console |
| TheGamesDB API | https://api.thegamesdb.net |
| Google ML Kit Flutter | https://pub.dev/packages/google_mlkit_text_recognition |
| Gemini API Flutter | https://pub.dev/packages/google_generative_ai |
| Supabase Flutter | https://supabase.com/docs/reference/dart/introduction |
| Supabase Dashboard | https://supabase.com |
| Riverpod | https://riverpod.dev |
| go_router | https://pub.dev/packages/go_router |

---

*Documento criado em: 23/02/2026*
*Versão do planejamento: 1.1 — migrado Firebase → Supabase*
*Status: Aprovado*
