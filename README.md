# Proxima Workspace — Presence-Driven Virtual Office

Este repositório contém o design e a implementação de uma plataforma de **escritório virtual colaborativo**, com foco em comunicação em tempo real, presença espacial, previsibilidade de comportamento, performance e fácil self-hosting via Docker.

O projeto é desenhado inicialmente para uso interno em empresas e, posteriormente, para distribuição open source.

---

## Objetivos do Projeto

* Escritório virtual com presença espacial (avatars, proximidade, ZoneTypes)
* Comunicação por áudio/vídeo em tempo real
* Colaboração síncrona: screen share, whiteboard
* Escalabilidade progressiva (P2P → TURN → SFU)
* Self-host simples e padronizado via Docker
* Documentação como contrato de arquitetura

---

## Vocabulário Oficial (Congelado)

Os termos abaixo são **normativos**. Toda a base de código e documentação deve utilizá-los exatamente como definidos.

### ZoneType

Contexto funcional declarativo que governa:

* regras de áudio e vídeo
* ferramentas de colaboração
* política de conectividade

Exemplos: `OpenArea`, `PersonalRoom`, `TeamArea`, `MeetingRoom`, `AFK`

---

### Client

Aplicação executada pelo usuário final.

Inclui:

* renderização do mundo
* máquina de estados
* mídia WebRTC

Nunca utilizar termos alternativos como *frontend*, *app* ou *player*.

---

### Signaling Server

Serviço responsável por:

* presença
* estado
* troca de mensagens
* negociação de conexões WebRTC

Não transmite mídia.

---

### SFU (Selective Forwarding Unit)

Servidor de mídia responsável por receber streams WebRTC e encaminhá-los seletivamente para outros participantes, sem decodificação ou mixagem.

---

## Estrutura de Documentação

A documentação do projeto é **parte essencial do design** e deve ser lida na ordem abaixo.

### 1. Visão Geral de Arquitetura

* **ARCHITECTURE_GUIDE.md** — Define arquitetura global, princípios de self-hosting e componentes principais.

### 2. Mundo e Presença

* **WORLD_SYSTEM.md** — Filosofia do mundo virtual e limites conceituais.
* **SPATIAL_MODEL.md** — Modelo de espaço, proximidade, visibilidade, sharding.
* **AVATAR_SYSTEM.md** — Definição de avatars, estados e indicadores visuais.
* **MAP_FORMAT.md** — Formato de mapas, escritórios prontos e customizados.

### 3. Design de Interação

* **INTERACTION_DESIGN.md** — Como usuários se movimentam, interagem e transitam entre contextos.

### 4. Ferramentas de Colaboração

* **COLLABORATION_TOOLS.md** — Screen share, whiteboard, integrações externas e limites de escopo.

### 5. Regras por Zona

* **ZONE_RULES.md** — Regras comportamentais, permissões e políticas de mídia por ZoneType.

### 6. Protocolo de Signaling

* **SIGNALING_PROTOCOL.md** — Contrato de mensagens entre Client e Signaling Server.

### 7. Máquina de Estados do Client

* **CLIENT_STATE_MACHINE.md** — Traduz ZoneTypes e regras em comportamento executável no Client.

### 8. Modos de Falha

* **FAILURE_MODES.md** — Falhas esperadas, degradação graciosa e estratégias de recuperação.

---

## Princípios Fundamentais

* ZoneType governa comportamento
* Mídia nunca é implícita
* Performance tem prioridade sobre customização
* Integração é preferível à reinvenção
* Docker é o contrato operacional

---

## Self-Hosting

Todo o ambiente self-hosted deve ser executado exclusivamente via Docker.

Não há suporte para:

* instalação manual
* execução direta de binários
* setups não containerizados

---

## Contribuições

Este projeto valoriza contribuições que:

* respeitem o vocabulário oficial
* mantenham consistência arquitetural
* evitem feature creep

Toda nova funcionalidade deve declarar explicitamente:

* ZoneTypes afetados
* impacto em mídia
* impacto em performance

---

## Status do Projeto

* Design: consolidado
* Documentação: estruturante
* Implementação: em progresso

---

## Nota Final

Este repositório trata documentação como **fonte da verdade**.

Código que viola decisões documentadas deve ser considerado incorreto por definição.
