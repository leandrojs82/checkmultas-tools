# Redesign da interface — CheckMultas Ferramentas

**Data:** 2026-08-21
**Arquivo alvo:** `index.html` (aplicação de arquivo único)

## Problema

As sete ferramentas foram portadas para o `index.html` uma a uma, cada qual trazendo o CSS
da sua origem. O resultado é um conjunto que parece colado em vez de um produto só:

- **Três widgets de upload** com desenhos diferentes: `.upload-zone` (Excel), `.frota-slot`
  (Frota), `.cons-drop` (Consolidador).
- **Cinco famílias de botão**: `.btn`, `.btn-wide`, `.play-btn`, `.btn-soft`, além dos avulsos
  `.cons-btn-remove`, `.dl-validas`, `.dl-invalidas`.
- **Cinco componentes de mensagem de estado**: `.status-msg`, `.exc-status`, `.cons-alert`,
  `.toast-error`, `#frota-alert`.
- Tamanhos de fonte sem escala: `12.5px`, `13.5px`, `.82rem`, `.95rem`, `.9rem` convivendo.
- Raios de canto de 4 a 20px sem critério.
- Estilo inline pesado nas telas mais recentes (Frota e Consolidador), porque não existem
  tokens de espaçamento.
- Nenhum estado de foco visível em toda a aplicação.

## Objetivo

Unificar o vocabulário visual e modernizar o acabamento, sem tocar em lógica.

## Direção estética

**Console de dados refinado.** Fundo escuro mais profundo, bordas finas no lugar de sombras
pesadas, tipografia apertada, números em mono tabular, laranja reservado para ação e foco.
Densidade média — legível para lote grande sem virar planilha.

## Escopo

### Dentro

Tokens de CSS, tipografia, o shell (sidebar, topbar, home), o vocabulário de componentes,
estados de foco/hover, e o markup das sete telas na medida necessária para adotar as novas
classes.

### Fora

Qualquer alteração de comportamento. Parsers, conversões, validações, geração de CSV/XLSX/SQL
e leitura de arquivos ficam byte a byte como estão.

## Decisões

### 1. Fundação de tokens

Acrescentar as escalas que faltam e recalibrar as existentes.

- **Espaçamento** — `--sp-1` a `--sp-8` = 4/8/12/16/20/24/32/40px. Todo padding, margin e gap
  passam a referenciar esses tokens.
- **Raios** — reduzir para três: `--r-sm: 6px`, `--r-md: 10px`, `--r-lg: 14px`. O token
  `--r-xl` (20px) é removido e seus usos remapeados para `--r-lg`. Pílula (`999px`) apenas em
  botão e tag.
- **Neutros** — dark ganha separação real entre planos: app `#0b0b0d`, surface `#121214`,
  elevated `#191a1d`, hover `#202126`; bordas `#232428` (sutil) e `#33353a` (média). O tema
  light recebe tratamento equivalente.
- **Laranja** — vira família em vez de valor único com opacidades hardcoded espalhadas:
  `--accent` (ação), `--accent-h` (hover), `--accent-soft` (tint 12%, para fundos),
  `--accent-ring` (anel de foco).
- **Elevação** — no dark, profundidade vem da borda; sombra só no light.

### 2. Tipografia

- **UI:** Inter via CDN, com `system-ui` de fallback. Se a CDN não responder, o layout não
  quebra — só troca a fonte.
- **Mono:** JetBrains Mono com fallback `ui-monospace`. Aplicada em placas, chassi, renavam,
  saída SQL e todos os números de tabela.
- **Escala fixa:** 11 / 12 / 13 / 15 / 18 / 24 / 30px. Nada fora disso.
- **Altura de linha:** 1.5 no corpo, 1.25 em títulos.
- `font-variant-numeric: tabular-nums` em tabelas e stats — é o que alinha as colunas de
  números verticalmente.

### 3. Shell

- **Sidebar** — largura 232px. Item ativo indicado por barra de 2px à esquerda mais fundo
  sutil, substituindo o ponto à direita (`.nav-dot`, que sai). Ícones SVG de traço 1.5px
  herdando `currentColor`.
- **Navegação agrupada** em dois rótulos:
  - *Placas* — Conversor de Placas, Validador de Placas
  - *Dados* — JSON → CSV, Processador Excel, Conversor SQL IN, Unificador de Frota,
    Consolidador de Arquivos

  "Início" fica acima dos grupos, sem rótulo.
