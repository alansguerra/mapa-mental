# Mapa mental

Editor de mapas mentais que roda no navegador, com os mapas salvos no Google Drive do usuário. Interface em pt-BR, funciona no PC e no celular (toque, pinça para zoom).

## Organização

O projeto é um único arquivo, [index.html](index.html), sem build e sem dependências de npm. Ele tem três partes:

- `<style>`: CSS com tokens em `:root` (cores, sombras) e media queries para telas com menos de 700px.
- Markup: barra superior (`#top`), menu (`#menu`), telas de login (`#login`) e da lista de mapas (`#home`), canvas (`#stage` > `#world` com `#links` em SVG e `#nodes` em divs), barra de ações (`#actions`), zoom e painel lateral de detalhes (`#panel`).
- `<script>`: uma IIFE em JS puro, dividida por comentários `/* ---------- seção ---------- */`: state, undo, IndexedDB, Google auth, Drive API, save pipeline, layout, render, node ops, editing, pointer, keyboard, toolbar, panel, export/import, views e boot.

Dependências externas: Google Identity Services (`accounts.google.com/gsi/client`) e a fonte Figtree (Google Fonts).

As telas são trocadas pela classe do `body`: `v-login`, `v-home` ou `v-editor` (função `showView`).

## Modelo de dados

```js
map = { id, title, root, nodes: { [id]: node }, view: {x,y,k}, driveId }
node = { id, text, parent, children: [ids], color, icon, notes, att: [], collapsed, dx, dy, side }
```

- `side` (`'R'` ou `'L'`) só vale para os filhos diretos da raiz; os outros nós herdam o lado do ramo.
- `dx`/`dy` guardam o deslocamento manual em relação ao layout automático.
- `x`, `y`, `w`, `h` e `_sh` são calculados a cada render e retirados por `cleanMap()` antes de salvar.
- Os anexos (`att`) podem ser `kind: 'link'`, `'drive'` ou `'file'`. O tipo `'file'` é legado, com o arquivo embutido em base64, e é migrado para o Drive na importação.

## Principais funcionalidades

- **Login com Google** (OAuth por token, escopo `drive.file`). O token fica no `localStorage` e é renovado sem interação perto de expirar.
- **Persistência**: cada mapa é um arquivo JSON na pasta "Mapas mentais" do Drive, marcado com `appProperties.mapa=1`. Os anexos vão para a subpasta "Anexos". O salvamento é automático, com debounce de 900ms.
- **Offline**: uma cópia local fica no IndexedDB (`mapamental2`, chaves `index`, `map:<id>` e `dirty:<id>`). As alterações pendentes são enviadas quando a conexão volta.
- **Edição**: adicionar filho ou irmão, editar o texto, apagar, recolher ou expandir ramos, e arrastar um nó sobre outro para mover o ramo. Desfazer e refazer guardam até 60 snapshots em JSON.
- **Layout automático** em árvore horizontal com ramos para os dois lados da raiz, links em curvas bézier e cores por ramo de primeiro nível.
- **Painel de detalhes**: cor, ícone (emoji), notas e anexos (arquivo no Drive ou link).
- **Navegação**: pan, zoom com roda, Ctrl+roda ou pinça, e enquadrar (`fit`).
- **Exportação**: cópia em JSON (Ctrl+S), imagem PNG desenhada em canvas e importação de JSON.
- **Atalhos**: Tab (filho), Enter (irmão), Delete, F2 ou espaço (editar), setas (navegar) e Ctrl+Z ou Ctrl+Shift+Z.

## Convenções

- O código é compacto, com funções de uma linha e muitas declarações por linha. Siga esse estilo.
- `$` é um atalho para `document.querySelector`, e `N(id)` devolve o nó do mapa.
- Toda alteração no mapa segue a ordem `snapshot()`, depois a mutação, depois `render()` e por fim `save()`.
- Os textos da interface são em português.
