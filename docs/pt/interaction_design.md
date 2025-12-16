# Interaction Design – Escritórios Virtuais e Zonas

Este documento define o **design de interação** da aplicação: como usuários se comportam, se comunicam e colaboram dentro do mundo virtual.

Ele substitui a noção clássica de *game design* por algo mais apropriado ao domínio do produto: **regras sociais, estados de presença e colaboração em tempo real**.

Este arquivo é uma **fonte normativa**: o comportamento do sistema deve seguir o que está descrito aqui.

---

## 1. Conceito Central

O mundo virtual é composto por **Zonas** (*ZoneType*). Cada zona impõe um conjunto de regras claras de comunicação, privacidade e colaboração.

O layout visual é secundário. **O comportamento emerge do tipo da zona, não da posição no mapa.**

---

## 2. ZoneType

Cada zona possui:

* regras de áudio
* regras de vídeo
* regras de entrada
* regras de visibilidade
* permissões de colaboração

As regras possuem **valores padrão**, mas podem ser **sobrescritas** na configuração da sala.

---

## 3. Tipos de Zona

### 3.1 OPEN (Aberto)

**Função social**

* Circulação livre
* Interações espontâneas
* Presença passiva

**Regras padrão**

* Áudio: proximidade
* Vídeo: desligado
* Entrada: livre
* Screen share: desabilitado
* Ferramentas colaborativas: desabilitadas

**Observações**

* Zona padrão ao entrar no mundo
* Ideal para P2P + sharding espacial

---

### 3.2 PERSONAL (Sala Pessoal)

**Função social**

* Trabalho individual
* Conversas privadas
* Interrupções controladas

**Regras padrão**

* Entrada: somente com permissão
* Áudio: apenas entre presentes
* Vídeo: opcional
* Screen share: opcional
* Whiteboard: opcional

**Funcionalidades**

* Pedido de entrada (knock)
* Controle explícito de permissões

---

### 3.3 TEAM_AREA (Área do Time)

**Função social**

* Comunicação recorrente
* Coordenação de equipe
* Visibilidade coletiva

**Regras padrão**

* Entrada: membros do time
* Áudio: grupo
* Vídeo: opcional
* Chamada rápida: habilitada
* Screen share: opcional

**Funcionalidades**

* Iniciar chamada rápida sem agendamento
* Visualização de presença do time

---

### 3.4 MEETING (Sala de Reunião)

**Função social**

* Reuniões formais
* Discussões focadas
* Tomada de decisão

**Regras padrão**

* Entrada: livre ou restrita (configurável)
* Áudio: conectado automaticamente
* Vídeo: opcional
* Screen share: habilitado
* Whiteboard: habilitado

**Comportamento**

* Entrar na zona conecta automaticamente na reunião
* Possibilidade de fechar a sala
* Integração direta com SFU quando necessário

---

### 3.5 AFK

**Função social**

* Ausência explícita
* Não perturbe

**Regras padrão**

* Áudio: desligado
* Vídeo: desligado
* Screen share: desabilitado
* Notificações: silenciadas

**Comportamento**

* Usuário não participa de shards
* Estado visível para outros usuários

---

## 4. Override de Regras

Cada instância de zona pode sobrescrever regras padrão.

### Exemplos

* Sala pessoal sem vídeo permanentemente
* Área do time apenas com áudio push-to-talk
* Sala de reunião com vídeo forçado desligado

Overrides são definidos via configuração da sala, não no código.

---

## 5. Ferramentas de Colaboração

### 5.1 Screen Sharing

Disponível em:

* Sala Pessoal
* Área do Time
* Sala de Reunião

Implementação:

* WebRTC Screen Capture
* Compartilhamento restrito aos participantes da zona

---

### 5.2 Whiteboard Colaborativo

Ferramenta nativa para:

* desenho livre
* texto
* cursores simultâneos

Características:

* Sincronização em tempo real
* Estado efêmero por padrão
* Pode ser salvo/exportado

---

### 5.3 Integrações Externas

Integrações não substituem ferramentas externas, apenas **as trazem para dentro do contexto da zona**.

Exemplos:

* Slack (threads, notificações)
* GitHub / GitLab (PRs, issues, commits)
* Google Docs
* Draw.io

Integrações aparecem como:

* painéis embutidos
* links contextuais
* ações rápidas

---

## 6. Estados do Usuário

Estados explícitos:

* Online
* Em reunião
* AFK
* Não perturbe

Estados impactam:

* notificações
* visibilidade
* mídia

---

## 7. Princípios de UX

* Nenhuma ação deve surpreender o usuário
* Entrar em uma zona define expectativas claras
* Vídeo nunca deve ser forçado por padrão
* Presença é mais importante que câmera

---

## 8. Relação com Arquitetura

Cada ZoneType mapeia diretamente para:

* política de mídia
* política P2P / SFU
* sharding espacial

Este documento complementa `ARCHITECTURE_GUIDE.md`.

---

## 9. Evolução do Documento

Este documento deve evoluir junto com:

* feedback de usuários
* limitações técnicas
* novos modos de colaboração

---

## 10. Objetivo Final

Criar um ambiente virtual que:

* respeite foco e contexto
* reduza fricção social
* aumente colaboração
* seja previsível e humano

---

Este documento é deliberadamente mais conceitual que técnico, mas define regras que **o código deve respeitar**.
