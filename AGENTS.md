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

Este repositório contém **dois projetos independentes** — sem código, estilo ou assets compartilhados:

```
index.html    → LinearLand: jogo educativo com boneco de palito + 4 níveis
demo.html     → Matrix Lab: demo visual independente (formas + sliders T/R/S, sem personagem)
README.md     → Descrição do repositório (quase vazio — pendente)
```

- **`index.html` (LinearLand)** é o projeto principal deste guia; sua evolução acontece nele ou em versões incrementais (`v2.html`, `v3.html`...).
- **`demo.html` (Matrix Lab)** é um projeto separado, com identidade visual própria (paleta neon, fontes Space Mono/Syne, canvas quadrado com grid cartesiano). Mudanças em um arquivo **não** devem se propagar para o outro, e ele não é referência de estilo nem de código para o jogo.

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
| Game state | `levelDone`, `player` (com `scale`), `keys` |
| World definitions | Array `worlds[]`: cada mundo tem `{ name, concept, phases[] }`; cada fase `{ name, objective, extraHint, init, update, draw, getObjMatrix?, lcText }` |
| Physics | `updatePlayer(dt, state)`: gravidade (900), pulo (-480), velocidade (220), colisão AABB com chão/plataformas/caixas/paredes; largura e altura do jogador escalam com `player.scale` |
| Drawing | `drawBackground()`, `drawPlatform()`, `drawStickFigure()` (escala), `drawStar()`, `drawStars()`, `drawBoxes()`, `drawPlats()`, `drawCombos()` |
| Matrix panel | `updateMatrix(M)`, `updateVectors()` — flash nas células que mudam |
| Audio | `ac()`, `beep()`, `sfxStar()`, `sfxWin()`, `sfxReset()`, `tfx()` — sons sintetizados (Web Audio, sem arquivos) |
| Particles | `burst()`, `updateParticles()`, `drawParticles()` — partículas de coleta |
| Progression | `progress[]` por mundo, persistido em localStorage (`linearland-progress`); estrelas coletadas por fase |
| Phase management | `initPhase(w,p)`, `goWorld()`, `goPhase()`, `nextPhase()`, `completeLevel()`, `refreshTopbar()` |
| Targets/selection | `state.targets[]` (objetos + `player`), `selTarget()`, `selObj()`, `Tab` alterna; `scaleTargetBy()`, `rotateTargetBy()` |
| UI helpers | `buildControls()`, `syncSliders()`, `sliderRot()`, `sliderScale()` |
| Main loop | `requestAnimationFrame` com `dt` em segundos (cap de 0.05s) |

### Sistema de coordenadas

- Canvas: y cresce para **baixo** (padrão do canvas HTML)
- Exibição para o usuário: y é invertido (`H - player.y`) para parecer coordenadas matemáticas normais
- `groundY()` retorna `H - 80`
- Matriz exibida no painel: se o alvo selecionado é o **player**, mostra a escala dele (`[[s,0,0],[0,s,0],[0,0,1]]`); se é um **objeto**, mostra a matriz do objeto; sem alvo, mostra a matriz de translação do jogador. Sempre 3×3 homogênea.
- Estilos de objeto: `plat` (gira), `box` (escala; sólida), `wall` (fixa, sólida; `anchor:'center'` = centrada em y), `combo` (gira + escala)

---

## Estrutura de gameplay: 4 mundos × fases

Cada **mundo** ensina um conceito. Cada mundo tem **3-4 fases** com arco didático:
**Descobrir** (mecânica nova sem falha possível) → **Praticar** (variações) → **Virar** (uso não óbvio) → **Testar** (desafio combinado).

| Mundo | Fases | Conceito | Mecânicas nas fases |
|---|---|---|---|
| 1 — Translação | 4 | Somar vetores move objetos | Andar/pular; plataformas em escada; vãos |
| 2 — Escala | 4 | Multiplicar diagonal redimensiona | Encolher caixas bloqueadoras; **encolher o próprio personagem** (Fenda); combinar |
| 3 — Rotação | 3 | Seno e cosseno giram pontos | Girar plataformas até deitar; subir escada giratória |
| 4 — Composição | 3 | Produto de matrizes combina tudo | Girar + esticar plataformas para atravessar pontes |