- **Topbar** — 56px, `backdrop-filter: blur(8px)`, título e subtítulo separados por divisor
  fino.
- **Home** — hero mais contido: título em 30px, subtítulo em 15px, margem inferior de 32px em
  vez de 40px. Cards com ícone monocromático em caixa neutra e borda que acende no hover. O
  `transform: translateY(-3px)` sai, substituído por transição de borda e fundo.
- **Ícones** — os emojis atuais (🏠 🔄 ✅ 📋 📊 🔎 🚗 🧩) dão lugar a SVG inline de traço
  1.5px, um por ferramenta, usados tanto na sidebar quanto nos cards da home. Também saem os
  emojis embutidos em rótulos de botão (`⬇ Baixar`, `🗑 Limpar`, `📋 Copiar`, `🔄 Processar
  outro arquivo`), substituídos pelo mesmo conjunto de SVG.

### 4. Vocabulário unificado de componentes

Um componente por função, aplicado nas sete ferramentas.

| Componente novo | Substitui |
| --- | --- |
| `.drop` (`--multi`, `--slot`) | `.upload-zone`, `.frota-slot`, `.cons-drop` |
| `.btn` (`--primary`, `--ghost`, `--danger`, `--sm`, `--block`) | `.btn`, `.btn-wide`, `.play-btn`, `.btn-soft`, `.cons-btn-remove`, `.dl-validas`, `.dl-invalidas` |
| `.panel` | `.input-card`, `.copy-panel`, `.json-stat`, `.exc-detail` |
| `.table` | tabelas de `.exc-preview-wrap` e de `#page-consolidador` |
| `.status` (`info`, `ok`, `warn`, `err`) | `.status-msg`, `.exc-status`, `.cons-alert`, `.toast-error`, `#frota-alert` |
| `.tag` | `.cons-badge`, `.pill`, `#sqlin-badge` |
| `.field` | inputs, textareas e selects hoje estilizados avulso |

Consequência: a maior parte do `style="..."` inline das telas de Frota e Consolidador
desaparece, porque passa a existir token de espaçamento para expressar o mesmo.

Os cartões de placa do Conversor (`.plate-box` e as variantes `is-trad` / `is-merc` /
`is-conv`) permanecem como componente próprio — são identidade da ferramenta, não um card
genérico.

### 5. Estados e movimento

- `:focus-visible` com anel de 2px em `--accent-ring` e offset, em todo elemento interativo.
  Hoje não existe estado de foco em lugar nenhum.
- Transições de 120–180ms com `ease-out`.
- Bloco `@media (prefers-reduced-motion: reduce)` zerando transições e animações.
- Hover contido: mudança de borda e fundo, sem deslocamento.
- Empty state com o mesmo desenho nas sete telas.

## Restrições

- **Arquivo único, sem build.** Todo CSS e JS continuam inline no `index.html`.
- **IDs preservados.** Todo `id` consultado pelo JavaScript permanece com o mesmo nome. As
  classes podem mudar; os ganchos do script, não.
- **Compatibilidade de tema.** Tudo funciona em dark e light, com a alternância e a
  persistência em `localStorage` intactas.
- **Laranja `#ff5b1f` segue sendo a cor da marca.**

## Riscos

O risco principal é quebrar um gancho do JS ao reescrever markup. Duas telas concentram esse
risco, por manipularem HTML por string:

- **Frota** — `frota_renderSlot()` reescreve o `innerHTML` do slot, incluindo um `onclick`
  inline. Ao trocar `.frota-slot` por `.drop--slot`, essa função precisa ser atualizada junto.
- **Consolidador** — `render()` e `renderWidths()` montam linhas de tabela por template
  string, com `.cons-badge` e `.cons-btn-remove` embutidos.

Também há classes referenciadas de dentro do JS em outras telas (`.dragover` no Excel,
`.show` no toast do Conversor). Cada uma precisa de verificação após a troca.

## Verificação

Servir o projeto localmente e exercitar as sete ferramentas no navegador, em dark e em light:

1. **Conversor** — converter um lote misto e copiar SQL.
2. **Validador** — validar lote com placas boas e ruins.
3. **JSON → CSV** — carregar o exemplo e processar.
4. **Excel** — subir um `.xlsx` e conferir o preview.
5. **SQL IN** — converter e copiar.
6. **Frota** — preencher os três slots, processar, conferir a barra de progresso.
7. **Consolidador** — subir arquivos compatíveis e divergentes.

Critério de aceite: nenhum erro de console e nenhuma regressão funcional nas sete telas.
