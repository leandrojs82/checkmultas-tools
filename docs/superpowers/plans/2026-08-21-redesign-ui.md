# Redesign da Interface — Plano de Implementação

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [x]`) syntax for tracking.

**Goal:** Unificar o vocabulário visual das 7 ferramentas do `index.html` e modernizar o acabamento, sem alterar nenhum comportamento.

**Architecture:** O `index.html` é uma aplicação de arquivo único com CSS e JS inline. O trabalho acontece em três camadas, nessa ordem: primeiro a fundação (tokens de cor, espaçamento, raio e tipografia), depois o vocabulário de componentes adicionado de forma aditiva, e só então a migração tela a tela para as novas classes. O CSS antigo só é removido na última task, quando nada mais o referencia. Essa ordem garante que a aplicação continua funcional ao fim de cada task.

**Tech Stack:** HTML, CSS e JavaScript puros. SheetJS 0.18.5 via CDN. Sem build, sem npm.

## Global Constraints

- **Arquivo único.** Todo CSS e JS permanecem inline em `index.html`. Nenhum arquivo novo de asset.
- **Zero mudança de comportamento.** Parsers, conversões, validações e exportações não são tocados.
- **IDs preservados.** Todo `id` consultado pelo JavaScript mantém o nome exato. Classes podem mudar; ganchos de script, não.
- **Dark e light.** Toda mudança vale nos dois temas. A alternância e a persistência em `localStorage` (chave `cm-theme`) seguem intactas.
- **Cor da marca:** `#ff5b1f` no dark, `#ea580c` no light.
- **Escala tipográfica fechada:** 11 / 12 / 13 / 15 / 18 / 24 / 30px. Nenhum tamanho fora dela.
- **Raios:** `--r-sm: 6px`, `--r-md: 10px`, `--r-lg: 14px`, e `999px` só em botão e tag.
- **Sem commits.** O usuário versiona por conta própria. Nenhuma task executa `git add` ou `git commit`.

## Harness de verificação

Toda task termina verificando no navegador. Suba o servidor uma vez:

```bash
cd "C:/Users/Dev/Documents/CHECKMULTAS/checkmultas-tools" && (python -m http.server 8777 >/dev/null 2>&1 &) ; sleep 2; curl -s -o /dev/null -w "%{http_code}" http://localhost:8777/index.html
```

Esperado: `200`.

Navegue para `http://localhost:8777/index.html` e rode o **smoke check padrão** no console da página:

```js
const pages=['home','conversor','validador','jsoncsv','excel','sqlin','frota','consolidador'];
const missing=pages.filter(p=>!document.getElementById('page-'+p));
const broken=[];
pages.forEach(p=>{ try{ showPageByName(p); }catch(e){ broken.push(p+': '+e.message); } });
showPageByName('home');
JSON.stringify({missing, broken, theme:document.documentElement.dataset.theme});
```

Esperado: `{"missing":[],"broken":[],"theme":"dark"}`.

> **Armadilha ao verificar tema:** o `body` tem `transition:background 200ms,color 200ms`. Ler `getComputedStyle(...).backgroundColor` logo após `applyTheme()` devolve o valor **interpolado**, ainda próximo do tema anterior — o que parece um bug de tema que não existe. Sempre esperar antes de ler:
>
> ```js
> const lerTema=async t=>{ applyTheme(t); await new Promise(r=>setTimeout(r,350));
>   return getComputedStyle(document.body).backgroundColor; };
> ```
>
> Os custom properties em si (`getPropertyValue('--bg-surface')`) não sofrem transição e podem ser lidos na hora.

Ao final de tudo, derrube o servidor:

```powershell
Get-NetTCPConnection -LocalPort 8777 -State Listen -ErrorAction SilentlyContinue | Select-Object -ExpandProperty OwningProcess -Unique | ForEach-Object { Stop-Process -Id $_ -Force }
```

## File Structure

- **Modify:** `index.html` — único arquivo tocado. É intencionalmente monolítico: a restrição de distribuição é "basta enviar o arquivo". Não dividir em `.css` / `.js` externos.

Dentro dele, o `<style>` passa a ter seções nomeadas por comentário, nesta ordem: TOKENS → RESET → SHELL → COMPONENTES → TELAS → RESPONSIVO. A task 7 garante essa organização.

---

### Task 1: Fundação — tokens e tipografia

**Files:**
- Modify: `index.html` — bloco `<head>` (links de fonte) e o `:root` / `[data-theme="light"]` no início do `<style>`

**Interfaces:**
- Consumes: nada.
- Produces: os custom properties `--sp-1`..`--sp-8`, `--fs-11`..`--fs-30`, `--r-sm/--r-md/--r-lg`, `--accent`, `--accent-h`, `--accent-soft`, `--accent-ring`, `--font-ui`, `--font-mono`, e a escala de neutros. Todas as tasks seguintes consomem esses nomes.

- [x] **Step 1: Adicionar as fontes no `<head>`**

Logo depois da linha do SheetJS, antes do `<style>`:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap">
```

- [x] **Step 2: Substituir o bloco `:root` inteiro**

Trocar o `:root { ... }` atual por:

```css
:root {
  /* neutros — dark */
  --bg-app:      #0b0b0d;
  --bg-surface:  #121214;
  --bg-elevated: #191a1d;
  --bg-hover:    #202126;
  --text-1:      #f4f4f5;
  --text-2:      #a1a1aa;
  --text-3:      #71717a;
  --border-s:    #232428;
  --border-m:    #33353a;

  /* marca */
  --accent:      #ff5b1f;
  --accent-h:    #ff7340;
  --accent-soft: rgba(255,91,31,.12);
  --accent-ring: rgba(255,91,31,.55);
  --accent-tx:   #ffffff;

  /* semânticas */
  --ok:       #4ade80;  --ok-soft:   rgba(74,222,128,.12);
  --err:      #f87171;  --err-soft:  rgba(248,113,113,.12);
  --warn:     #fbbf24;  --warn-soft: rgba(251,191,36,.12);
  --blue:     #5b8def;  --blue-soft: rgba(91,141,239,.15);

  /* placas — identidade do Conversor */
  --plate-trad-bg:#1e2a3a; --plate-trad-bd:#3b82f6; --plate-trad-tx:#93c5fd;
  --plate-merc-bg:#14302a; --plate-merc-bd:#10b981; --plate-merc-tx:#6ee7b7;
  --plate-conv-bg:#27272a; --plate-conv-bd:#52525b; --plate-conv-tx:#d4d4d8;
  --danger-bg:#3a1414; --danger-tx:#fca5a5;

  /* espaçamento */
  --sp-1:4px; --sp-2:8px;  --sp-3:12px; --sp-4:16px;
  --sp-5:20px; --sp-6:24px; --sp-7:32px; --sp-8:40px;

  /* tipografia */
  --font-ui:   "Inter", system-ui, -apple-system, "Segoe UI", sans-serif;
  --font-mono: "JetBrains Mono", ui-monospace, "SF Mono", Menlo, Consolas, monospace;
  --fs-11:11px; --fs-12:12px; --fs-13:13px; --fs-15:15px;
  --fs-18:18px; --fs-24:24px; --fs-30:30px;

  /* forma */
  --r-sm:6px; --r-md:10px; --r-lg:14px;
  --shadow: none;
  --sidebar-w: 232px;
}
```

- [x] **Step 3: Substituir o bloco `[data-theme="light"]`**

```css
[data-theme="light"] {
  --bg-app:      #fbfbfa;
  --bg-surface:  #ffffff;
  --bg-elevated: #f5f5f4;
  --bg-hover:    #ebebe8;
  --text-1:      #18181b;
  --text-2:      #52525b;
  --text-3:      #8b8b93;
  --border-s:    #e7e5e4;
  --border-m:    #d4d4d8;

  --accent:      #ea580c;
  --accent-h:    #c2410c;
  --accent-soft: rgba(234,88,12,.10);
  --accent-ring: rgba(234,88,12,.45);

  --ok-soft:   rgba(22,163,74,.12);
  --err-soft:  rgba(220,38,38,.10);
  --warn-soft: rgba(217,119,6,.12);

  --plate-trad-bg:#dbeafe; --plate-trad-bd:#3b82f6; --plate-trad-tx:#1e40af;
  --plate-merc-bg:#d1fae5; --plate-merc-bd:#10b981; --plate-merc-tx:#065f46;
  --plate-conv-bg:#f4f4f5; --plate-conv-bd:#d4d4d8; --plate-conv-tx:#3f3f46;
  --danger-bg:#fee2e2; --danger-tx:#991b1b;

  --shadow: 0 1px 2px rgba(0,0,0,.05), 0 1px 3px rgba(0,0,0,.06);
}
```

- [x] **Step 4: Aplicar as fontes no `body`**

No seletor `body` existente, trocar a linha `font-family:` por:

```css
  font-family:var(--font-ui);
  font-size:var(--fs-13);
  font-feature-settings:"cv05" 1;
