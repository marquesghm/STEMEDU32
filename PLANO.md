# STEMEDU32 — Plano inicial

Programação tangível na web para crianças (~7 anos). Blocos horizontais
(estilo quebra-cabeça) → comandos Lua simples → microcontrolador (ESP32 / CH32V006).

## Foco: experiência do usuário (UX)
A UX é um dos focos principais do projeto. O público tem ~7 anos e usa principalmente
celular/tablet com o dedo, então cada interação deve ser óbvia, tolerante a erro e
gostosa de usar.

- Alvos de toque grandes; gestos naturais (arrastar, pinça) e sem precisar de precisão.
- Nada deve "brigar" com a criança: movimentos automáticos inesperados atrapalham.
- Feedback visual imediato (bloco ativo aceso, marcador de encaixe, avisos curtos).
- Pouco texto; ícones e cores dizem o que cada coisa faz.
- O modo simplificado é o padrão; o avançado fica no menu.

**Processo:** sugestões de UX são discutidas antes de implementar, e quando uma solução
pedida tiver um ponto fraco de UX, isso é apontado (com alternativa) antes de seguir.

## Arquivos (fase 1)

| Arquivo | Sistema | Responsabilidade |
|---|---|---|
| `index.html` | Editor + Conexão | Paleta, área de montagem, geração de Lua, envio via Web Serial |
| `simulador.html` | Simulador 3D | Three.js + robô + cenário; recebe comandos via `postMessage`; `?editar=1` pinta terrenos |
| `cenario.html` | Construtor de cenários | Junta o simulador em modo edição, a paleta de terrenos e o editor (`index.html?modo=cenario`) |
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
| *f* Função (início) | vermelho/azul/verde/roxo | `function f_<cor>() ... end` |
| *f*↷ Chamar função | mesma cor | `f_<cor>()` |
| 💡 Lâmpada | verde-água | `luz(ms)` — acende a bolinha da cabeça (simplificado: 500 ms; avançado: 0,5 a 5 s na bolinha) |

`n` = 0..9: tocar na bolinha do número abre um menu com 0 a 9.
Nos blocos de função, um toque troca a cor. O "chamar" pula para a sequência
que começa com a função da mesma cor e depois volta (máx. 8 níveis, para
proteger contra função que chama a si mesma).

## Workspace
Folha grande de tamanho fixo (4000×3000 px); a vista e os blocos não passam da borda.
Cada sequência (cadeia) tem posição própria.
- Arrastar o fundo move a vista; pinça com 2 dedos (ou Ctrl+roda) dá zoom 0,4x–2x.
- Arrastar do meio de uma sequência leva o bloco e os seguintes; perto de outra
  sequência ele encaixa (marcador azul); soltar na paleta apaga.
- Paleta: deslizar para os lados rola; puxar o bloco para cima pega.
- Minimapa no canto quando algum bloco fica fora da tela; tocar nele mostra tudo.
- Divisor entre simulador e workspace ajusta o tamanho dos dois.

## Cenários
Menu ☰ → **Construir cenário** abre `cenario.html`. Terrenos (uma casa cada, chão 11×11):

| Terreno | Efeito pronto | Reação programável |
|---|---|---|
| 🧱 Parede | robô bate e não entra | sim |
| ⭐ Estrela | é coletada; todas = desafio completo | sim |
| 💧 Água | robô afunda e o programa acaba | — |
| 🟥🟦🟩 Chão colorido | nenhum | sim |
| ⬛ Piso preto | fica amarelo quando a lâmpada acende na casa do robô ou nas 8 em volta (até reiniciar) | sim |

Bloco laranja **sensor**: chapéu cujos blocos rodam logo que o robô toca no terreno do bloco
(interrompe o resto do comando atual). Simplificado: um sensor por terreno na paleta.
Avançado: um sensor só, com o terreno na bolinha de baixo (toque abre menu).
Lua: `function sensor_<terreno>() ... end`.

O cenário (`localStorage.cenario`) é o mesmo nas duas páginas. Os workspaces são
independentes: `programa` (página de programar) e `programaCenario` (construtor).

Mensagens extras simulador ↔ editor: `tocou {terreno, indice}` → `reagir {comandos}`
(o simulador espera a resposta), `vitoria`, `caiu`, `cenarioMudou`.

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
