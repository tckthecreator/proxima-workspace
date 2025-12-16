# Arquitetura do Sistema – Escritório Virtual P2P com Escala Progressiva

Este documento define a arquitetura de referência do projeto, servindo como **guia técnico**, **fonte de decisão** e **base para contribuição open source**.

O objetivo do sistema é oferecer um escritório virtual 2D com comunicação em tempo real, priorizando **baixo custo**, **simplicidade operacional** e **escala progressiva**, começando em P2P puro e evoluindo para SFU apenas quando necessário.

---

## 1. Princípios de Arquitetura

1. **P2P First** – comunicação direta sempre que possível.
2. **Fallback Inteligente** – TURN apenas para conectividade, SFU apenas para escala.
3. **Escala Progressiva** – o sistema cresce conforme a necessidade.
4. **Client-centric** – o servidor não é dono do estado do mundo.
5. **Self-host Friendly** – instalação simples via Docker.
6. **Open Source Ready** – stack acessível e extensível.

---

## 2. Componentes do Sistema

### 2.1 Client Web

Responsável por renderização, interação e mídia.

**Tecnologias sugeridas:**

* PixiJS (render 2D)
* WebRTC (áudio, vídeo e dados)
* Web Audio API
* React (UI fora do canvas)
* Zustand (estado local)

Funções principais:

* Renderizar o mundo
* Detectar proximidade espacial
* Gerenciar conexões WebRTC
* Coletar métricas de performance

---

### 2.2 Servidor de Signaling (Go)

Responsável apenas por **coordenação**, nunca por mídia.

Funções:

* Descoberta de peers
* Troca de SDP / ICE
* Autenticação básica
* Estado mínimo da sala
* Decisão P2P vs SFU

O servidor de signaling **não mantém estado persistente do mundo**.

---

### 2.3 TURN Server (Opcional)

Utilizado exclusivamente como **fallback de conectividade**.

Características:

* Relay de pacotes
* Não entende mídia
* Não escala salas

Exemplos:

* coturn (self-hosted)

---

### 2.4 SFU (Opcional)

Ativado apenas quando o modelo P2P se torna ineficiente.

Funções:

* Receber streams WebRTC
* Encaminhar seletivamente
* Reduzir carga do client

Características:

* Não decodifica mídia
* Não mistura áudio
* Pode usar simulcast

---

## 3. Estados de Operação da Sala

Cada sala opera em exatamente um estado:

* `P2P_DIRECT`
* `P2P_TURN`
* `SFU`

A transição é automática e baseada em métricas.

---

## 4. Política Automática de Decisão

### 4.1 Fluxo Inicial

1. Tentativa de P2P direto
2. Fallback individual para TURN
3. Observação de métricas globais

---

### 4.2 Promoção para SFU

A sala migra para SFU quando qualquer condição for verdadeira:

* Participantes > 12
* CPU média > 70%
* > 30% dos peers usando TURN
* Packet loss > 5%

---

### 4.3 Migração

A migração ocorre **sem derrubar a sala**:

1. Conexão SFU paralela
2. Redirecionamento de mídia
3. Encerramento P2P

---

## 5. Sharding Espacial

O mundo é dividido em regiões lógicas.

Efeitos:

* Conexões WebRTC apenas com peers próximos
* Áudio/vídeo desativado fora do shard
* Menor consumo de CPU e banda

O sharding funciona tanto em P2P quanto em SFU.

---

## 6. Configuração e Deploy (Self-hosted)

Toda a parte **self-hosted** do sistema é projetada para funcionar **exclusivamente via Docker**.

Não há suporte oficial para instalação manual de binários, serviços de sistema ou configurações fora de containers.

### 6.1 Princípios de Deploy

* Todos os componentes rodam em containers
* Configuração via arquivos e variáveis de ambiente
* Nenhuma dependência externa obrigatória
* Atualização via pull de imagem

---

### 6.2 Componentes Dockerizados

Os seguintes serviços possuem imagens Docker oficiais:

* Signaling Server
* SFU (opcional)
* TURN (opcional, recomendado via coturn)

---

### 6.3 Arquivo de Configuração

Toda a política de operação é controlada via `config.yaml`, montado como volume no container.

O cliente **não precisa alterar código** para:

* ativar SFU
* ativar TURN
* limitar escala

---

### 6.4 Orquestração

A forma recomendada de execução é via `docker-compose`.

Para ambientes maiores, Kubernetes é suportado, mas não obrigatório.

---

## 7. Limitações Conhecidas

* P2P não escala acima de ~12 usuários por sala
* TURN não resolve problemas de escala
* SFU aumenta custo operacional
* Estado do mundo é efêmero

---

## 8. Roadmap Técnico (Resumo)

### Fase 1 – MVP

* Mundo 2D simples
* P2P direto
* Signaling básico

### Fase 2 – Robustez

* TURN opcional
* Métricas
* Política automática

### Fase 3 – Escala

* SFU
* Simulcast
* Sharding espacial avançado

---

## 9. Público-alvo

* Pequenas empresas
* Times remotos
* Comunidades open source

---

## 10. Objetivo Final

Entregar um sistema que:

* Funcione sem custo inicial
* Escale quando necessário
* Seja transparente tecnicamente
* Seja fácil de operar e contribuir

---

Este documento é **vivo** e deve evoluir junto com o projeto.
