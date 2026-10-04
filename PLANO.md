# STEMEDU32 — Plano inicial

Programação tangível na web para crianças (~7 anos). Blocos horizontais
(estilo quebra-cabeça) → comandos Lua simples → microcontrolador (ESP32 / CH32V006).

## Arquivos (fase 1)

| Arquivo | Sistema | Responsabilidade |
|---|---|---|
| `index.html` | Editor + Conexão | Paleta, área de montagem, geração de Lua, envio via Web Serial |
| `simulador.html` | Simulador 3D | Three.js + robô; recebe comandos via `postMessage` |
| `assets/` | Referência | Esboço visual dos blocos |

`index.html` embute `simulador.html` num `<iframe>`. A única ligação entre os dois
é uma lista de comandos, então o simulador pode ser trocado/reusado sem mexer no editor.

### Mensagens editor ↔ simulador
```js
// editor → simulador
{ tipo: 'executar', comandos: [{ cmd: 'frente', n: 2 }, ...] }
{ tipo: 'parar' } | { tipo: 'reiniciar' }
// simulador → editor
{ tipo: 'passo', indice: 0 }   // destaca o bloco em execução
{ tipo: 'fim' }
```

## Blocos (fase 1)

| Bloco | Cor | Lua gerado |
|---|---|---|
| ▶ Play (início) | amarelo | `-- inicio` (cabeçalho do programa) |
| ↑ Frente | rosa | `frente(n)` |
| ↓ Trás | rosa | `tras(n)` |
| ↺ Esquerda | rosa | `esquerda(n)` (n × 90°) |
| ↻ Direita | rosa | `direita(n)` |

`n` = 1..9, a criança toca na bolinha do número para mudar.

## Protocolo com o microcontrolador (Web Serial, 115200)
```
inicio\n
frente(2)\n
direita(1)\n
fim\n
```
O firmware acumula as linhas entre `inicio` e `fim` e executa.

## Ponto de atenção: CH32V006
O CH32V006 tem ~62 KB de flash e 8 KB de RAM — **Lua completo não cabe**.
Proposta: no CH32 um *parser mínimo* que só entende `nome(numero)` (o subconjunto
que os blocos geram). No ESP32 dá para rodar Lua de verdade (ex.: NodeMCU/eLua
ou Lua 5.4 embutido no Arduino/ESP-IDF). Como os blocos geram só esse subconjunto,
o mesmo texto serve aos dois.

## Próximas fases
1. Firmware ESP32 (Lua real) e CH32V006 (parser mínimo) + motores.
2. Web Bluetooth (BLE) para ESP32 — celular sem cabo.
3. Blocos: espera, repetir, som, LED, sensores.
4. Salvar/abrir programas (localStorage), PWA offline.
