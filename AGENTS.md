# LinearLand — AGENTS.md

Guia de contexto para agentes de código (Claude Code, OpenCode, Codex, etc.) trabalhando no projeto LinearLand.
Leia este arquivo inteiro antes de qualquer mudança no código.

---

## O que é este projeto

**LinearLand** é um jogo educativo interativo em HTML/CSS/JS puro (single-file, sem frameworks, sem build step) que ensina álgebra linear — especificamente transformações de matrizes — de forma visual e progressiva.

O público-alvo é **qualquer pessoa**, do ensino médio à faculdade. O jogo precisa ser intuitivo o suficiente para funcionar sem instruções longas.

O projeto nasceu de uma conversa sobre como jogos de computador são essencialmente álgebra linear aplicada em tempo real.

---

## Arquivos do projeto

```
index.html    → Jogo principal: boneco de palito + 4 níveis (arquivo principal)
demo.html     → Demo v1: canvas com formas + sliders de T/R/S (sem personagem)
README.md     → Descrição do repositório (quase vazio — pendente)
```

Toda evolução do jogo deve acontecer em `index.html` ou em versões incrementais (`v2.html`, `v3.html`...).

`demo.html` é independente: não compartilha código com o jogo principal e **não** deve ser usado como referência de estilo (usa paleta neon própria, fontes Space Mono/Syne e canvas quadrado com grid cartesiano).

---

## Arquitetura do código (index.html)

O jogo é **um único arquivo HTML** com três seções:

1. `<style>` — CSS com variáveis de cor, layout em grid (game canvas + painel direito de 280px)
2. `<body>` — estrutura HTML: topbar com abas de nível, canvas do jogo, painel direito com matriz + vetores + controles + conceito + progresso
3. `<script>` — toda a lógica em JS vanilla

### Estrutura do JS

| Seção | Responsabilidade |
|---|---|
| Canvas setup | `resize()`, devicePixelRatio |
| Game state | `currentLevel`, `unlockedLevels`, `levelDone`, `player`, `keys` |
| Level definitions | Array `levels[]`: `{ name, objective, concept, init, update, draw, getObjMatrix, lcText, extraHint }` |
| Physics | `updatePlayer(dt, state)`: gravidade (900), pulo (-480), velocidade (220), colisão AABB com chão/plataformas/caixas |
| Drawing | `drawBackground()`, `drawPlatform()`, `drawStickFigure()`, `drawStar()` |
| Matrix panel | `updateMatrix(M)`, `updateVectors()` — flash nas células que mudam |
| Level management | `initLevel()`, `goLevel()`, `nextLevel()`, `completeLevel()` |
| Object sliders | `applyScaleSlider()`, `applyRotSlider()`, `applyRotSlider4()`, `applyScaleSlider4()` |
| Main loop | `requestAnimationFrame` com `dt` em segundos (cap de 0.05s) |

### Sistema de coordenadas

- Canvas: y cresce para **baixo** (padrão do canvas HTML)
- Exibição para o usuário: y é invertido (`H - player.y`) para parecer coordenadas matemáticas normais
- `groundY()` retorna `H - 80`
- Matriz exibida no painel: níveis com objetos mostram a matriz do **objeto selecionado** (`getObjMatrix`); caso contrário, mostra a matriz do **jogador** (`getPlayerMatrix`, translação pura). Sempre 3×3 homogênea.

---

## Os 4 níveis e seus objetivos didáticos

| Nível | Conceito | Mecânica principal | Matriz exibida |
|---|---|---|---|
| 1 — Translação | Somar vetores move objetos | WASD/setas para mover e pular | Matriz do personagem com `tx`, `ty` |
| 2 — Escala | Multiplicar diagonal redimensiona | Q/E para encolher/crescer caixas, Tab troca objeto | Matriz do objeto: `[[s,0,0],[0,s,0],[0,0,1]]` |
| 3 — Rotação | Seno e cosseno giram pontos | Q/E para girar plataformas, Tab troca objeto | Matriz de rotação com cos/sen |
| 4 — Composição | Produto de matrizes combina tudo | Q/E girar + Z/X escalar, Tab troca objeto | Matriz composta R·S do objeto |

**Regra de ouro:** cada nível deve ensinar **um conceito novo** por vez, construindo sobre o anterior. O objetivo é sempre alcançar a estrela; completar o nível desbloqueia o próximo (abas no topo).

---

## Design visual (não altere sem motivo)