```

E acrescentar, logo após o bloco `body`:

```css
code,kbd,pre,.mono{font-family:var(--font-mono);}
table,.tabular{font-variant-numeric:tabular-nums;}
```

- [x] **Step 5: Verificar no navegador**

Rodar o smoke check padrão. Depois conferir que os tokens existem e que a fonte carregou:

```js
const cs=getComputedStyle(document.documentElement);
const t=['--sp-4','--fs-13','--r-md','--accent-ring','--font-mono'].map(k=>k+'='+cs.getPropertyValue(k).trim());
applyTheme('light'); const lightBg=getComputedStyle(document.body).backgroundColor;
applyTheme('dark');  const darkBg=getComputedStyle(document.body).backgroundColor;
JSON.stringify({t,lightBg,darkBg,inter:document.fonts.check('12px Inter')});
```

Esperado: os cinco tokens com valor não vazio, `lightBg` claro, `darkBg` escuro, `inter:true`. Se `inter:false`, a CDN não respondeu — o layout deve continuar íntegro com `system-ui`, o que é comportamento aceito.

---

### Task 2: Vocabulário de componentes (aditivo)

Nenhuma tela muda nesta task. As classes novas são adicionadas e convivem com as antigas.

**Files:**
- Modify: `index.html` — nova seção `/* === COMPONENTES === */` no `<style>`, inserida **no fim da folha, imediatamente antes de `</style>`**

> **Por que no fim, e não junto do RESET:** o CSS legado define `.btn` (e outros nomes que o vocabulário novo reutiliza) com a mesma especificidade. Em empate de especificidade vence a regra que vier **depois**. Se a seção nova ficar no topo, cada componente novo nasce sobrescrito pelo legado, e as tasks 4 a 6 migrariam o markup sem conseguir verificar o resultado — as telas continuariam com o estilo velho até a limpeza da Task 7. A seção nova no fim garante que cada task migrada é verificável na hora.
>
> O bloco `@media(max-width:640px)` legado fica antes da seção nova, mas não há colisão: ele só toca `.sidebar`, `.main`, `.menu-toggle`, `.topbar`, `.app-wrap`, `.pair`, `.arrow` e `.json-grid`, e nenhum desses nomes pertence ao vocabulário novo.

**Interfaces:**
- Consumes: todos os tokens da Task 1.
- Produces: `.panel`, `.btn` (+ `--primary`, `--ghost`, `--danger`, `--sm`, `--block`), `.field`, `.tag` (+ `ok`, `err`, `warn`), `.status` (+ `info`, `ok`, `warn`, `err`), `.table`, `.drop` (+ `--slot`), `.dot` (+ `ok`, `err`, `warn`). As tasks 4, 5 e 6 migram as telas para esses nomes.

- [x] **Step 1: Adicionar a seção de componentes**

```css
/* ============================================================
   COMPONENTES
   ============================================================ */

/* --- Painel --- */
.panel{
  background:var(--bg-surface);border:1px solid var(--border-s);
  border-radius:var(--r-lg);padding:var(--sp-5);
  margin-bottom:var(--sp-4);box-shadow:var(--shadow);
}
.panel__label{
  display:block;font-size:var(--fs-11);font-weight:600;
  text-transform:uppercase;letter-spacing:.08em;
  color:var(--text-3);margin-bottom:var(--sp-3);
}

/* --- Botões --- */
.btn{
  display:inline-flex;align-items:center;justify-content:center;gap:var(--sp-2);
  padding:9px var(--sp-4);border-radius:999px;
  border:1px solid var(--border-m);background:var(--bg-elevated);color:var(--text-1);
  font-family:inherit;font-size:var(--fs-13);font-weight:500;cursor:pointer;
  transition:background 150ms ease-out,border-color 150ms ease-out,color 150ms ease-out;
}
.btn:hover:not(:disabled){background:var(--bg-hover);border-color:var(--border-m);}
.btn:disabled{opacity:.4;cursor:not-allowed;}
.btn svg{width:14px;height:14px;flex-shrink:0;}
.btn--primary{background:var(--accent);border-color:var(--accent);color:var(--accent-tx);}
.btn--primary:hover:not(:disabled){background:var(--accent-h);border-color:var(--accent-h);}
.btn--ghost{background:transparent;border-color:var(--border-m);color:var(--text-2);}
.btn--ghost:hover:not(:disabled){color:var(--text-1);border-color:var(--text-3);}
.btn--danger{background:transparent;border-color:var(--border-m);color:var(--text-2);}
.btn--danger:hover:not(:disabled){border-color:var(--err);color:var(--err);background:var(--err-soft);}
.btn--sm{padding:5px var(--sp-3);font-size:var(--fs-12);}
.btn--block{width:100%;}
.btn-row{display:flex;gap:var(--sp-3);flex-wrap:wrap;align-items:center;margin-top:var(--sp-4);}

/* --- Campos --- */
.field{
  width:100%;padding:10px var(--sp-3);
  background:var(--bg-elevated);color:var(--text-1);
  border:1px solid var(--border-m);border-radius:var(--r-md);
  font-family:inherit;font-size:var(--fs-13);
  transition:border-color 150ms ease-out;outline:none;
}
.field:hover{border-color:var(--text-3);}
.field--mono{font-family:var(--font-mono);line-height:1.6;}
.field--area{min-height:110px;resize:vertical;}

/* --- Tag --- */
.tag{
  display:inline-flex;align-items:center;gap:var(--sp-1);
  padding:2px var(--sp-2);border-radius:999px;
  font-size:var(--fs-11);font-weight:600;
  background:var(--bg-elevated);color:var(--text-2);border:1px solid var(--border-s);
}
.tag.ok{background:var(--ok-soft);color:var(--ok);border-color:transparent;}
.tag.err{background:var(--err-soft);color:var(--err);border-color:transparent;}
.tag.warn{background:var(--warn-soft);color:var(--warn);border-color:transparent;}

/* --- Ponto de status --- */
.dot{width:6px;height:6px;border-radius:50%;background:var(--text-3);flex-shrink:0;}
.dot.ok{background:var(--ok);}
.dot.err{background:var(--err);}
.dot.warn{background:var(--warn);}

/* --- Mensagem de estado --- */
.status{
  display:flex;align-items:flex-start;gap:var(--sp-3);
  padding:var(--sp-3) var(--sp-4);border-radius:var(--r-md);
  font-size:var(--fs-13);line-height:1.55;margin-bottom:var(--sp-4);
  border:1px solid transparent;
}
.status__icon{width:18px;height:18px;flex-shrink:0;margin-top:1px;}
.status.info{background:var(--blue-soft);color:var(--blue);}
.status.ok  {background:var(--ok-soft);color:var(--ok);}
.status.warn{background:var(--warn-soft);color:var(--warn);}
.status.err {background:var(--err-soft);color:var(--err);}
.status.hidden{display:none;}

/* --- Tabela --- */
.table{width:100%;border-collapse:collapse;font-size:var(--fs-13);}
.table th,.table td{text-align:left;padding:9px var(--sp-3);border-bottom:1px solid var(--border-s);}
.table th{
  color:var(--text-3);font-size:var(--fs-11);font-weight:600;
  text-transform:uppercase;letter-spacing:.06em;
}
.table td.num{font-family:var(--font-mono);font-variant-numeric:tabular-nums;}
.table td.action{text-align:right;}
.table tbody tr:last-child td{border-bottom:none;}
.table tbody tr:hover td{background:var(--bg-elevated);}
.table tr.is-bad td{background:var(--err-soft);}
.table-wrap{
  overflow:auto;max-height:320px;
  border:1px solid var(--border-s);border-radius:var(--r-md);margin-bottom:var(--sp-4);
}
.table-wrap thead th{position:sticky;top:0;background:var(--bg-elevated);z-index:1;}

