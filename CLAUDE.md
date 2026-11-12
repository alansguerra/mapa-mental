# Mapa mental

Editor de mapas mentais que roda no navegador, com os mapas salvos no Google Drive do usuário. Interface em pt-BR, funciona no PC e no celular, com tema claro e escuro.

## Arquivos

- [index.html](index.html): o app. É um arquivo único, sem build e sem npm.
- [antigo.html](antigo.html): a versão anterior, guardada como reserva. **Não editar.**

Dependências externas: Google Identity Services (`accounts.google.com/gsi/client`) e a fonte Schibsted Grotesk (Google Fonts).

## Organização do index.html

- `<style>`: tokens em `:root` e em `html[data-theme=dark]`.
- Markup: barra superior, busca (`#find`), menus (`#menu`, `#pop`), telas `#login` e `#home`, canvas (`#stage` > `#world` com `#links` em SVG, `#nodes` e `#dropline`), barra de ações, zoom, painel `#panel` e diálogo `#dlg`.
- `<script>`: uma IIFE em JS puro, dividida por comentários `/* ---------- seção ---------- */`.

A tela ativa é a classe do `body` (`v-login`, `v-home` ou `v-editor`, via `showView`). Os elementos aparecem conforme as classes `in-login`, `in-home` e `in-editor`.

## Modelo de dados

```js
map  = { id, title, root, nodes: { [id]: node }, view, driveId }
node = { id, text, parent, children: [ids], color, icon, notes, att: [], collapsed, dx, dy, side }
```

- `side` (`'R'` ou `'L'`) só vale para os filhos diretos da raiz.
- `x`, `y`, `w`, `h` e `_sh` são calculados e saem na serialização (`ser()` e `cleanMap()`).
- Os anexos podem ser `kind: 'link'`, `'drive'` ou `'file'`. O tipo `'file'` é legado (base64) e é migrado para o Drive na importação.
- Na IndexedDB `mapamental2` ficam as chaves `index`, `map:<id>`, `dirty:<id>`, `view:<id>`, `mod:<id>` e `base:<id>` (a última versão que o aparelho sabe que está no Drive).

## Regras

- **Compatibilidade:** o formato do mapa no Drive tem que continuar compatível com o `antigo.html`. Não renomeie nem remova campos de `map` e `node`.
- **Salvamento:** `rev` sobe a cada alteração, e `savedRev` é a última revisão que chegou ao Drive. Nunca marque como salvo sem comparar os dois. A cópia local é gravada em 250ms e o Drive em 1,5s.
- **Visão:** mover a tela e dar zoom não chamam `save()`. Eles usam `saveView()`, que grava só neste aparelho, na chave `view:<id>`.
- **Seleção e render:** selecionar um nó não chama `render()`; use `select()` ou `markSel()`. `render()` só refaz o HTML de um nó quando a assinatura dele (`el._sig`) muda.
- **Arrastar:** arrastar reorganiza a estrutura com `moveNode`. A posição livre (`dx`/`dy`) só acontece com Alt ou Shift.
- **Sincronização:** com o mapa na tela, `checkRemote()` confere o Drive a cada 5s e recarrega quando outro aparelho o alterou. Não remova essa função. `flush()` nunca envia por cima de uma versão do Drive que o aparelho não viu: nesse caso, `conflict()` junta as duas com `merge3(base, daqui, Drive)`, tópico por tópico. Se o mesmo texto mudou nos dois, fica o daqui e o outro vai para as notas. Nada pode se perder numa junção.
- **Login:** o token do Google dura 1h. O `pointerdown` renova em silêncio quando ele está perto de vencer ou já venceu, e `reconnected()` envia o pendente e busca o Drive. Sem login o app não sincroniza nada, então não deixe o usuário editar por muito tempo sem perceber.
- **Versão:** ao publicar uma mudança visível, atualize o texto `.ver` no fim do menu, que serve para conferir se o aparelho carregou a versão nova.
- **Fluxo de alteração:** toda alteração no mapa segue a ordem `snapshot()`, depois a mutação, depois `render()` e por fim `save()`.

## Funcionalidades

- Login com Google (escopo `drive.file`). Os mapas ficam na pasta "Mapas mentais" e os anexos na subpasta "Anexos".
- Modo offline com cópia local. As alterações pendentes sobem quando a conexão volta.
- Criar, editar, apagar, duplicar, recolher e reordenar tópicos.
- Copiar, recortar e colar ramos. Um texto colado de fora vira tópicos, um por linha, com a hierarquia pela indentação.
- Busca em textos e notas (Ctrl+F).
- Painel de detalhes com cor, ícone, notas e anexos.
- Exportação em JSON, PNG e TXT, e importação de JSON.
- Na tela inicial: buscar, renomear, duplicar e apagar mapas.

## Convenções

- O código é compacto, com funções de uma linha e muitas declarações por linha. Siga esse estilo.
- `$` e `$$` são atalhos de `querySelector` e `querySelectorAll`, e `N(id)` devolve o nó do mapa.
- Use `dialog()` em vez de `alert`, `confirm` e `prompt`.
- Os textos da interface são em português.
