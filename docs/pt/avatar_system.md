# Avatar System

Este documento descreve o **Avatar System** — a representação lógica do usuário no mundo — incluindo estados, sincronização e limites intencionais. O avatar é uma **entidade de presença**, não um personagem de jogo.

---

## Relações

Este documento se relaciona com:

* WORLD_SYSTEM.md
* SPATIAL_MODEL.md
* ZONE_RULES.md
* CLIENT_STATE_MACHINE.md

---

## Objetivos do Avatar System

O Avatar System existe para:

* Representar a presença de um usuário no mundo
* Comunicar estados relevantes (falando, AFK, ocupado)
* Integrar-se ao Spatial Model e ZoneType
* Ser leve, previsível e escalável

Ele **não** existe para:

* Customização estética profunda
* Progressão, stats ou gameplay
* Executar regras de mídia ou permissão

---

## Avatar como Entidade Lógica

Um avatar é composto por:

* Identidade (userId)
* Posição espacial
* Estado atual
* Indicadores de presença

O avatar **não** contém:

* Lógica de mídia
* Regras de ZoneType
* Estados persistentes complexos

---

## Estados do Avatar

Estados são mutuamente exclusivos.

### Estados Principais

* `Idle` — parado, disponível
* `Moving` — em deslocamento
* `Speaking` — áudio ativo
* `AFK` — ausente
* `Busy` — não interromper

Estados são **informativos**, não autoritativos.

---

## Indicadores Visuais

Indicadores devem ser:

* Claros
* Discretos
* Consistentes

Exemplos:

* Ícone de microfone ativo
* Indicador de câmera
* Status AFK

Indicadores **não** substituem regras de ZoneType.

---

## Atualização e Sincronização

### Frequência

* Atualizações de estado são event-driven
* Atualizações de posição seguem o Spatial Model

Evitar broadcast contínuo.

---

## Prioridade de Dados

Em ordem de importância:

1. Presença (online/offline)
2. ZoneType atual
3. Estado do avatar
4. Posição

Se necessário, dados menos importantes são descartados.

---

## Relação com ZoneType

* ZoneType define permissões
* Avatar apenas reflete estado

Exemplo:

* Avatar pode mostrar microfone ativo
* ZoneType decide se áudio é transmitido

---

## Limites Intencionais

* Número máximo de avatares visíveis
* Número máximo de atualizações por segundo
* Estados finitos e pré-definidos

Customização visual é propositalmente limitada.

---

## Falhas e Degradação

Em caso de falha:

* Avatar entra em estado seguro (`Idle` ou `AFK`)
* Indicadores são resetados
* Mídia é encerrada

Referência: FAILURE_MODES.md

---

## Anti-Objetivos

Explicitamente fora de escopo:

* Skins dinâmicas ilimitadas
* Animações complexas
* Emotes persistentes
* Economia de itens

---

## Extensibilidade Controlada

Extensões futuras devem:

* Não afetar sincronização
* Não alterar regras de ZoneType
* Ser opcionais

---

## Nota Final

O Avatar System deve ser **simples o suficiente para desaparecer**.

Se o usuário passa mais tempo pensando no avatar do que no trabalho, o sistema falhou.
