# Map Format

Este documento define o **formato de mapas** do sistema — o contrato de dados que descreve escritórios, salas e zonas. Ele existe para garantir **consistência, compatibilidade futura e facilidade de criação**, tanto para mapas prontos quanto customizados.

---

## Relações

Este documento se relaciona com:

* WORLD_SYSTEM.md
* SPATIAL_MODEL.md
* AVATAR_SYSTEM.md
* ZONE_RULES.md

---

## Objetivos do Formato de Mapa

O Map Format existe para:

* Descrever a estrutura espacial do mundo
* Associar áreas a ZoneTypes
* Permitir escritórios prontos e customizados
* Ser facilmente versionável e validável

Ele **não** existe para:

* Executar lógica
* Definir regras de mídia
* Permitir scripts arbitrários

---

## Princípios

* Mapas são **declarativos**
* Mapas são **imutáveis em runtime**
* Toda regra vem de ZoneType, não do mapa
* Mapas devem ser simples de validar

---

## Estrutura Geral

Formato sugerido: **JSON** (legível, versionável, fácil de validar)

Estrutura de alto nível:

```json
{
  "version": "1.0",
  "metadata": {},
  "dimensions": {},
  "zones": [],
  "staticElements": [],
  "interactiveElements": []
}
```

---

## Versionamento

* Campo obrigatório: `version`
* Compatibilidade retroativa sempre que possível
* Quebras exigem major version

Clients devem:

* Validar versão
* Rejeitar mapas incompatíveis

---

## Metadata

Usada apenas para:

* Nome do escritório
* Autor
* Thumbnail
* Tags

Metadata **não afeta comportamento**.

---

## Dimensões

Define:

* Largura
* Altura
* Unidade base (tiles ou unidades abstratas)

Sem impacto em regras.

---

## Zones

Zona é o elemento central do mapa.

```json
{
  "id": "zone-1",
  "zoneType": "OPEN",
  "shape": "rectangle",
  "bounds": {}
}
```

### Regras

* Toda área do mapa deve pertencer a exatamente um ZoneType
* Zonas não podem se sobrepor
* ZoneType deve existir no sistema

---

## Static Elements

Exemplos:

* Paredes
* Mesas
* Decoração

Características:

* Não possuem estado
* Não são sincronizados
* Apenas visuais

---

## Interactive Elements (Limitados)

Exemplos permitidos:

* Portas
* Áreas de transição

Características:

* Estados binários simples
* Sem scripts
* Sem lógica condicional

---

## Teleporte e Transição

Portas podem:

* Mover avatar para outra posição
* Trocar ZoneType implicitamente

Nunca executar regras.

---

## Validação de Mapa

Antes de aceitar um mapa:

* Schema validation
* Verificação de ZoneType
* Limites de tamanho
* Limites de elementos

Mapas inválidos são rejeitados.

---

## Segurança

Mapas:

* Não podem conter código
* Não podem conter URLs executáveis
* Não podem afetar networking

---

## Extensibilidade

Permitido no futuro:

* Novos tipos de shapes
* Novos elementos visuais

Sempre mantendo:

* Declaratividade
* Compatibilidade

---

## Anti-Objetivos

Explicitamente fora de escopo:

* Scripts customizados
* Mods client-side
* Lógica embutida
* Regras por mapa

---

## Nota Final

O Map Format é um **contrato**, não uma ferramenta criativa livre.

Limites claros hoje garantem liberdade sustentável amanhã.
