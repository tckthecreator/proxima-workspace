# Client State Machine

Este documento descreve a **máquina de estados do client** responsável por controlar presença, mídia, colaboração e conectividade. Ele traduz ZoneTypes e regras em **comportamento executável**, garantindo previsibilidade, performance e resiliência.

---

## Objetivos

* Definir estados claros do usuário
* Padronizar transições entre ZoneTypes
* Controlar abertura/fechamento de mídia
* Integrar política P2P / TURN / SFU
* Evitar vazamento de recursos

---

## Princípios

* Um estado ativo por usuário
* Transições explícitas e atômicas
* Mídia só existe quando o estado permite
* Falhas retornam a estados seguros

---

## Estados Principais

### 1. Disconnected

Estado inicial ou após falha crítica.

**Características**:

* Sem signaling
* Sem mídia
* UI mínima

**Transições**:

* `connect()` → Connecting

---

### 2. Connecting

Estabelecimento de conexão com o signaling server.

**Ações**:

* Autenticação
* Sync inicial de mundo e salas

**Transições**:

* sucesso → Idle
* falha → Disconnected

---

### 3. Idle

Usuário conectado, sem mídia ativa.

**Características**:

* Presença ativa
* Movimentação no mundo
* Sem WebRTC

**Transições**:

* `enterZone(OpenArea)` → OpenArea
* `enterZone(AFK)` → AFK

---

### 4. OpenArea

Circulação com áudio por proximidade.

**Ações**:

* Inicializar áudio P2P local
* Atualizar distância entre peers

**Transições**:

* `enterZone(PersonalRoom)` → PersonalRoom
* `enterZone(TeamArea)` → TeamArea
* `enterZone(MeetingRoom)` → MeetingRoom
* `enterZone(AFK)` → AFK

---

### 5. PersonalRoom

Sala individual controlada pelo usuário.

**Ações**:

* Abrir áudio
* Vídeo opcional
* Screen share sob demanda

**Transições**:

* `inviteUser()` (permanece)
* `leaveZone()` → Idle
* `enterZone(AFK)` → AFK

---

### 6. TeamArea

Área colaborativa de time.

**Ações**:

* Abrir áudio grupo
* Negociar SFU se necessário
* Whiteboard ativo

**Transições**:

* `enterZone(MeetingRoom)` → MeetingRoom
* `leaveZone()` → Idle

---

### 7. MeetingRoom

Reunião estruturada.

**Ações**:

* Conectar via SFU
* Áudio ativo
* Vídeo opcional
* Screen share permitido

**Transições**:

* `lockRoom()` (permanece)
* `leaveZone()` → Idle

---

### 8. AFK

Usuário indisponível.

**Ações**:

* Encerrar todas as mídias
* Suspender notificações

**Transições**:

* `return()` → Idle

---

## Estados de Mídia (Subestados)

Estados de mídia são **ortogonais** ao estado principal.

### AudioState

* Off
* P2P
* SFU

### VideoState

* Off
* Camera
* ScreenShare

### CollaborationState

* None
* Whiteboard

---

## Política de Conectividade

A decisão de conectividade ocorre em dois momentos:

1. Entrada em ZoneType
2. Mudança dinâmica (ex: mais participantes)

**Critérios**:

* ZoneType
* Número de peers
* ICE failures
* CPU/banda

---

## Tratamento de Falhas

* Falha de mídia → desligar track
* Falha de SFU → fallback P2P (quando possível)
* Falha geral → Disconnected

---

## Observabilidade

O client deve emitir eventos:

* state_change
* media_started
* media_stopped
* connectivity_changed

Esses eventos alimentam logs e métricas.

---

## Evolução Futura

* Estados customizados por empresa
* Plugins de estados
* Automação baseada em contexto

---

## Nota Final

Esta máquina de estados é o **núcleo comportamental do client**.
Qualquer feature nova deve:

* Declarar quais estados afeta
* Definir transições válidas
* Respeitar regras de ZoneType