/* --- Área de upload --- */
.drop{
  display:flex;flex-direction:column;align-items:center;justify-content:center;
  gap:var(--sp-2);text-align:center;cursor:pointer;
  border:1.5px dashed var(--border-m);border-radius:var(--r-md);
  padding:var(--sp-7) var(--sp-5);
  background:var(--bg-elevated);color:var(--text-2);
  transition:border-color 150ms ease-out,background 150ms ease-out,color 150ms ease-out;
}
.drop:hover,.drop.is-over{border-color:var(--accent);color:var(--text-1);background:var(--accent-soft);}
.drop svg{width:30px;height:30px;stroke:var(--accent);stroke-width:1.5;fill:none;}
.drop__main{font-size:var(--fs-15);color:var(--text-1);}
.drop__hint{font-size:var(--fs-12);color:var(--text-3);}
.drop input[type=file]{display:none;}
.drop--slot{padding:var(--sp-4) var(--sp-3);min-height:104px;gap:var(--sp-1);}
.drop--slot svg{width:20px;height:20px;}
.drop--slot.is-filled{
  border-style:solid;border-color:var(--ok);background:var(--ok-soft);cursor:default;
}
.drop--slot.is-filled svg{stroke:var(--ok);}

/* --- Estado vazio --- */
.empty{text-align:center;padding:var(--sp-8) var(--sp-5);color:var(--text-3);}
.empty svg{width:36px;height:36px;opacity:.5;margin-bottom:var(--sp-2);stroke-width:1.5;fill:none;stroke:currentColor;}
.empty p{font-size:var(--fs-13);}
```

- [x] **Step 2: Adicionar foco visível e redução de movimento**

No fim da seção de componentes:

```css
:where(button,a,input,textarea,select,[tabindex]):focus-visible{
  outline:2px solid var(--accent-ring);
  outline-offset:2px;
  border-radius:var(--r-sm);
}
@media (prefers-reduced-motion: reduce){
  *,*::before,*::after{
    transition-duration:.01ms!important;
    animation-duration:.01ms!important;
    animation-iteration-count:1!important;
  }
}
```

- [x] **Step 3: Verificar no navegador**

Rodar o smoke check padrão — nada deve ter mudado visualmente ainda. Depois confirmar que as classes novas existem na folha de estilo:

```js
const want=['.panel','.btn--primary','.field','.tag','.status','.table','.drop','.dot','.empty'];
const found=new Set();
for(const s of document.styleSheets){
  let rules; try{ rules=s.cssRules; }catch(e){ continue; }
  for(const r of rules||[]) if(r.selectorText) want.forEach(w=>{ if(r.selectorText.includes(w)) found.add(w); });
}
JSON.stringify({missing:want.filter(w=>!found.has(w))});
```

Esperado: `{"missing":[]}`.

---

### Task 3: Shell — ícones SVG, navegação agrupada, topbar e home

**Files:**
- Modify: `index.html` — `<nav class="sidebar">`, `.topbar`, `#page-home`, a função `showPage()`, e as regras CSS de `.sidebar`, `.nav-item`, `.topbar`, `.home-*`

**Interfaces:**
- Consumes: tokens da Task 1, `.btn` e `.panel` da Task 2.
- Produces: `.nav-item` com indicador `::before`; o `<symbol>` sprite com os ids `ic-home`, `ic-conversor`, `ic-validador`, `ic-jsoncsv`, `ic-excel`, `ic-sqlin`, `ic-frota`, `ic-consolidador`, `ic-upload`, `ic-download`, `ic-copy`, `ic-trash`, `ic-check`, `ic-alert`, `ic-play`, `ic-file`. As tasks 4–6 referenciam esses ids via `<use href="#ic-...">`.

- [x] **Step 1: Inserir o sprite de ícones no início do `<body>`**

Antes de `<div class="shell">`:

```html
<svg width="0" height="0" style="position:absolute" aria-hidden="true">
  <symbol id="ic-home" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M3 10.5L12 3l9 7.5"/><path d="M5 9.5V21h14V9.5"/><path d="M9.5 21v-6h5v6"/></symbol>
  <symbol id="ic-conversor" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M4 8h13l-3-3"/><path d="M20 16H7l3 3"/></symbol>
  <symbol id="ic-validador" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M20 6L9 17l-5-5"/></symbol>
  <symbol id="ic-jsoncsv" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M8 4H6a2 2 0 00-2 2v12a2 2 0 002 2h12a2 2 0 002-2V6a2 2 0 00-2-2h-2"/><rect x="8" y="2" width="8" height="4" rx="1"/><path d="M8 12h8M8 16h5"/></symbol>
  <symbol id="ic-excel" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="4" width="18" height="16" rx="2"/><path d="M3 10h18M9 10v10M3 15h18"/></symbol>
  <symbol id="ic-sqlin" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="7"/><path d="M20 20l-3.5-3.5"/></symbol>
  <symbol id="ic-frota" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M5 17h14M4 17v-4l2-5h12l2 5v4"/><circle cx="7.5" cy="17.5" r="1.5"/><circle cx="16.5" cy="17.5" r="1.5"/></symbol>
  <symbol id="ic-consolidador" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="4" width="7" height="7" rx="1"/><rect x="3" y="14" width="7" height="7" rx="1"/><path d="M14 12h7M17 8l4 4-4 4"/></symbol>
  <symbol id="ic-upload" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M12 16V4m0 0L7 9m5-5l5 5"/><path d="M4 17v2a1 1 0 001 1h14a1 1 0 001-1v-2"/></symbol>
  <symbol id="ic-download" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M12 4v12m0 0l-5-5m5 5l5-5"/><path d="M4 18v1a1 1 0 001 1h14a1 1 0 001-1v-1"/></symbol>
  <symbol id="ic-copy" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><rect x="9" y="9" width="12" height="12" rx="2"/><path d="M5 15H4a1 1 0 01-1-1V4a1 1 0 011-1h10a1 1 0 011 1v1"/></symbol>
  <symbol id="ic-trash" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M4 7h16M9 7V5a1 1 0 011-1h4a1 1 0 011 1v2"/><path d="M18 7l-1 13a1 1 0 01-1 1H8a1 1 0 01-1-1L6 7"/></symbol>
  <symbol id="ic-check" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="9"/><path d="M8.5 12.5l2.5 2.5 4.5-5"/></symbol>
  <symbol id="ic-alert" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="9"/><path d="M12 7.5v5M12 16h.01"/></symbol>
  <symbol id="ic-play" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M8 5.5l10 6.5-10 6.5z"/></symbol>
  <symbol id="ic-file" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M14 3H7a1 1 0 00-1 1v16a1 1 0 001 1h10a1 1 0 001-1V7z"/><path d="M14 3v4h4"/></symbol>
</svg>
```

- [x] **Step 2: Substituir o CSS de sidebar, nav e topbar**

Trocar as regras `.sidebar`, `.sidebar-logo`, `.logo-*`, `.sidebar-nav`, `.nav-label`, `.nav-item`, `.nav-icon`, `.nav-dot`, `.topbar` por:

```css
.sidebar{
  width:var(--sidebar-w);background:var(--bg-surface);
  border-right:1px solid var(--border-s);
  display:flex;flex-direction:column;
  position:fixed;top:0;left:0;height:100vh;z-index:50;
  transition:transform 200ms ease-out;
}
.sidebar-logo{
  padding:var(--sp-5) var(--sp-4) var(--sp-4);
  border-bottom:1px solid var(--border-s);
  display:flex;align-items:center;gap:var(--sp-3);
}
.logo-badge{
  width:32px;height:32px;border-radius:var(--r-md);
  background:var(--accent);color:#fff;
  display:grid;place-items:center;flex-shrink:0;
}
.logo-badge svg{width:17px;height:17px;}
.logo-text{font-size:var(--fs-13);font-weight:600;line-height:1.2;letter-spacing:-.01em;}
.logo-sub{font-size:var(--fs-11);color:var(--text-3);}

.sidebar-nav{flex:1;padding:var(--sp-3) var(--sp-2);overflow-y:auto;}
.nav-label{
  font-size:var(--fs-11);font-weight:600;text-transform:uppercase;
  letter-spacing:.09em;color:var(--text-3);
  padding:var(--sp-4) var(--sp-2) var(--sp-1);
}
.nav-item{
  position:relative;display:flex;align-items:center;gap:var(--sp-3);
  padding:8px var(--sp-3);border-radius:var(--r-md);
  font-family:inherit;font-size:var(--fs-13);color:var(--text-2);
  cursor:pointer;border:none;background:none;width:100%;text-align:left;
  transition:background 120ms ease-out,color 120ms ease-out;
}
.nav-item::before{
  content:"";position:absolute;left:0;top:50%;transform:translateY(-50%);
  width:2px;height:0;border-radius:1px;background:var(--accent);
  transition:height 150ms ease-out;
}
.nav-item:hover{background:var(--bg-elevated);color:var(--text-1);}
.nav-item.active{background:var(--bg-elevated);color:var(--text-1);font-weight:600;}
.nav-item.active::before{height:16px;}
.nav-item svg{width:17px;height:17px;flex-shrink:0;}

.topbar{
  height:56px;background:color-mix(in srgb,var(--bg-surface) 88%,transparent);
  backdrop-filter:blur(8px);-webkit-backdrop-filter:blur(8px);
  border-bottom:1px solid var(--border-s);
  display:flex;align-items:center;padding:0 var(--sp-6);
  position:sticky;top:0;z-index:40;gap:var(--sp-3);
}
.topbar-title{font-size:var(--fs-13);font-weight:600;}
.topbar-sub{
  font-size:var(--fs-12);color:var(--text-3);
  padding-left:var(--sp-3);border-left:1px solid var(--border-s);
}
.topbar-sub:empty{display:none;}
```

- [x] **Step 3: Substituir o markup da sidebar**

Trocar todo o conteúdo de `<div class="sidebar-nav">` por:

```html
    <div class="sidebar-nav">
      <button class="nav-item active" onclick="showPage('home',this)">
        <svg><use href="#ic-home"/></svg> Início
      </button>

      <div class="nav-label">Placas</div>
      <button class="nav-item" onclick="showPage('conversor',this)">
        <svg><use href="#ic-conversor"/></svg> Conversor de Placas
      </button>
      <button class="nav-item" onclick="showPage('validador',this)">
        <svg><use href="#ic-validador"/></svg> Validador de Placas
      </button>

      <div class="nav-label">Dados</div>
      <button class="nav-item" onclick="showPage('jsoncsv',this)">
        <svg><use href="#ic-jsoncsv"/></svg> JSON → CSV
      </button>
      <button class="nav-item" onclick="showPage('excel',this)">
        <svg><use href="#ic-excel"/></svg> Processador Excel
      </button>
      <button class="nav-item" onclick="showPage('sqlin',this)">
        <svg><use href="#ic-sqlin"/></svg> Conversor SQL IN
      </button>
      <button class="nav-item" onclick="showPage('frota',this)">
        <svg><use href="#ic-frota"/></svg> Unificador de Frota
      </button>
      <button class="nav-item" onclick="showPage('consolidador',this)">
        <svg><use href="#ic-consolidador"/></svg> Consolidador de Arquivos
      </button>
    </div>
```

Também trocar o `<div class="logo-badge">⚡</div>` por:

```html
      <div class="logo-badge"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M13 2L4.5 13.5H11l-1 8.5L19.5 10H13z"/></svg></div>
```

> `showPageByName()` localiza o botão pelo atributo `onclick`, que segue no mesmo formato — nenhuma mudança no JS é necessária aqui.

- [x] **Step 4: Atualizar a home**

Trocar o CSS de `.home-*` e `.cards-grid`:

```css
.home-screen{max-width:840px;margin:0 auto;padding:var(--sp-8) var(--sp-6);}
.home-hero{text-align:center;margin-bottom:var(--sp-7);}
.home-hero h1{font-size:var(--fs-30);font-weight:700;margin-bottom:var(--sp-2);letter-spacing:-.02em;}
.home-hero h1 span{color:var(--accent);}
.home-hero p{color:var(--text-2);font-size:var(--fs-15);}
.cards-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(240px,1fr));gap:var(--sp-4);}
.home-card{
  background:var(--bg-surface);border:1px solid var(--border-s);
  border-radius:var(--r-lg);padding:var(--sp-5);cursor:pointer;
  display:flex;flex-direction:column;gap:var(--sp-3);
  transition:border-color 180ms ease-out,background 180ms ease-out;
}
.home-card:hover{border-color:var(--accent);background:var(--bg-elevated);}
.card-icon-box{
  width:38px;height:38px;border-radius:var(--r-md);
  background:var(--bg-elevated);border:1px solid var(--border-s);
  color:var(--text-2);display:grid;place-items:center;
  transition:color 180ms ease-out,border-color 180ms ease-out;
}
.card-icon-box svg{width:19px;height:19px;}
.home-card:hover .card-icon-box{color:var(--accent);border-color:var(--accent);}
.home-card h3{font-size:var(--fs-15);font-weight:600;}
.home-card p{font-size:var(--fs-13);color:var(--text-2);line-height:1.55;flex:1;}
.card-cta{font-size:var(--fs-12);color:var(--accent);font-weight:600;}
```

E em cada um dos 7 `.home-card`, trocar `<div class="card-icon-box icon-XXX">EMOJI</div>` por `<div class="card-icon-box"><svg><use href="#ic-NOME"/></svg></div>`, onde `NOME` é o id da ferramenta (`conversor`, `validador`, `jsoncsv`, `excel`, `sqlin`, `frota`, `consolidador`). As classes `.icon-orange`, `.icon-green`, `.icon-blue` e `.icon-purple` deixam de ser usadas e são removidas na Task 7.

- [x] **Step 5: Ajustar o subtítulo da Frota em `showPage()`**

No objeto `titles`, a entrada de `frota` tem título longo. Trocar por:

```js
    frota:     ['Unificador de Frota','3 planilhas → 1 CSV'],
```

- [x] **Step 6: Verificar no navegador**

Rodar o smoke check padrão, depois:

```js
const items=[...document.querySelectorAll('.nav-item')];
const labels=[...document.querySelectorAll('.nav-label')].map(l=>l.textContent.trim());
showPageByName('frota');
const activeAfter=document.querySelector('.nav-item.active')?.textContent.trim();
const icons=[...document.querySelectorAll('.nav-item use')].map(u=>u.getAttribute('href'));
const broken=icons.filter(h=>!document.querySelector(h));
JSON.stringify({navCount:items.length,labels,activeAfter,brokenIcons:broken});
```

Esperado: `navCount:8`, `labels:["Placas","Dados"]`, `activeAfter` contendo "Unificador de Frota", `brokenIcons:[]`.

Conferir visualmente com screenshot nos dois temas: a barra laranja de 2px deve aparecer só no item ativo.

---

### Task 4: Migrar Conversor, Validador e SQL IN

As três telas mais simples — só texto, sem upload.

**Files:**
- Modify: `index.html` — `#page-conversor`, `#page-validador`, `#page-sqlin` e as regras `.input-*`, `.copy-*`, `.summary`, `.pill`, `.grupo`, `.item`, `#val-*`, `#sqlin-*`

**Interfaces:**
- Consumes: `.panel`, `.btn`, `.field`, `.tag`, `.empty`, `.table` da Task 2; sprite da Task 3.
- Produces: nenhuma interface nova.

- [x] **Step 1: Migrar as classes do Conversor**

No `#page-conversor`:

| De | Para |
| --- | --- |
| `<section class="input-card">` | `<section class="panel">` |
| `class="input-label section-label"` | `class="panel__label"` |
| `class="input-area"` | `class="field field--area field--mono"` |
| `class="copy-panel"` | `class="panel"` |
| `class="btn btn-soft"` | `class="btn btn--sm"` |
| `class="btn btn-primary"` | `class="btn btn--primary"` |
| `class="empty-state"` | `class="empty"` |

No botão "Converter", trocar o SVG inline por `<svg><use href="#ic-conversor"/></svg>`; no "Limpar", por `<svg><use href="#ic-trash"/></svg>`.

Envolver o texto do empty state em `<p>`: `<p>Cole suas placas e clique em Converter</p>`.

- [x] **Step 2: Ajustar `.toast-error` para `.status`**