**Regra de ouro:** cada fase ensina **um** conceito novo; o objetivo típico é coletar **todas as estrelas** da fase. Fases do mesmo mundo desbloqueiam em sequência; mundos desbloqueiam ao completar o anterior. Progresso salvo em localStorage.

**Controles:** ← → mover · ↑ pular · Q/E transformar alvo selecionado (escala no M2, rotação no M3, rotação no M4) · Z/X escalar no M4 · Tab troca alvo (objetos + "você"). Sliders no painel direito refletem o alvo selecionado.

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
- Exibir sempre a **matriz 3×3 homogênea** no painel direito, mesmo que a fase use só 2×2
- Fazer o painel direito atualizar em **tempo real** a cada frame
- Adicionar **feedback visual** claro quando o usuário interage (flash, glow, animação)
- Comentar seções novas com `// ── NOME DA SEÇÃO ──` e blocos com `// ═══ ... ═══`
- Preservar o **boneco de palito** como personagem principal — ele é central para a identidade do projeto
- Ao adicionar novos mundos/fases, seguir a estrutura `worlds[]` e o arco Descobrir→Praticar→Virar→Testar

### O que NÃO FAZER

- Não usar frameworks (React, Vue, Three.js, etc.)
- Não separar em múltiplos arquivos sem necessidade
- Não remover a exibição da matriz — é o elemento didático central
- Não tornar o jogo difícil demais: o objetivo é **aprender**, não frustrar
- Não usar física complexa: o modelo atual (gravidade simples + AABB) é proposital
- Não quebrar o sistema de progressão — fases/mundos desbloqueiam em sequência
- Não alterar as variáveis CSS de cor sem manter coerência com os conceitos matemáticos (verde=translação, etc.)

### Gotchas conhecidos

- `Tab` é interceptado como tecla de jogo (troca alvo selecionado); o `keydown` faz `preventDefault` em todas as teclas
- `Esc` alterna pause; `R` reinicia a fase — ambos ignorados quando a fase já foi completada
- Física congela quando `levelDone || paused` (o overlay/prévia não movimenta mais o boneco)
- Teclas são limpas no `blur` da janela (evita boneco "andando sozinho" após Alt+Tab)
- Áudio: Web Audio API gera os sons sintetizados (sem arquivos); contexto criado sob demanda e retomado se suspenso; `tfx()` faz throttle do som de transformação
- Partículas vivem no array `particles`, atualizadas a cada frame e desenhadas por cima da cena
- Colisão com plataformas rotacionadas usa o **AABB aproximado** do retângulo girado (`getPlatforms`) — simplificação intencional
- `getBoxes`: caixas/paredes bloqueiam lateralmente; caixas com topo ≤ `STEP_OVER` (36px) podem ser pisadas
- Fases de rotação/composição: plataformas começam verticais — girar muda a altura do topo do AABB; escalar estica o alcance (mínimo ~1.6× para o salto da “Ponte dupla”)
- `player.scale` (0.5–1.5) altera a largura/altura de colisão do boneco e o desenho; usado nas fases "Fenda" e "Caminho apertado"
- Fricção horizontal (`player.vx *= 0.7` por frame) não usa `dt` — pequena dependência de framerate
- Resize da janela **reinicia a fase atual** preservando as estrelas já coletadas
- Fases/chips travados ignoram cliques: `goPhase` retorna cedo se `i > progress[w]`

---

## Melhorias planejadas (backlog)

Prioridade sugerida:

- [ ] **Modo Quiz:** o aluno digita os valores da matriz e vê o resultado aplicado
- [ ] **Tooltips educativos:** hover em cada célula da matriz explica o que aquele valor representa
- [ ] **Destaque vivo:** ressaltar no painel quais células a tecla atual está alterando
- [ ] **Tela de conceito do mundo** ao desbloquear um mundo novo
- [ ] **Menu inicial / mapa de mundos** com estrelas por fase
- [ ] **Mundo 5 — Reflexão:** espelhar objetos com matriz `[[-1,0],[0,1]]`, personagem atravessa espelho
- [ ] **Histórico de transformações:** mostrar as últimas N matrizes aplicadas em cadeia
- [ ] **Modo professor:** pause o jogo e mostre derivação da fórmula passo a passo
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
