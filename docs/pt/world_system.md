# World System

Este documento define o **sistema de mundo** da aplicação — como o espaço virtual é estruturado, carregado e sincronizado entre clients — inspirado em produtos como o **Gather**, porém com foco em **simplicidade técnica, performance e escalabilidade**.

---

## Relações

Este documento se relaciona com:

* SPATIAL_MODEL.md
* AVATAR_SYSTEM.md
* FAILURE_MODES.md

---

## Objetivos do World System

O mundo existe para:

* Fornecer **presença espacial** aos usuários
* Servir como base para **áudio por proximidade** e interações
* Organizar ZoneTypes de forma visual
* Manter baixo custo cognitivo e técnico

O mundo **não** existe para:

* Simular física realista
* Criar gameplay complexo
* Servir como sandbox artístico irrestrito

---

## Filosofia (Gather-like, não Game-like)

O mundo segue estes princípios:

* 2.5D ou top-down simplificado
* Navegação intuitiva (WASD / setas / clique)
* Colisão simples ou inexistente
* Leitura imediata de quem está próximo

O mundo é um **meio de comunicação**, não um jogo competitivo.

---

## Estrutura do Mundo

### Mundo como Container

* Um mundo é composto por:

  * Layout (mapa)
  * Zonas (ZoneType)
  * Elementos estáticos

* O mundo pode ser:

  * Global
  * Por organização
  * Por escritório

---

## ZoneTypes no Mundo

Cada área do mapa está associada a um **ZoneType**, que define regras de mídia e interação.

Exemplos:

* Open Area
* Personal Room
* Team Area
* Meeting Room
* AFK

O mundo **não contém regras de mídia**, apenas **referências ao ZoneType**.

---

## Elementos do Mapa

### Estáticos

* Paredes
* Portas
* Mesas
* Decoração

Características:

* Não sincronizados em tempo real
* Definidos no carregamento do mundo

---

### Interativos (limitados)

* Portas (teleporte / troca de zona)
* Áreas de trigger

Características:

* Estados simples (aberto/fechado)
* Baixa frequência de atualização

---

## Escala e Limites

### Limites Intencionais

* Número máximo de usuários visíveis por client
* Área máxima renderizada simultaneamente
* Número limitado de elementos interativos

Esses limites são **feature, não bug**.

---

## Carregamento do Mundo

### Estratégia

* Carregamento inicial leve
* Assets estáticos
* Lazy load para áreas adjacentes

O mundo deve estar visível **antes** da mídia iniciar.

---

## Sincronização

### O que é sincronizado

* Posição do avatar
* ZoneType atual
* Estado básico (AFK, speaking)

### O que NÃO é sincronizado

* Decoração dinâmica livre
* Física
* Animações complexas

---

## Relação com Áudio e Vídeo

* Proximidade espacial influencia áudio
* O mundo **não** controla mídia diretamente
* O mundo apenas fornece contexto

---

## Performance

Diretrizes:

* Mundo deve rodar a 60fps em hardware médio
* Renderização simples
* Baixo consumo de memória

---

## Extensibilidade

O sistema deve permitir:

* Criação de mapas customizados
* Templates prontos de escritório
* Evolução visual sem quebrar contratos

---

## Anti-Objetivos

Explicitamente fora de escopo:

* Mundo totalmente 3D
* Customização irrestrita em tempo real
* Mods client-side arbitrários

---

## Nota Final

O World System é um **facilitador social**.

Ele deve ser previsível, leve e estável — mesmo que isso signifique abrir mão de liberdade artística.
