# Spatial Model

Este documento descreve o **modelo espacial** do sistema: como posições são representadas, como proximidade é calculada e como isso impacta presença, mídia e performance.

---

## Relações

Este documento se relaciona com:

* WORLD_SYSTEM.md
* ZONE_RULES.md
* ARCHITECTURE_GUIDE.md
* FAILURE_MODES.md

---

## Objetivos do Modelo Espacial

O Spatial Model existe para:

* Determinar **quem vê quem**
* Determinar **quem ouve quem**
* Servir de base para sharding e escalabilidade
* Ser simples de sincronizar e prever

Ele **não** existe para:

* Física realista
* Colisão complexa
* Gameplay ou puzzles

---

## Sistema de Coordenadas

### Representação

* Espaço 2D
* Coordenadas contínuas (float)
* Origem definida pelo mapa

Formato lógico:

```
(x: float, y: float)
```

Z não é utilizado.

---

## Atualização de Posição

### Frequência

* Atualizações enviadas em baixa frequência
* Throttling agressivo
* Interpolação no client

Objetivo: minimizar tráfego sem perder percepção de movimento.

---

## Proximidade Espacial

### Raio de Interação

Cada avatar possui:

* Raio de visibilidade
* Raio de áudio

Esses raios:

* São configuráveis
* Podem variar por ZoneType

---

## Áudio por Proximidade

* Distância afeta volume
* Fora do raio → áudio mutado
* ZoneType pode sobrescrever regras

O Spatial Model **não controla mídia**, apenas fornece dados.

---

## Visibilidade

### Quem é renderizado

O client deve:

* Renderizar apenas avatares dentro do raio de visibilidade
* Ignorar entidades distantes

Isso é essencial para performance.

---

## Sharding Espacial

### Motivação

Evitar que:

* Um client receba eventos de todo o mundo
* A mídia escale quadraticamente

---

### Estratégia

* Dividir o mundo em células espaciais
* Cada célula mantém seu próprio conjunto de usuários
* Células adjacentes podem ser parcialmente visíveis

---

## Transição entre Células

Quando um avatar cruza uma fronteira:

* Client notifica o Signaling Server
* Atualiza o conjunto de peers relevantes
* Reavalia política de mídia

A transição deve ser invisível ao usuário.

---

## Relação com ZoneType

ZoneType:

* Define regras
* Pode sobrescrever raios

Spatial Model:

* Aplica geometria
* Nunca lógica de permissão

---

## Limites Intencionais

* Máximo de avatares por célula
* Máximo de peers ativos por client

Ao atingir limites:

* Priorizar proximidade
* Desligar mídia distante

---

## Falhas e Degradação

Se Spatial Model falhar:

* Client entra em modo seguro
* Desliga mídia
* Mantém presença mínima

Referência: FAILURE_MODES.md

---

## Observabilidade

Eventos importantes:

* cell_enter
* cell_leave
* proximity_changed

---

## Nota Final

O Spatial Model é o **filtro fundamental** do sistema.

Se ele for simples, previsível e barato, todo o resto escala melhor.