O JS do Conversor usa `errorEl.classList.add('show')`. Manter o gancho:

```css
.toast-error{
  display:none;padding:var(--sp-3) var(--sp-4);
  background:var(--err-soft);color:var(--err);
  border-radius:var(--r-md);font-size:var(--fs-13);margin-bottom:var(--sp-4);
}
.toast-error.show{display:block;}
```

- [x] **Step 3: Migrar o Validador**

No `#page-validador`:

- `class="section-label"` → `class="panel__label"`
- `class="input-area"` → `class="field field--area field--mono"`
- `class="btn btn-primary"` → `class="btn btn--primary"`
- `class="btn dl-validas"` → `class="btn btn--sm"` e o rótulo passa a `<svg><use href="#ic-download"/></svg> Válidas (.xlsx)`
- `class="btn dl-invalidas"` → idem, com "Inválidas (.xlsx)"

E o CSS dos grupos de resultado:

```css
.grupo{margin-bottom:var(--sp-5);}
.grupo h2{
  font-size:var(--fs-11);text-transform:uppercase;letter-spacing:.08em;
  margin-bottom:var(--sp-2);padding-bottom:var(--sp-1);
  border-bottom:1px solid var(--border-s);
}
.grupo.validas h2{color:var(--ok);}
.grupo.invalidas h2{color:var(--err);}
.grupo .lista{display:flex;flex-direction:column;gap:var(--sp-2);}
.item{
  padding:var(--sp-3) var(--sp-4);border-radius:var(--r-md);
  font-size:var(--fs-13);display:flex;align-items:baseline;gap:var(--sp-3);
  border:1px solid transparent;
}
.item.ok{background:var(--ok-soft);border-color:rgba(74,222,128,.25);}
.item.erro{background:var(--err-soft);border-color:rgba(248,113,113,.25);}
.linha{flex-shrink:0;font-family:var(--font-mono);color:var(--text-3);min-width:52px;font-variant-numeric:tabular-nums;}
.placa{font-family:var(--font-mono);font-weight:600;letter-spacing:1px;}
.info{color:var(--text-2);}
.sugestao{color:var(--warn);}
```

- [x] **Step 4: Migrar o SQL IN**

No `#page-sqlin`:

- `class="input-card"` → `class="panel"`
- `class="section-label"` → `class="panel__label"`
- `class="input-area"` → `class="field field--area field--mono"`
- `class="btn btn-primary"` → `class="btn btn--primary"`
- `class="empty-state"` → `class="empty"`, com o texto em `<p>`
- O `<span id="sqlin-badge" style="...">` perde o `style` inline e ganha `class="tag"`
- O `<div id="sqlin-box" style="...">` perde o `style` inline e ganha `class="sqlin-box"`, com:

```css
.sqlin-box{
  background:var(--bg-surface);border:1px solid var(--border-s);
  border-radius:var(--r-md);padding:var(--sp-4);
  font-family:var(--font-mono);font-size:var(--fs-13);
  color:var(--text-1);word-break:break-all;white-space:pre-wrap;
  line-height:1.7;margin-bottom:var(--sp-3);
}
```

No botão de copiar, trocar `📋 Copiar` por `<svg><use href="#ic-copy"/></svg> Copiar`.

> **Atenção:** `sqlin_copy()` faz `btn.textContent='✓ Copiado!'` e depois `btn.innerHTML='📋 Copiar'`. Isso destrói o SVG. Trocar as duas linhas por:
> ```js
>       btn.innerHTML='<svg><use href="#ic-check"/></svg> Copiado!';
>       setTimeout(()=>{ btn.innerHTML='<svg><use href="#ic-copy"/></svg> Copiar'; },1800);
> ```
> e o `.catch` por `btn.innerHTML='<svg><use href="#ic-alert"/></svg> Erro ao copiar';`

- [x] **Step 5: Verificar no navegador**

```js
showPageByName('conversor');
document.getElementById('conv-input').value="'HKT0416','RGA2D75','PTG8995'";
document.getElementById('conv-btn-convert').click();
const convCards=document.querySelectorAll('#conv-results .result-card').length;

showPageByName('validador');
document.getElementById('val-input').value='ABC1234\nABC1D23\nXX!123';
val_verificar();
const valItems=document.querySelectorAll('#val-resultado .item').length;

showPageByName('sqlin');
document.getElementById('sqlin-ta').value='TG001\nTG002';
sqlin_convert();
const badge=document.getElementById('sqlin-badge').textContent;

JSON.stringify({convCards,valItems,badge});
```

Esperado: `convCards:3`, `valItems:3`, `badge:"2 itens"`. Conferir que não há erro no console.

---

### Task 5: Migrar JSON → CSV e Processador Excel

**Files:**
- Modify: `index.html` — `#page-jsoncsv`, `#page-excel`, e as regras `.json-*`, `.play-btn`, `.btn-wide`, `.bicon`, `.chev`, `.status-msg`, `.upload-zone`, `.exc-*`, `details.preview-block`

**Interfaces:**
- Consumes: `.panel`, `.btn`, `.field`, `.status`, `.table`, `.table-wrap`, `.drop` da Task 2; sprite da Task 3.
- Produces: `.stat-grid` / `.stat` (usado só por essas telas).

- [x] **Step 1: Trocar `.play-btn` e `.btn-wide` por `.btn`**

No `#page-jsoncsv`, o bloco `.actions-card` vira:

```html
          <div class="actions-card">
            <button class="btn btn--primary btn--block" id="json-btn-process" style="height:56px;">
              <svg><use href="#ic-play"/></svg> Processar
            </button>
            <button class="btn btn--block" id="json-btn-download" disabled>
              <svg><use href="#ic-download"/></svg> Baixar CSV
            </button>
            <button class="btn btn--block" id="json-btn-sql" disabled>
              <svg><use href="#ic-copy"/></svg> Copiar placas SQL IN
            </button>
            <button class="btn btn--block" id="json-btn-sample">
              <svg><use href="#ic-file"/></svg> Carregar exemplo
            </button>
            <button class="btn btn--danger btn--block" id="json-btn-clear">
              <svg><use href="#ic-trash"/></svg> Limpar
            </button>
          </div>
```

E o CSS de apoio:

```css
.json-grid{display:grid;grid-template-columns:1.6fr 1fr;gap:var(--sp-4);margin-bottom:var(--sp-4);}
.actions-card{display:flex;flex-direction:column;gap:var(--sp-2);}
@media(max-width:780px){.json-grid{grid-template-columns:1fr;}}
```

- [x] **Step 2: Unificar os stats**

Trocar `.json-stats` / `.json-stat` / `.json-stat-label` / `.json-stat-value` por:

```css
.stat-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(150px,1fr));gap:var(--sp-3);margin-bottom:var(--sp-4);}
.stat{
  background:var(--bg-surface);border:1px solid var(--border-s);
  border-radius:var(--r-md);padding:var(--sp-3) var(--sp-4);
}
.stat__label{
  font-size:var(--fs-11);color:var(--text-3);text-transform:uppercase;
  letter-spacing:.06em;margin-bottom:var(--sp-1);
}
.stat__value{
  font-family:var(--font-mono);font-size:var(--fs-24);font-weight:500;
  letter-spacing:-.02em;font-variant-numeric:tabular-nums;
}
```

No markup: `class="json-stats"` → `class="stat-grid"`, `class="json-stat"` → `class="stat"`, `json-stat-label` → `stat__label`, `json-stat-value` → `stat__value`. Os ids `json-count-records`, `json-count-cols` e `json-count-plates` **não mudam**.

Aplicar o mesmo em `#page-excel`: `.exc-details` → `class="stat-grid"`, `.exc-detail` → `class="stat"`, `.exc-detail-label` → `stat__label`, `.exc-detail-value` → `stat__value`. Ids `exc-d-name`, `exc-d-sheets`, `exc-d-rows`, `exc-d-size` preservados.

- [x] **Step 3: Migrar as mensagens de estado**

No markup, `<div id="json-status" class="status-msg hidden">` vira `<div id="json-status" class="status hidden">`.

No IIFE do JSON→CSV, substituir o objeto `ICONS` e as funções `showStatus` / `hideStatus` por:

```js
  const ICONS={
    success:'<svg class="status__icon"><use href="#ic-check"/></svg>',
    error:  '<svg class="status__icon"><use href="#ic-alert"/></svg>'
  };
  const STATUS_CLS={success:'ok',error:'err'};
  function showStatus(msg,type){
    statusEl.innerHTML=ICONS[type]+'<div>'+escHtml(msg)+'</div>';
    statusEl.className='status '+STATUS_CLS[type];
  }
  function hideStatus(){statusEl.className='status hidden';}
```

> **Atenção:** o `.status-msg` antigo tinha `white-space:pre-wrap`, e `process()` depende disso — a mensagem de erro de validação é montada com `invalid.join('\n• ')`. Sem isso, a lista de itens inválidos vira uma linha só. Acrescentar na seção de componentes:
> ```css
> .status>div{white-space:pre-wrap;word-wrap:break-word;}
> ```

No IIFE do Processador Excel, substituir `mostrarStatus` por:

```js
  function mostrarStatus(tipo,mensagem){
    const map={info:'info',processando:'warn',sucesso:'ok',erro:'err'};
    const icons={info:'#ic-alert',sucesso:'#ic-check',erro:'#ic-alert'};
    const cls=map[tipo]||'info';
    const icon = tipo==='processando'
      ? '<div class="spinner"></div>'
      : `<svg class="status__icon"><use href="${icons[tipo]||'#ic-alert'}"/></svg>`;
    statusWrap.innerHTML=`<div class="status ${cls}">${icon}<div>${mensagem}</div></div>`;
  }
```

E remover os emojis dos oito call sites de `mostrarStatus`, já que a classe agora carrega a cor e o ícone:

| Antes | Depois |
| --- | --- |
| `'❌ Formato inválido. Use .xls ou .xlsx'` | `'Formato inválido. Use .xls ou .xlsx'` |
| `'❌ Arquivo muito grande (máximo 50MB)'` | `'Arquivo muito grande (máximo 50MB)'` |
| `'⏳ Lendo arquivo...'` | `'Lendo arquivo...'` |
| `'❌ Arquivo Excel vazio'` | `'Arquivo Excel vazio'` |
| `'❌ Nenhum dado encontrado'` | `'Nenhum dado encontrado'` |
| `'⏳ Reorganizando colunas...'` | `'Reorganizando colunas...'` |
| `'✅ Arquivo processado com sucesso!'` | `'Arquivo processado com sucesso!'` |
| `'❌ Erro ao processar: '+e.message` | `'Erro ao processar: '+e.message` |
| `'❌ Erro ao ler o arquivo'` | `'Erro ao ler o arquivo'` |
| `'✅ Arquivo CSV baixado com sucesso!'` | `'Arquivo CSV baixado com sucesso!'` |
| `'ℹ️ Pronto para processar. Selecione um arquivo Excel para começar.'` (2 ocorrências) | `'Pronto para processar. Selecione um arquivo Excel para começar.'` |

Ajustar também `.spinner` para o novo tamanho:

```css
.spinner{
  display:inline-block;width:16px;height:16px;flex-shrink:0;
  border:2px solid var(--border-m);border-top-color:currentColor;
  border-radius:50%;animation:spin .8s linear infinite;
}
@keyframes spin{to{transform:rotate(360deg);}}
```

- [x] **Step 4: Migrar o upload do Excel**

Trocar `class="upload-zone"` por `class="drop"` no `#exc-upload-zone` e o conteúdo interno por:

```html
          <svg><use href="#ic-upload"/></svg>
          <span class="drop__main">Arraste um arquivo Excel aqui</span>
          <span class="drop__hint">ou clique para selecionar (.xls ou .xlsx)</span>
```

> **Atenção:** o JS faz `uploadZone.classList.add('dragover')` e `.remove('dragover')` em quatro lugares. Trocar todas as ocorrências da string `'dragover'` por `'is-over'`, que é a classe da Task 2.

- [x] **Step 5: Migrar o preview do Excel**

`class="exc-preview-wrap"` → `class="table-wrap"`, e na função que monta a tabela, adicionar `class="table"` ao `<table>` gerado. As células numéricas recebem `class="num"`.

O bloco `.exc-transforms` vira `.panel` com `.panel__label`, e a lista usa `.dot.ok` no lugar do `content:"✓"`:

```css
.exc-transforms li{display:flex;align-items:center;gap:var(--sp-2);margin-bottom:var(--sp-1);font-size:var(--fs-12);color:var(--text-2);}
.exc-transforms li::before{content:"";width:5px;height:5px;border-radius:50%;background:var(--ok);flex-shrink:0;}
```

Nos botões de ação: `⬇ Baixar CSV` → `<svg><use href="#ic-download"/></svg> Baixar CSV`; `🔄 Processar outro arquivo` → `<svg><use href="#ic-conversor"/></svg> Processar outro arquivo`.

- [x] **Step 6: Verificar no navegador**

```js
showPageByName('jsoncsv');
document.getElementById('json-btn-sample').click();
document.getElementById('json-btn-process').click();
const stats=['json-count-records','json-count-cols','json-count-plates']
  .map(id=>document.getElementById(id).textContent);
const statusCls=document.getElementById('json-status').className;

showPageByName('excel');
const dropOk=!!document.querySelector('#exc-upload-zone.drop');
const hasDragover=document.documentElement.innerHTML.includes('dragover');

JSON.stringify({stats,statusCls,dropOk,hasDragover});
```

Esperado: `stats` com números maiores que zero, `statusCls` contendo `status ok`, `dropOk:true`, `hasDragover:false`.

Depois subir um `.xlsx` real de teste e confirmar que o preview renderiza e o CSV baixa.

---

### Task 6: Migrar Frota e Consolidador

As duas telas de maior risco: ambas montam HTML por template string dentro do JS.

**Files:**
- Modify: `index.html` — `#page-frota`, `#page-consolidador`, as regras `.frota-*` e `#page-consolidador *`, e as funções `frota_renderSlot()`, `cons_render()` (o `render()` interno do Consolidador) e `renderWidths()`

**Interfaces:**
- Consumes: `.panel`, `.btn`, `.field`, `.tag`, `.status`, `.table`, `.drop--slot`, `.dot` da Task 2; sprite da Task 3.
- Produces: `.pipeline` / `.pipeline__dot` (usado só pela Frota).

- [x] **Step 1: Substituir o CSS da Frota**

```css
.pipeline{
  display:flex;align-items:center;gap:var(--sp-2);
  padding:var(--sp-4);margin-bottom:var(--sp-5);
  background:var(--bg-elevated);border:1px solid var(--border-s);border-radius:var(--r-md);
}
.pipeline__step{display:flex;flex-direction:column;align-items:center;gap:var(--sp-1);flex:1;}
.pipeline__step span{font-size:var(--fs-11);color:var(--text-3);}
.pipeline__line{height:1px;flex:.5;background:var(--border-s);margin-bottom:18px;}
.pipeline__dot{
  width:30px;height:30px;border-radius:var(--r-sm);
  background:var(--bg-surface);border:1px solid var(--border-m);
  display:grid;place-items:center;
  font-family:var(--font-mono);font-size:var(--fs-12);color:var(--text-3);
  transition:border-color 200ms ease-out,color 200ms ease-out,background 200ms ease-out;
}
.pipeline__dot.active{border-color:var(--accent);color:var(--accent);background:var(--accent-soft);}
.pipeline__dot.done{border-color:var(--ok);color:var(--ok);background:var(--ok-soft);}
.progress{height:5px;border-radius:999px;background:var(--bg-elevated);border:1px solid var(--border-s);overflow:hidden;}
.progress__fill{height:100%;width:0;background:var(--accent);border-radius:999px;transition:width 400ms ease-out;}
```

No markup, `.frota-dot` → `.pipeline__dot` (os **ids** `frota-dot-1`, `frota-dot-2`, `frota-dot-3`, `frota-dot-merge`, `frota-dot-csv` seguem iguais), e os `style` inline do pipeline dão lugar a `.pipeline`, `.pipeline__step` e `.pipeline__line`.

> `frota_dot()` faz `el.classList.remove('active','done')` — essas classes não mudam de nome, então a função continua válida.

- [x] **Step 2: Atualizar `frota_renderSlot()`**

Essa função reescreve o slot inteiro. Substituir por:

