# Collaboration Tools

Este documento define as **ferramentas de colaboração** suportadas pela aplicação, seus objetivos, limites de escopo e decisões arquiteturais. O foco é viabilizar trabalho síncrono eficiente sem transformar o produto em uma plataforma genérica ou excessivamente complexa.

---

## Objetivos

* Permitir colaboração em tempo real dentro do escritório virtual
* Integrar ferramentas já consolidadas no ecossistema de tecnologia
* Evitar duplicar funcionalidades complexas de terceiros
* Manter baixo acoplamento e fácil self-hosting

---

## Screen Share

### Funcionalidade

* Compartilhamento de tela individual
* Compartilhamento de janela específica
* Compartilhamento de aba do navegador (quando suportado)

### Regras

* Vídeo da câmera é **opcional**
* Apenas **um screen share ativo por usuário**
* Em ZoneTypes de reunião, múltiplos usuários podem compartilhar simultaneamente

### Implementação

* WebRTC `getDisplayMedia`
* Tratado como um *media track separado*
* Pode ser roteado via:

  * P2P (salas pequenas)
  * SFU (salas médias/grandes)

### Fora de escopo

* Gravação de tela
* Controle remoto
* Anotações diretas sobre o vídeo

---

## Whiteboard

### Objetivo

Permitir desenho, escrita e esquemas rápidos durante discussões, sem competir com ferramentas completas de design.

### Funcionalidades suportadas

* Canvas compartilhado
* Desenho livre
* Texto simples
* Apagar / limpar quadro
* Cursor colaborativo

### Modelo técnico

* Canvas 2D
* Sincronização via eventos (não via vídeo)
* Estado eventual (event sourcing leve)

### Persistência

* Opcional por ZoneType
* Pode ser descartável (default)

### Fora de escopo

* Versionamento avançado
* Templates complexos
* Exportação profissional

---

## Integrações Externas

### Princípio geral

> Integrar, não substituir.

A aplicação **não replica** funcionalidades complexas já resolvidas por outras plataformas.

---

### Slack / Discord

* Webhooks de presença
* Notificações de reuniões iniciadas
* Deep links para salas

Fora de escopo:

* Chat bidirecional completo

---

### GitHub / GitLab

* Visualização de PRs/Issues
* Notificações de eventos relevantes
* Links contextuais por sala ou time

Fora de escopo:

* Edição de código
* Gestão de repositório

---

### Google Docs / Notion / Confluence

* Embed via iframe (quando permitido)
* Compartilhamento de links contextualizados

Fora de escopo:

* Edição nativa
* Autenticação delegada complexa

---

### Draw.io / Diagramas

* Embed externo
* Links persistentes por sala

Fora de escopo:

* Editor gráfico próprio avançado

---

## Limites de Escopo (Anti Feature Creep)

Funcionalidades explicitamente **fora do produto**:

* Editor de código colaborativo
* Substituto de Slack/Teams
* Sistema de tickets
* Gravação automática de reuniões
* Transcrição automática
* Armazenamento de arquivos genérico

Esses recursos devem ser consumidos via integração, não implementação.

---

## Princípios de Design

* Preferir **embeds e links inteligentes**
* Cada ferramenta deve ser opcional
* ZoneType define permissões e disponibilidade
* Tudo deve funcionar em ambiente self-hosted

---

## Roadmap sugerido

### MVP

* Screen share
* Whiteboard básico
* Links externos

### Iteração 2

* Notificações externas
* Persistência opcional

### Iteração futura

* Automação via webhooks
* Plugins

---

## Nota final

Este documento existe para **proteger o produto**.
Toda nova feature de colaboração deve responder claramente:

* Isso integra ou substitui algo?
* Isso escala?
* Isso é essencial para colaboração síncrona?
