# STEMEDU32

Programação tangível na web para crianças de ~7 anos. A criança monta um programa
encaixando **blocos horizontais** (estilo quebra-cabeça) no celular ou tablet, vê o
resultado num **robô 3D** e, no futuro, envia o mesmo programa para um robô de verdade
(**ESP32** ou **CH32V006**) como comandos **Lua** simples.

> Foco principal do projeto: **experiência de uso (UX)** para criança, no celular, com o dedo.

---

## Sumário

1. [Como rodar](#como-rodar)
2. [Visão geral da tela](#visão-geral-da-tela)
3. [Blocos](#blocos)
4. [Workspace (área de montar)](#workspace-área-de-montar)
5. [Executar: play, passo a passo e parar](#executar-play-passo-a-passo-e-parar)
6. [Simulador 3D](#simulador-3d)
7. [Construtor de cenários](#construtor-de-cenários)
8. [Sensores e reações](#sensores-e-reações)
9. [Menu ☰](#menu-)
10. [Modo simplificado × avançado](#modo-simplificado--avançado)
11. [Código Lua e conexão com o robô](#código-lua-e-conexão-com-o-robô)
12. [Arquitetura](#arquitetura)
13. [O que fica salvo](#o-que-fica-salvo)
14. [Princípios de UX](#princípios-de-ux)
15. [Próximos passos](#próximos-passos)

---

## Como rodar

Não há build nem dependências: são arquivos `.html` servidos por qualquer servidor estático.

```bash
python -m http.server 8032
```

Abra **http://localhost:8032**.

- Abrir pelo arquivo (`file:///...`) não serve: o navegador isola cada arquivo (os iframes
  reclamam) e bloqueia o Web Serial.
- **No celular:** mesmo Wi-Fi do PC, abra `http://IP-DO-PC:8032` (descubra o IP com
  `ipconfig`). Se não abrir, libere o Python no Firewall do Windows (rede privada).
  Pelo IP da rede o Web Serial não funciona (só em `localhost` ou `https`).
- O Three.js vem de CDN, então é preciso internet.

---

## Visão geral da tela

De cima para baixo, na página principal (`index.html`):

| Área | O que é |
|---|---|
| **Header** | ☰ menu · título · **▶** (verde) · **⏭** passo (azul) · **⏹** (vermelho) |
| **Simulador** | robô 3D no chão quadriculado; cubo gizmo no canto inferior esquerdo; ↶/↷ no inferior direito |
| **Divisor** | linha fina arrastável que aumenta/diminui simulador × workspace |
| **Workspace** | folha onde se montam os programas |
| **Paleta** | faixa com os blocos disponíveis |

---

## Blocos

| Bloco | Cor | O que faz | Lua |
|---|---|---|---|
| ▶ **Play** | amarelo | começo do programa | — |
| ↑ **Frente** / ↓ **Trás** | rosa | anda *n* casas | `frente(n)` / `tras(n)` |
| ↺ **Esquerda** / ↻ **Direita** | rosa | gira *n* × 90° | `esquerda(n)` / `direita(n)` |
| 💡 **Lâmpada** | verde-água | acende a bolinha da cabeça do robô | `luz(ms)` |
| *f* **Início de função** | vermelho/azul/verde/roxo | começo de uma sequência reutilizável | `function f_<cor>() … end` |
| *f*↷ **Chamar função** | mesma cor | executa a função da mesma cor e volta | `f_<cor>()` |
| 🔶 **Sensor** | laranja | começo de uma reação a um terreno | `function sensor_<terreno>() … end` |

Interações com blocos (padrão: **tocar abre uma janelinha com opções**):

- **Bolinha embaixo do bloco** (modo avançado): número de vezes (0–9), tempo da lâmpada
  (0,5–5 s) ou terreno do sensor.
- **Chamar função**: tocar troca a cor (o início de função da mesma cor aparece sozinho
  no modo simplificado).

---

## Workspace (área de montar)

- **Folha grande** (4000 × 3000) com o programa no meio: dá para ir para qualquer lado.
- **Arrastar o fundo** move a vista; **pinça com 2 dedos** (ou Ctrl+roda) dá zoom.
- **Arrastar um bloco** do meio de uma sequência leva ele e todos os seguintes.
- Perto de outra sequência o bloco **encaixa** (marcador azul mostra onde).
- **Soltar na paleta apaga** (a paleta fica vermelha).
- **Paleta:** deslizar para os lados rola; puxar o bloco **para cima** (até bem na
  diagonal) pega o bloco. Tocar num bloco da paleta adiciona no fim da última sequência.
- **Minimapa** semitransparente aparece no canto quando algum bloco fica fora da tela;
  tocar nele afasta até mostrar tudo.
- Ao abrir a página, se algum bloco estiver escondido, a vista afasta (animado) até
  mostrar todos.
- **↶ / ↷** (canto inferior direito do simulador) desfazem/refazem mudanças nos blocos.
  O ↷ só aparece (acima do ↶) quando há algo para refazer.

---

## Executar: play, passo a passo e parar

- **▶** executa as sequências que começam com Play (de cima para baixo).
  Tocar no bloco Play do workspace executa só aquela sequência.
- **⏭ passo a passo** (botão azul, entre ▶ e ⏹):
  - com o programa parado → começa e executa só o **primeiro bloco**, depois pausa;
  - durante a execução → pausa no fim do bloco atual;
  - pausado → executa **um bloco** e pausa de novo, até o fim.
- **▶ com o programa pausado** continua rodando até o fim.
- **⏹** para tudo e coloca o robô de volta na saída.
- O bloco em execução **acende em azul**.

---

## Simulador 3D

- Chão de **11 × 11 casas**; o robô anda uma casa por vez e começa **de costas** para
  quem olha (frente = afastar-se da tela).
- **Câmera:** 1 dedo gira, pinça dá zoom/arrasta (mouse: esquerdo gira, roda zoom).
- **Cubo gizmo** (canto inferior esquerdo): gira junto com a câmera; tocar abre
  **Normal** / **De cima**.
- Rastro rosa mostra o caminho; a borda do chão funciona como parede (o robô bate e volta).

---

## Construtor de cenários

Menu ☰ → **Construir cenário** abre `cenario.html`:

- **Header:** ← volta a programar · ▶ testar · ⏭ passo a passo · ⏹ parar.
- **Cenário 3D** (abre visto de cima) com **↶/↷ próprios** (só dos terrenos).
- **Paleta de terrenos:** escolha a ferramenta e toque/arraste no chão para pintar.
  Com ✋ **Mover**, um dedo gira a câmera; com uma ferramenta, a câmera usa 2 dedos.
- **Workspace próprio** (independente do da página de programar) para montar as reações.

| Terreno | Efeito pronto |
|---|---|
| 🤖 **Robô** | tocar numa casa leva a saída do robô; tocar no robô abre a janelinha ↑→↓← de orientação |
| 🧱 **Parede** | o robô bate e não entra (não pode ficar na casa de saída) |
| ⭐ **Estrela** | é coletada e some; todas coletadas → “⭐ Desafio completo!” |
| 💧 **Água** | o robô afunda e o programa acaba |
| 🟥🟦🟩 **Chão colorido** | nenhum (serve para sensores) |
| ⬛ **Piso preto** | a lâmpada com o robô em cima troca **preto ↔ amarelo** |
| 🧽 **Apagar** / 🗑️ **Limpar** | remove um terreno / todo o cenário (e o robô volta ao centro) |

Pisos (menos parede) podem ficar na casa de saída; o terreno da saída **conta no início**.

---

## Sensores e reações

Um **sensor** é um bloco de início laranja: os blocos encaixados depois dele formam a
**reação** que roda quando o robô encontra aquele terreno.

| Sensor | Dispara quando |
|---|---|
| Parede | passa a existir uma parede **na casa da frente** (no início, ou depois de andar/girar) |
| Estrela, chão colorido, piso preto | o robô **entra** na casa (ou começa nela) |
| Piso amarelo | entra num piso preto aceso, ou a luz **troca** o piso embaixo para amarelo |
| Piso preto | entra num piso preto apagado, ou a luz troca o piso embaixo para preto |

Como a reação acontece:

1. **O sensor espera a vez:** o bloco atual termina inteiro; só então a reação roda.
   Vários toques num mesmo bloco geram reações na ordem em que aconteceram.
2. **O salto fica visível:** o bloco interrompido pisca, o sensor acende com brilho
   laranja e ⚡, e há uma pausa curta (0,7 s). Se o sensor estiver fora da tela, o
   minimapa pisca.
3. Ao fim da reação, o programa principal **continua de onde parou**.

Limite de 50 reações por execução (evita robô preso para sempre).
Os sensores aparecem na paleta no construtor, ou na página principal quando há cenário.

---

## Menu ☰

| Opção | O que faz |
|---|---|
| 🔌 Conectar robô | Web Serial (o ☰ fica verde quando conectado) |
| { } Mostrar código | janela com o Lua gerado |
| 🧸 Modo simplificado / avançado | alterna os modos |
| 🏗️ Construir cenário | abre o construtor |
| ⚙️ Configurações | tempo da lâmpada (simplificado): 0,25/0,5/1/2 s · tempo de cada passo do robô: 0,3/0,6/1/1,5 s |
| 🗑️ Limpar tudo | apaga os blocos (com confirmação) |

---

## Modo simplificado × avançado

O **simplificado é o padrão**.

| | Simplificado | Avançado |
|---|---|---|
| Números nos blocos | não (cada bloco = 1 ação) | sim, bolinha embaixo |
| Lâmpada | tempo das Configurações | tempo na bolinha |
| Play | um só, já posicionado, fora da paleta, não pode ser apagado | na paleta, quantos quiser |
| Início de função | criado sozinho abaixo do Play quando um “chamar” entra | na paleta |
| Sensor | um bloco por terreno na paleta | um bloco, terreno na bolinha |

---

## Código Lua e conexão com o robô

Exemplo do que o editor gera:

```lua
-- programa STEMEDU32

function f_azul()
  direita(1)
  frente(2)
end

function sensor_parede()
  direita(1)
end

-- play
frente(1)
luz(500)
f_azul()
```

**Web Serial** (Chrome/Edge no PC e Android), 115200 baud. Ao executar com o robô
conectado, a página envia:

```
inicio
<linhas do Lua, sem comentários>
fim
```

e `parar` no ⏹. O firmware acumula as linhas entre `inicio` e `fim` e executa.

> **CH32V006** tem 8 KB de RAM: Lua completo não cabe. Plano: no CH32 um interpretador
> mínimo (ou o site envia a lista já “desenrolada”); no ESP32, Lua de verdade.

---

## Arquitetura

Poucos arquivos `.html`, cada um com HTML + CSS + JS, só o mínimo para separar sistemas:

| Arquivo | Sistema |
|---|---|
| `index.html` | **Editor de blocos**: workspace, paleta, geração de Lua, Web Serial, menu. Com `?modo=cenario` vira só workspace + paleta (usado dentro do construtor). |
| `simulador.html` | **Simulador 3D** (Three.js via CDN): robô, cenário, execução, sensores. Com `?editar=1` pinta terrenos. |
| `cenario.html` | **Construtor de cenários**: junta o simulador em edição, a paleta de terrenos e o editor. |
| `PLANO.md` | plano e decisões de design |
| `CLAUDE.md` | regras de trabalho do projeto (UX, padrões, convenções) |

As páginas conversam por `postMessage`. No construtor, `cenario.html` repassa as
mensagens entre o simulador e o editor.

Principais mensagens:

| De → para | Mensagem | Uso |
|---|---|---|
| editor → simulador | `executar {comandos}` · `reiniciar` · `modoPasso` · `avancar` · `continuar` | rodar / parar / passo a passo |
| simulador → editor | `passo {indice}` · `fim` | destacar bloco em execução |
| simulador → editor | `tocou {terreno, indice}` | sensor disparou (o simulador espera a resposta) |
| editor → simulador | `reagir {comandos}` | comandos da reação (podem ser vazios) |
| simulador → editor | `vitoria` · `caiu` · `cenarioMudou` | avisos e atualizar a paleta |
| construtor ↔ simulador | `ferramenta` · `orientarRobo` · `desfazerCenario`/`refazerCenario` · `historicoCenario` · `menuRobo` | edição do cenário |

---

## O que fica salvo

Tudo no `localStorage` do navegador:

| Chave | Conteúdo |
|---|---|
| `programa` | blocos da página de programar |
| `programaCenario` | blocos do construtor (workspace independente) |
| `cenario` | terrenos (`"x,z": "parede"`, …) — compartilhado pelas duas páginas |
| `inicioRobo` | casa de saída e orientação do robô |
| `modo` | `simples` ou `avancado` |
| `config` | tempo da lâmpada e de cada passo |
| `alturaSim` | posição do divisor simulador × workspace |

---

## Princípios de UX

- Público de ~7 anos, celular/tablet, dedo: **alvos grandes**, gestos naturais.
- **Tocar na coisa abre uma janelinha com as opções**, junto dela.
- Nada de movimento automático que a criança não pediu.
- Feedback visual imediato (bloco aceso, marcador de encaixe, ⚡ do sensor, avisos curtos).
- Pouco texto; ícones e cores dizem o que cada coisa faz.

---

## Próximos passos

- Firmware ESP32 (Lua) e CH32V006 (interpretador mínimo) com motores e LED.
- Web Bluetooth (BLE) para o ESP32 funcionar no celular sem cabo.
- Blocos de espera, repetir e som.
- Janelinha de cores para os blocos de função (seguindo o padrão de UX).
- Publicar em `https` (ex.: GitHub Pages) para testar no celular com Web Serial/BLE.