- **Tema:** dark, estilo "laboratório de código" / retro-futurista
- **Paleta de cores (CSS vars):**
  - `--accent: #58e06a` — verde (translação, elementos primários)
  - `--accent2: #f78166` — laranja/vermelho (escala, flash de mudança)
  - `--accent3: #79c0ff` — azul claro (rotação, vetores)
  - `--accent4: #ffa657` — amarelo/laranja (composição)
  - `--bg: #0d1117`, `--panel-bg: #161b22` — fundo escuro estilo GitHub dark
- **Fontes:** `Pixelify Sans` — títulos, estilo pixel/game; `DM Mono` — código, valores numéricos, UI
- **Célula da matriz:** pisca em `--accent2` (laranja) quando o valor muda — efeito `.flash` em `updateMatrix`

---

## Regras de desenvolvimento

### O que FAZER

- Manter tudo em **arquivo único** (HTML/CSS/JS inline) — sem dependências externas além das fontes Google Fonts
- Usar `requestAnimationFrame` com `dt` para física independente de framerate
- Exibir sempre a **matriz 3×3 homogênea** no painel direito, mesmo que o nível use só 2×2
- Fazer o painel direito atualizar em **tempo real** a cada frame
- Adicionar **feedback visual** claro quando o usuário interage (flash, glow, animação)
- Comentar seções novas com `// ── NOME DA SEÇÃO ──` e blocos com `// ═══ ... ═══`
- Preservar o **boneco de palito** como personagem principal — ele é central para a identidade do projeto
- Ao adicionar novos níveis, seguir a estrutura do array `levels[]`

### O que NÃO FAZER

- Não usar frameworks (React, Vue, Three.js, etc.)
- Não separar em múltiplos arquivos sem necessidade
- Não remover a exibição da matriz — é o elemento didático central
- Não tornar o jogo difícil demais: o objetivo é **aprender**, não frustrar
- Não usar física complexa: o modelo atual (gravidade simples + AABB) é proposital
- Não quebrar o sistema de níveis progressivos — cada nível deve ser desbloqueado em sequência
- Não alterar as variáveis CSS de cor sem manter coerência com os conceitos matemáticos (verde=translação, etc.)

### Gotchas conhecidos

- `Tab` é interceptado como tecla de jogo (troca objeto selecionado); o `keydown` faz `preventDefault` em todas as teclas
- Colisão com plataformas rotacionadas usa o **AABB aproximado** do retângulo girado (`getPlatforms`) — simplificação intencional
- `getBoxes` (nível 2) só resolve colisão **por cima** das caixas — não há colisão lateral
- Fricção horizontal (`player.vx *= 0.7` por frame) não usa `dt` — pequena dependência de framerate
- Abas de nível travadas (`locked`) ignoram cliques: `goLevel` retorna cedo se `n > unlockedLevels`

---

## Melhorias planejadas (backlog)

Prioridade sugerida:

- [ ] **Nível 5 — Reflexão:** espelhar objetos com matriz `[[-1,0],[0,1]]`, personagem atravessa espelho
- [ ] **Tooltips educativos:** hover em cada célula da matriz explica o que aquele valor representa
- [ ] **Modo Quiz:** o aluno digita os valores da matriz e vê o resultado aplicado
- [ ] **Histórico de transformações:** mostrar as últimas N matrizes aplicadas em cadeia
- [ ] **Modo professor:** pause o jogo e mostre derivação da fórmula passo a passo
- [ ] **Efeitos de partícula** quando o personagem coleta a estrela
- [ ] **Música/sons** sutis (Web Audio API) sincronizados com transformações
- [ ] **Salvamento de progresso** via localStorage
- [ ] **Mobile touch controls** para funcionar em tablets em sala de aula
- [ ] **Preencher README.md** com descrição real do projeto

---

## Contexto pedagógico

Este projeto é para uma **aula de revisão de matrizes e introdução à álgebra linear**. A ideia central é que alunos *sintam* as transformações antes de ver as fórmulas — inversão da ordem tradicional.

Estratégia de uso em aula:
1. Antes do jogo: perguntar "o que acontece com a matriz quando giro 90°?"
2. Durante: pausar e discutir os números na tela
3. Depois: mostrar o código-fonte e conectar com a matemática formal

---

## Comandos úteis

```bash
# Abrir o jogo no navegador (macOS)
open index.html

# Abrir no navegador (Linux)
xdg-open index.html

# Validar HTML
npx html-validate index.html
```

---

## Como colaborar neste arquivo

Se durante o desenvolvimento você descobrir padrões importantes, decisões de arquitetura ou gotchas — atualize este AGENTS.md. Mantenha-o abaixo de 200 linhas. Se crescer demais, extraia seções para `docs/`.

Nota: `CLAUDE.md` importa este arquivo — mantenha **AGENTS.md** como fonte única de verdade para evitar duplicação.
