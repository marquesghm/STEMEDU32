# STEMEDU32 — instruções para o Claude

Programação tangível web para crianças (~7 anos): blocos horizontais → Lua → ESP32 / CH32V006.
Detalhes de design e protocolo em [PLANO.md](PLANO.md).

## Como trabalhar neste projeto

- **UX é foco principal.** Público de ~7 anos, celular/tablet, dedo.
  - Sugestões minhas de UX: **perguntar antes** de implementar.
  - Se uma solução pedida for fraca em UX: **avisar** (motivo + alternativa) antes de seguir.
  - Ao apontar falha ou sugestão de UX: **ser direto** — 1 ou 2 linhas, sem textão.
- **Modo simplificado é o padrão da conversa.** Pedidos sobre o editor sem modo explícito
  referem-se ao modo simplificado (blocos sem número, um único Play fixo fora da paleta,
  início de função criado automaticamente). Perguntar se deve valer também no avançado
  quando não for óbvio.
- **Commit e push a cada etapa** concluída e testada (branch `main`).
- Testar no navegador em tamanho de celular (375×812) antes de dar como pronto.

## Estrutura

Poucos `.html`, cada um com seu HTML+CSS+JS, só o mínimo para separar sistemas:

| Arquivo | Sistema |
|---|---|
| `index.html` | Editor de blocos (workspace, paleta, Lua, Web Serial). `?modo=cenario` = só workspace, dentro do construtor |
| `simulador.html` | Simulador 3D (Three.js via CDN). `?editar=1` = pinta terrenos |
| `cenario.html` | Construtor de cenários: junta simulador em edição + paleta de terrenos + editor |

As páginas conversam por `postMessage` (mensagens descritas no PLANO.md).
Programa e cenário ficam no `localStorage` (`programa`, `cenario`), compartilhados entre páginas.

## Convenções de código

- Nomes, comentários e textos em **português** (sem acentos em identificadores).
- Sem build, sem dependências locais; bibliotecas só por CDN.
- Rodar local: `python -m http.server 8032` → http://localhost:8032
  (`file://` quebra iframes/Web Serial).
