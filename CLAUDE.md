# STEMEDU32 — instruções para o Claude

Programação tangível web para crianças (~7 anos): blocos horizontais → Lua → ESP32 / CH32V006.
Documentação completa em [README.md](README.md); plano e decisões em [PLANO.md](PLANO.md).

## Como trabalhar neste projeto

- **UX é foco principal.** Público de ~7 anos, celular/tablet, dedo.
  - Sugestões minhas de UX: **perguntar antes** de implementar.
  - Se uma solução pedida for fraca em UX: **avisar** (motivo + alternativa) antes de seguir.
  - Ao apontar falha ou sugestão de UX: **ser direto** — 1 ou 2 linhas, sem textão.
  - **Padrão de interação:** tocar na coisa abre uma janelinha com as opções, junto dela
    (ex.: número do bloco, terreno do sensor, tempo da lâmpada, orientação do robô).
    Nada de "toque repetido troca para o próximo".
  - Nada de movimento automático que a criança não pediu (ex.: rolagem na borda, elástico).
- **Modo simplificado é o padrão da conversa.** Pedidos sobre o editor sem modo explícito
  referem-se ao modo simplificado (blocos sem número, um único Play fixo fora da paleta,
  início de função criado automaticamente, um sensor por terreno). Perguntar se deve
  valer também no avançado quando não for óbvio.
- **Commit e push a cada etapa** concluída e testada (branch `main`).
- Testar no navegador em tamanho de celular (375×812) antes de dar como pronto.
- Manter `README.md` (e `PLANO.md` quando mudar design/protocolo) atualizados junto com a mudança.

## Estrutura

Poucos `.html`, cada um com seu HTML+CSS+JS, só o mínimo para separar sistemas:

| Arquivo | Sistema |
|---|---|
| `index.html` | Editor de blocos (workspace, paleta, Lua, Web Serial, menu ☰, configurações, ▶/⏭/⏹). `?modo=cenario` = só workspace + paleta, dentro do construtor |
| `simulador.html` | Simulador 3D (Three.js via CDN): robô, cenário, execução, sensores, passo a passo. `?editar=1` = pinta terrenos, move a saída do robô |
| `cenario.html` | Construtor de cenários: repassa mensagens entre simulador (edição) e editor; paleta de terrenos; janelinha de orientação do robô |

As páginas conversam por `postMessage` (tabela de mensagens no README).

No `localStorage`:

| Chave | Conteúdo |
|---|---|
| `programa` / `programaCenario` | blocos da página de programar / do construtor (workspaces independentes) |
| `cenario` | terrenos `"x,z": tipo` (compartilhado) |
| `inicioRobo` | `{x, z, rot}` saída e orientação do robô |
| `modo` | `simples` (padrão) ou `avancado` |
| `config` | `{luz, passo}` tempo da lâmpada (simplificado) e de cada passo |
| `alturaSim` | posição do divisor |

## Regras do motor (para não quebrar comportamentos combinados)

- Robô anda **uma casa por vez**; começa de costas para a câmera (`ROT_INICIAL`).
- **Sensores esperam a vez:** terrenos tocados durante um bloco ficam em `pendentes` e as
  reações rodam quando o bloco termina (`ofsReacao` mantém a ordem). Editor e simulador
  inserem a reação na **mesma posição** (`indice` enviado em `tocou`).
- Sensor de **parede** = parede aparece na casa da frente (no início ou após andar/girar);
  bater não dispara.
- Terreno da casa de saída conta no início (sensor dispara, estrela coleta, água afunda).
- Luz sobre piso preto troca preto ↔ amarelo e dispara o sensor da cor nova ao apagar.
- Passo a passo: `modoPasso`/`avancar`/`continuar`; o marcador `pular` não conta como passo.

## Convenções de código

- Nomes, comentários e textos em **português** (sem acentos em identificadores).
- Sem build, sem dependências locais; bibliotecas só por CDN.
- Edições grandes com acentos: escrever o script num arquivo (Write) e rodar com Python
  lendo/gravando em UTF-8 — heredoc no bash do Windows estraga caracteres.
- Rodar local: `python -m http.server 8032` → http://localhost:8032
  (`file://` quebra iframes/Web Serial).