```js
  function frota_renderSlot(idx){
    const el=document.querySelector(`.frota-slot[data-slot="${idx}"]`);
    if(!el) return;
    const file=frota_slotFiles[idx];
    if(file){
      el.classList.add('is-filled');
      el.removeAttribute('onclick');
      el.innerHTML=`
        <span class="drop__hint">ARQUIVO ${idx+1}</span>
        <svg><use href="#ic-check"/></svg>
        <span class="drop__main" style="font-size:var(--fs-12);word-break:break-all;">${escHtml(file.name)}</span>
        <button type="button" class="btn btn--sm btn--danger" onclick="frota_removeSlot(${idx},event)">remover</button>`;
    } else {
      el.classList.remove('is-filled');
      el.setAttribute('onclick',`frota_triggerPick(${idx})`);
      el.innerHTML=`
        <span class="drop__hint">ARQUIVO ${idx+1}</span>
        <svg><use href="#ic-file"/></svg>
        <span class="drop__hint">Clique ou arraste</span>`;
    }
  }
```

E no markup dos três slots, a classe passa a `class="frota-slot drop drop--slot"` — `frota-slot` é mantida **apenas** como seletor de dados para `data-slot`, sem estilo próprio.

- [x] **Step 3: Migrar o alerta e o resultado da Frota**

`#frota-alert` perde o `style` inline e ganha `class="status err"`, começando com `hidden`:

```html
          <div id="frota-alert" class="status err hidden"></div>
```

Ajustar as duas funções:

```js
  function frota_showAlert(html){
    const b=document.getElementById('frota-alert');
    b.innerHTML='<svg class="status__icon"><use href="#ic-alert"/></svg><div>'+html+'</div>';
    b.classList.remove('hidden');
  }
  function frota_hideAlert(){document.getElementById('frota-alert').classList.add('hidden');}
```

O `#frota-progress-wrap` usa as novas `.progress` / `.progress__fill` (ids preservados). O botão de download vira `class="btn btn--primary btn--block"` com `<svg><use href="#ic-download"/></svg> Baixar CSV Unificado`, sem o `style` inline de cor verde.

- [x] **Step 4: Migrar o markup do Consolidador**

- `class="input-card"` → `class="panel"` (4 ocorrências)
- `class="section-label"` → `class="panel__label"`
- `class="cons-drop"` → `class="drop"`, e o `svg` interno vira `<svg><use href="#ic-upload"/></svg>`
- `class="cons-hint"` → `class="drop__hint"` dentro do drop; fora dele, `class="hint"` com:
  ```css
  .hint{color:var(--text-3);font-size:var(--fs-12);margin-top:var(--sp-2);}
  ```
- `id="cons-fname"` recebe `class="field"`, e `id="cons-fmt"` também
- As duas `<table>` recebem `class="table"`
- No JS, `drop.classList.add('over')` / `.remove('over')` → `'is-over'` (3 ocorrências)

- [x] **Step 5: Atualizar `render()` e `renderWidths()` do Consolidador**

Dentro do IIFE do Consolidador, as linhas de tabela são montadas por template string. Substituir os trechos correspondentes:

```js
      const rm=`<td class="action"><button class="btn btn--sm btn--danger" onclick="cons_removerArquivo(${i})">Remover</button></td>`;
      if(p.error){
        tr.className='is-bad';
        tr.innerHTML=`<td>${esc(p.name)}</td><td class="num">—</td><td class="num">—</td>
          <td><span class="tag err">${esc(p.error)}</span></td>${rm}`;
      }else{
        const diverge = p.ncols!==refCols;
        if(diverge) tr.className='is-bad';
        tr.innerHTML=`<td>${esc(p.name)}</td><td class="num">${p.nrows}</td><td class="num">${p.ncols}</td>
          <td><span class="tag ${diverge?'err':'ok'}">${diverge?'divergente':'ok'}</span></td>${rm}`;
      }
```

Os alertas:

```js
    if(errored.length) msgs.push(`<div class="status err"><svg class="status__icon"><use href="#ic-alert"/></svg><div>${errored.length} arquivo(s) com erro serão ignorados.</div></div>`);
    if(diverging.length && valid.length)
      msgs.push(`<div class="status err"><svg class="status__icon"><use href="#ic-alert"/></svg><div>Estrutura divergente: esperado ${refCols} coluna(s). ${diverging.length} arquivo(s) fora do padrão (destacados).</div></div>`);
    if(valid.length && !diverging.length)
      msgs.push(`<div class="status ok"><svg class="status__icon"><use href="#ic-check"/></svg><div>Todos os ${valid.length} arquivo(s) válidos têm ${refCols} coluna(s). Prontos para consolidar.</div></div>`);
```

E em `renderWidths()`:

```js
      tr.innerHTML=`<td class="num">${c+1}</td><td>${nome}</td><td class="num">${w}</td><td><code>${sug}</code></td>`;
```

- [x] **Step 6: Verificar no navegador**

```js
showPageByName('consolidador');
const mk=(n,t)=>new File([t],n,{type:'text/csv'});
const dt=new DataTransfer();
dt.items.add(mk('a.csv','placa,uf,modelo\nABC1234,SP,GOL\nXYZ9876,RJ,ONIX PLUS TURBO\n'));
dt.items.add(mk('b.csv','placa,uf,modelo\nQWE1A23,MG,HB20\n'));
dt.items.add(mk('c.csv','placa,uf\nRTY4567,BA\n'));
const inp=document.getElementById('cons-file-input');
inp.files=dt.files; inp.dispatchEvent(new Event('change'));
await new Promise(r=>setTimeout(r,600));
document.getElementById('cons-has-header').checked=true; cons_render();
const widths=[...document.querySelectorAll('#cons-width-body tr')].map(r=>[...r.cells].map(c=>c.textContent).join('|'));
const tags=[...document.querySelectorAll('#cons-report-body .tag')].map(t=>t.className+':'+t.textContent);
const hint=document.getElementById('cons-export-hint').textContent;
JSON.stringify({widths,tags,hint});
```

Esperado — idêntico ao comportamento de hoje, só com classes novas:
- `widths`: `["1|placa|7|VARCHAR(9)","2|uf|2|VARCHAR(3)","3|modelo|15|VARCHAR(18)"]`
- `tags`: dois `tag ok:ok` e um `tag err:divergente`
- `hint`: `"2 arquivo(s) serão unidos."`

Para a Frota, verificar os slots:

```js
showPageByName('frota');
frota_resetTudo();
const slot=document.querySelector('.frota-slot[data-slot="0"]');
JSON.stringify({
  cls:slot.className,
  icon:slot.querySelector('use')?.getAttribute('href'),
  btnDisabled:document.getElementById('frota-process-btn').disabled
});
```

Esperado: `cls` contendo `drop drop--slot`, `icon:"#ic-file"`, `btnDisabled:true`.

---

### Task 7: Limpeza de CSS morto e varredura final

**Files:**
- Modify: `index.html` — bloco `<style>` inteiro (reorganização e remoção)

**Interfaces:**
- Consumes: tudo das tasks anteriores.
- Produces: nada.

- [x] **Step 1: Listar as classes órfãs**

No console da página:

```js
const html=document.documentElement.innerHTML;
const dead=[];
for(const s of document.styleSheets){
  let rules; try{ rules=s.cssRules; }catch(e){ continue; }
  for(const r of rules||[]){
    if(!r.selectorText) continue;
    for(const sel of r.selectorText.split(',')){
      const m=sel.trim().match(/^\.([a-zA-Z][\w-]*)$/);
      if(m && !document.querySelector('.'+m[1]) && !html.includes(m[1])) dead.push(m[1]);
    }
  }
}
JSON.stringify([...new Set(dead)]);
```

Candidatas esperadas: `input-card`, `input-area`, `input-hint`, `copy-panel`, `section-label`, `btn-primary`, `btn-soft`, `btn-wide`, `play-btn`, `bicon`, `chev`, `label-inner`, `json-stats`, `json-stat`, `json-stat-label`, `json-stat-value`, `status-msg`, `status-msg-icon`, `upload-zone`, `upload-icon`, `exc-status`, `exc-details`, `exc-detail`, `exc-detail-label`, `exc-detail-value`, `exc-preview-wrap`, `frota-dot`, `cons-drop`, `cons-hint`, `cons-badge`, `cons-alert`, `cons-btn-remove`, `icon-orange`, `icon-green`, `icon-blue`, `icon-purple`, `nav-dot`, `nav-icon`, `empty-state`, `pill`.

