# Codex Task — Natitur: Integrar Logomarca Real no App e no PDF

## Contexto

Este repositório contém o app `index.html` da **Natitur – natureza · viagens**: um orçamento de churrasco mobile-first, single-file HTML, com exportação em PDF via jsPDF.

O arquivo `natitur.png` no repositório contém a **brand sheet oficial** da Natitur com três variações da logo em uma única imagem:
- **Variação 1 (logo principal)** — ícone circular + texto "Natítur" + tagline "natureza · viagens" → usar no **header do app HTML**
- **Variação 2 (ícone/símbolo)** — somente o círculo com montanhas, bússola e folha, sem texto → usar no **PDF como marca d'água leve no cabeçalho** e no rodapé
- **Variação 3 (monocromática)** — versão em preto e branco → reserva, não usar agora

A imagem tem **fundo branco**. No header do app (fundo verde escuro `#0b1a12`), a logo precisa de tratamento para integrar bem visualmente — ver instruções abaixo.

---

## O que fazer

### 1. Declarar constante base64 global

No topo do bloco `<script>`, **antes de qualquer outra declaração**, adicionar:

```js
// Logo principal (ícone + texto) — usar no header HTML
const LOGO_PRINCIPAL_B64 = "data:image/png;base64,SUBSTITUIR_COM_BASE64_DE_natitur.png_CROP_VARIACAO_1";

// Ícone isolado (somente o círculo) — usar no PDF
const LOGO_ICONE_B64 = "data:image/png;base64,SUBSTITUIR_COM_BASE64_DE_natitur.png_CROP_VARIACAO_2";
```

> **Instrução de crop:** A imagem `natitur.png` é uma brand sheet. Use a biblioteca `sharp` ou `canvas` via Node.js para recortar as duas regiões antes de converter para base64. Se não conseguir recortar automaticamente, use a imagem inteira (`natitur.png`) convertida para base64 como `LOGO_PRINCIPAL_B64` — o CSS cuidará do recorte via `object-position`.

**Alternativa simples (preferencial se crop for complexo):** converter o arquivo `natitur.png` inteiro para base64 e usar como `LOGO_PRINCIPAL_B64`. No CSS, usar `object-fit: cover` + `object-position` para enquadrar a variação 1.

---

### 2. Header do App HTML — substituir SVG pela logo real

**Localizar** o bloco:
```html
<div class="logo-wrap">
  <svg class="logo-svg" viewBox="0 0 56 56" ...>
    ...SVG manual...
  </svg>
  <div class="logo-text">
    <div class="logo-name">Na<em>tí</em>tur</div>
    <div class="logo-tagline">natureza · viagens</div>
  </div>
</div>
```

**Substituir por:**
```html
<div class="logo-wrap">
  <img
    id="logoHeader"
    src=""
    alt="Natitur"
    style="height:40px; width:auto; display:block;"
  />
</div>
```

E no script, após declarar `LOGO_PRINCIPAL_B64`, adicionar:
```js
document.getElementById('logoHeader').src = LOGO_PRINCIPAL_B64;
```

> **Tratamento visual no fundo escuro:** Como a logo tem fundo branco e o header é verde escuro (`#0b1a12`), aplicar `mix-blend-mode: multiply` no CSS do `#logoHeader`. Isso funde o branco com o fundo escuro, tornando apenas os elementos coloridos visíveis. Testar se o resultado é aceitável — se não for, usar `filter: brightness(0) invert(1)` apenas no ícone (versão monocromática branca). Escolher o que ficar melhor visualmente.

Remover também as classes CSS que não serão mais usadas: `.logo-svg`, `.logo-text`, `.logo-name`, `.logo-tagline`.

---

### 3. PDF (jsPDF) — substituir drawLogo() por addImage()

**Localizar** a função `drawLogo(lx, ly, size)` (aproximadamente linha 1950) e **substituí-la completamente** por:

```js
function drawLogo(lx, ly, size) {
  if (!LOGO_ICONE_B64) return;
  // Proporção aproximada do ícone circular: 1:1
  doc.addImage(LOGO_ICONE_B64, 'PNG', lx, ly, size, size);
}
```

> O ícone circular (variação 2) é quadrado, portanto `width = height = size`. O valor `size` passado na chamada é `32` (mm), resultando em um ícone de 32×32mm no PDF — tamanho adequado para cabeçalho A4.

**Localizar** a chamada existente:
```js
drawLogo(MARGIN, 8, 32);
```
Manter como está — só a função interna muda.

**Após o `drawLogo()`**, ajustar as coordenadas do texto "Natitur" e tagline para não sobrepor a imagem. O texto começa em `MARGIN + 38` — verificar se a imagem de 32mm não ultrapassa esse ponto; se ultrapassar, ajustar para `MARGIN + 36`.

---

### 4. Eliminar todos os emojis do PDF

O jsPDF com fonte helvetica não suporta emojis Unicode — eles aparecem como `Ø=Üe` e similares.

**Localizar e substituir** todas as ocorrências de texto com emoji dentro da função `exportPDF()`:

| Texto atual | Substituir por |
|---|---|
| `📅 ${eventDateDisplay}` | `Data: ${eventDateDisplay}` |
| `👥 ${state.people} pessoas` | `${state.people} pessoas` |
| `🏠 ${state.families} familias` | `${state.families} familias` |
| `👤 Por pessoa: R$ ${perPerson}` | `Por pessoa: R$ ${perPerson}` |
| `🏠 Por familia: R$ ${perFamily}` | `Por familia: R$ ${perFamily}` |
| Qualquer outro emoji em `doc.text(...)` | Remover o emoji, manter o texto |

> **Regra geral:** dentro de `exportPDF()`, nenhum `doc.text()` pode conter caracteres fora do range ASCII + latin1. Revisar toda a função.

---

### 5. Inicializar logo ao carregar a página

No bloco de inicialização no final do script (onde estão `initSlots()`, `loadState()`, `render()`), adicionar:

```js
// Aplica logo real assim que o base64 estiver disponível
document.getElementById('logoHeader').src = LOGO_PRINCIPAL_B64;
```

---

## Restrições obrigatórias

- **Não quebrar nenhuma funcionalidade existente**: slots, categorias, itens, peso/kg, obs, share text, configurações, listas salvas, exportação PDF
- **Edições cirúrgicas**: modificar apenas os trechos listados acima
- **Arquivo único**: o resultado deve continuar sendo um `index.html` self-contained — o base64 da imagem fica embutido no HTML
- **Não usar URLs externas** para a logo — apenas base64 inline
- **Não reescrever** o que já funciona corretamente

---

## Arquivos no repositório

| Arquivo | Descrição |
|---|---|
| `index.html` | App principal — único arquivo a modificar |
| `natitur.png` | Brand sheet com as 3 variações da logo |
| `ChatGPT Image 23 de mai. de 2026, 16_49_56.png` | Referência visual da brand sheet |
| `ChatGPT Image 23 de mai. de 2026, 16_52_19.png` | Referência visual adicional |
| `CODEX_PROMPT.md` | Este arquivo |

---

## Resultado esperado

1. Header do app mostra a **logo real da Natitur** (ícone + texto), integrada visualmente ao fundo escuro
2. PDF exportado mostra o **ícone circular** da Natitur no cabeçalho, sem distorções
3. PDF não tem mais nenhum caractere quebrado (Ø, Ü, ß, etc.)
4. Todos os valores de "Por família" e "Por pessoa" aparecem corretamente no PDF
5. App continua funcionando 100% — todas as features preservadas
