# Failure Modes

Este documento descreve os **modos de falha esperados** do sistema e como o client e a infraestrutura devem reagir a cada um deles. O objetivo é garantir **degradação graciosa**, previsibilidade e recuperação automática sempre que possível.

---

## Princípios

* Falha é um estado esperado
* Preferir degradação a interrupção total
* Recuperação automática sempre que possível
* Estados seguros acima de continuidade de mídia

---

## Categorias de Falha

1. Falhas de Conectividade
2. Falhas de Mídia (WebRTC)
3. Falhas de SFU
4. Falhas de TURN
5. Falhas de Signaling
6. Falhas de Client

---

## 1. Falhas de Conectividade

### Sintomas

* Perda temporária de rede
* Alta latência
* Packet loss elevado

### Comportamento Esperado

* Suspender envio de mídia
* Manter estado local
* Exibir status de reconexão

### Recuperação

* Tentativa automática de reconexão
* Reavaliação de política (P2P → TURN → SFU)

---

## 2. Falhas de Mídia (WebRTC)

### Sintomas

* ICE failed
* Track encerrada inesperadamente
* Áudio/vídeo congelado

### Comportamento Esperado

* Encerrar apenas a track afetada
* Manter signaling ativo
* Notificar usuário de forma não intrusiva

### Recuperação

* Renegociação automática
* Fallback de conectividade

---

## 3. Falhas de SFU

### Sintomas

* Desconexão do SFU
* Atraso excessivo
* Streams ausentes

### Comportamento Esperado

* Desligar SFU para a sala
* Manter presença

### Recuperação

* Fallback para P2P (quando permitido)
* Reentrada automática no SFU quando disponível

---

## 4. Falhas de TURN

### Sintomas

* TURN inacessível
* Custos elevados de relay

### Comportamento Esperado

* Evitar uso excessivo
* Notificar backend

### Recuperação

* Tentativa P2P direta
* Uso de SFU se disponível

---

## 5. Falhas de Signaling

### Sintomas

* WebSocket encerrado
* Timeout de mensagens

### Comportamento Esperado

* Entrar em estado Connecting
* Suspender todas as mídias

### Recuperação

* Reconexão exponencial
* Resync completo de estado

---

## 6. Falhas de Client

### Sintomas

* Uso excessivo de CPU
* Travamentos
* Crash

### Comportamento Esperado

* Encerrar mídias agressivamente
* Salvar estado mínimo

### Recuperação

* Reinício do client
* Reentrada segura em Idle

---

## Falhas por ZoneType

### Open Area

* Preferir desligar áudio

### Personal Room

* Manter controle local

### Team Area

* Priorizar SFU

### Meeting Room

* Notificar participantes
* Reentrada coordenada

### AFK

* Nenhuma mídia deve ser retomada automaticamente

---

## Observabilidade

Eventos obrigatórios:

* failure_detected
* recovery_attempted
* recovery_success
* recovery_failed

---

## Nota Final

Falhas são parte do fluxo normal do sistema.

A robustez do produto é medida não pela ausência de falhas, mas pela **qualidade da recuperação**.