> Conferir cada uma antes de apagar. `.pill` e `.summary` são usadas pelo Conversor via `innerHTML` no JS — se aparecerem na lista, é falso positivo do teste de `innerHTML`; verificar no código antes de remover.

- [x] **Step 2: Remover as regras confirmadas como órfãs**

Apagar do `<style>` apenas o que a Step 1 confirmou e a inspeção manual validou.

- [x] **Step 3: Reorganizar o `<style>` em seções nomeadas**

Ordem final, com comentário de cabeçalho em cada bloco:

```
/* === TOKENS === */
/* === RESET === */
/* === SHELL (sidebar, topbar, main, home) === */
/* === COMPONENTES === */
/* === TELAS (regras específicas por ferramenta) === */
/* === RESPONSIVO === */
```

- [x] **Step 4: Revisar o bloco responsivo**

Atualizar o `@media(max-width:640px)` para refletir os nomes novos:

```css
@media(max-width:640px){
  .sidebar{transform:translateX(-100%);}
  .sidebar.open{transform:translateX(0);}
  .main{margin-left:0;}
  .menu-toggle{display:flex;}
  .topbar{padding:0 var(--sp-4);}
  .app-wrap{padding:var(--sp-5) var(--sp-4) var(--sp-8);}
  .pair{flex-direction:column;align-items:stretch;}
  .arrow{transform:rotate(90deg);padding:0;}
  .json-grid{grid-template-columns:1fr;}
  .pipeline{overflow-x:auto;}
}
```

- [x] **Step 4b: Resolver os resíduos herdados das tasks anteriores**

Dois itens levantados na revisão da Task 4 e conscientemente adiados até aqui:

1. **`.dl-validas` / `.dl-invalidas` anulam o botão novo.** As classes precisam continuar existindo — `val_verificar()` faz `document.querySelector('.dl-validas').disabled=...` e quebraria sem elas. O problema são os `!important`, que sobrescrevem `.btn--sm` e fazem os dois botões ignorarem o redesign. Trocar as duas regras por versões aditivas, sem `!important`, mantendo só a cor semântica:

```css
.dl-validas{border-color:var(--ok);color:var(--ok);}
.dl-invalidas{border-color:var(--err);color:var(--err);}
```

2. **`#val-input` ainda tem `style="min-height:160px;"` inline.** Remover o atributo e expressar via classe:

```css
#val-input{min-height:160px;}
```

3. **`.num` está sendo aplicado a identificadores no preview do Excel.** A heurística usada é `v!=='' && !isNaN(v)`, que classifica `numero_renavam` e `codigo_empresa` como numéricos. São identificadores, não quantidades — alinhar à direita com numeral tabular passa a leitura errada. Restringir a classe às colunas que de fato são quantidade, ou, como este preview não tem nenhuma, remover a aplicação de `.num` nele e manter a classe só onde há contagem (tabelas do Consolidador).

4. **Três resíduos que a Task 5 deixou passar nas telas dela** (encontrados durante a Task 6, fora do escopo de quem os achou):

| Linha | Hoje | Deve virar |
| --- | --- | --- |
| `#page-jsoncsv` | `<div class="input-card" style="margin-bottom:0;">` | `<div class="panel" style="margin-bottom:0;">` |
| `#page-excel` | `<div class="section-label" ...>👁 Pré-visualização (primeiras 10 linhas)</div>` | `<div class="panel__label" ...>Pré-visualização (primeiras 10 linhas)</div>`, sem o emoji |
| `#page-excel` | `<button class="btn btn-primary" onclick="exc_download()">` | `<button class="btn btn--primary" onclick="exc_download()">` |

O terceiro é o mais relevante: `btn-primary` é a sintaxe legada de um traço, então esse botão está pegando a regra antiga em vez do componente novo.

> **Escopo reduzido da Step 1/2:** a Task 5 já removeu `.play-btn`, `.btn-wide`, `.bicon`, `.chev`, `.label-inner`, `.json-stats`/`.json-stat*`, `.status-msg*`, `.upload-zone*`/`.upload-icon`, `.exc-status*`, `.exc-details`/`.exc-detail*`, `.exc-preview-wrap*` e a `.spinner` duplicada. A varredura de órfãs vai encontrar menos coisa do que a lista original previa — isso é esperado, não sinal de que a varredura falhou.

- [x] **Step 5: Remapear as referências remanescentes a `--r-xl`**

A Task 1 removeu a **definição** de `--r-xl`, mas duas regras ainda o **referenciam**: `.input-card` e `details.preview-block`. Enquanto isso, `border-radius:var(--r-xl)` resolve para inválido e o canto vira 0px.

A `.input-card` some junto com a migração para `.panel` nas tasks 4–6. Já `details.preview-block` continua em uso pelo JSON→CSV e **não** é código morto — a Step 1 não vai listá-la. Trocar explicitamente:

```css
details.preview-block{background:var(--bg-surface);border-radius:var(--r-lg);border:1px solid var(--border-s);overflow:hidden;}
```

Confirmar que nenhuma referência sobrou:

```bash
grep -c 'r-xl' index.html
```

Esperado: `0`.

- [x] **Step 6: Corrigir a estrutura quebrada do `#page-sqlin`**

O `#page-sqlin` fecha com `</div><!-- /main --></div><!-- /shell -->` no meio do documento, antes das telas de Frota e Consolidador. O navegador tolera, mas é markup inválido. Fechar `#page-sqlin` e seu `.app-wrap` corretamente e mover os dois `</div>` de `/main` e `/shell` para depois da última `.app-page`.

- [x] **Step 6: Verificação final completa**

Rodar o smoke check padrão, depois exercitar as sete ferramentas nos dois temas, repetindo as verificações das tasks 4, 5 e 6. Conferir também:

```js
JSON.stringify({
  focoVisivel: [...document.styleSheets].some(s=>{
    try{ return [...s.cssRules].some(r=>r.selectorText&&r.selectorText.includes('focus-visible')); }
    catch(e){ return false; }
  }),
  emojisRestantes: (document.documentElement.innerHTML.match(/[\u{1F300}-\u{1FAFF}\u{2600}-\u{27BF}]/gu)||[]).length
});
```

Esperado: `focoVisivel:true`, `emojisRestantes:0`.

Tirar screenshot da home e de duas ferramentas em dark e em light. Derrubar o servidor.

## Self-Review

**Cobertura do spec:**

| Requisito do spec | Task |
| --- | --- |
| Tokens de espaçamento, raio, neutros, laranja, elevação | 1 |
| Tipografia (Inter, JetBrains Mono, escala, tabular-nums) | 1 |
| Sidebar com barra ativa e ícones SVG | 3 |
| Navegação agrupada em Placas / Dados | 3 |
| Topbar 56px com blur | 3 |
| Home contida, sem translateY | 3 |
| Remoção dos emojis (sidebar, cards e rótulos de botão) | 3, 4, 5, 6 |
| `.drop` unificado | 2 (definição), 5 e 6 (adoção) |
| `.btn` unificado | 2, 4, 5, 6 |
| `.panel`, `.table`, `.status`, `.tag`, `.field` | 2, 4, 5, 6 |
| `.plate-box` preservado como componente próprio | não migrado, por decisão do spec |
| `:focus-visible` | 2 |
| `prefers-reduced-motion` | 2 |
| Empty state uniforme | 2 (definição), 4 (adoção) |
| Remoção de `--r-xl` | 1 (não é redefinido) e 7 (usos remapeados) |
| Riscos de HTML-em-string (Frota, Consolidador) | 6 |
| Verificação das 7 ferramentas | 4, 5, 6, 7 |

Sem lacunas.

**Consistência de nomes:** `is-over` (não `dragover` nem `over`) é a classe de arraste em todos os lugares — Excel (Task 5, Step 4), Consolidador (Task 6, Step 4) e definição em `.drop` (Task 2). `is-filled` (não `filled`) é o slot preenchido — definição na Task 2, uso na Task 6, Step 2. `is-bad` (não `bad`) é a linha de tabela destacada — definição em `.table` na Task 2, uso na Task 6, Step 5. As classes `active` e `done` do pipeline da Frota mantêm o nome de propósito, porque `frota_dot()` as manipula por string.
