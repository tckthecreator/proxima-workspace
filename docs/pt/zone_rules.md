# Zone Rules

Este documento define as **regras comportamentais, técnicas e de permissões** associadas a cada *ZoneType* do escritório virtual. O objetivo é garantir previsibilidade, performance e coerência de produto, permitindo overrides controlados quando necessário.

---

## Conceito de ZoneType

Um **ZoneType** representa um contexto funcional dentro do escritório.
Ele controla:

* Áudio
* Vídeo
* Screen share
* Whiteboard
* Integrações
* Política de conexão (P2P / SFU)

ZoneTypes são **declarativos** e podem ser reutilizados em diferentes layouts.

---

## ZoneTypes Padrão

### 1. Open Area

Contexto padrão de circulação no mundo.

**Regras padrão**:

* Áudio: proximidade
* Vídeo: desativado
* Screen share: desativado
* Whiteboard: indisponível
* Integrações: links simples

**Conectividade**:

* P2P prioritário
* Sem SFU

**Objetivo**:

* Socialização leve
* Baixo custo computacional

---

### 2. Personal Room

Espaço individual de trabalho focado.

**Regras padrão**:

* Áudio: apenas participantes autorizados
* Vídeo: opcional
* Screen share: permitido
* Whiteboard: opcional
* Integrações: permitidas

**Conectividade**:

* P2P por padrão
* SFU opcional acima de limite configurável

**Objetivo**:

* Conversas pontuais
* Revisões rápidas

---

### 3. Team Area

Área compartilhada por um time.

**Regras padrão**:

* Áudio: grupo
* Vídeo: opcional
* Screen share: permitido
* Whiteboard: permitido
* Integrações: ativas

**Conectividade**:

* SFU recomendado

**Objetivo**:

* Alinhamento rápido
* Colaboração contínua

---

### 4. Meeting Room

Espaço dedicado a reuniões formais.

**Regras padrão**:

* Áudio: ativo ao entrar
* Vídeo: opcional
* Screen share: permitido
* Whiteboard: permitido
* Integrações: ativas

**Conectividade**:

* SFU obrigatório

**Objetivo**:

* Reuniões estruturadas
* Escalabilidade

---

### 5. AFK Zone

Área de ausência temporária.

**Regras padrão**:

* Áudio: desativado
* Vídeo: desativado
* Screen share: desativado
* Whiteboard: indisponível
* Integrações: notificações suspensas

**Conectividade**:

* Nenhuma mídia ativa

**Objetivo**:

* Reduzir ruído
* Sinalizar indisponibilidade

---

## Overrides de Regras

ZoneTypes permitem **override explícito** de regras.

### Exemplos

* Open Area com vídeo habilitado
* Personal Room sem screen share
* Meeting Room com acesso restrito

Overrides devem ser:

* Declarativos
* Visíveis ao usuário
* Registrados no estado da sala

---

## Política de Conectividade

A escolha entre P2P, TURN e SFU segue critérios automáticos:

* Número de participantes
* ZoneType
* Qualidade de rede
* Capacidade do client

A decisão é transparente para o usuário.

---

## Princípios

* ZoneType define o padrão, não a exceção
* Overrides existem, mas não quebram o modelo mental
* Performance tem prioridade sobre customização extrema

---

## Evolução Futura

* ZoneTypes customizados por empresa
* Templates de regras
* Analytics por ZoneType

---

## Nota final

ZoneTypes são o elo entre **game design**, **arquitetura** e **produto**.
Toda nova funcionalidade deve declarar claramente:

* Em quais ZoneTypes se aplica
* Quais regras altera
* Qual impacto de performance gera
