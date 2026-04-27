# Relatório de Melhorias — CTH.JP

> Análise gerada em 2026-04-27. Site hospedado em GitHub Pages, domínio custom `cth.jp`,
> 28 artigos HTML + 1 homepage + 89 imagens (~25MB).

---

## TL;DR

O site está muito bem feito. HTML5 semântico, dark/light mode, responsivo, SEO básico no lugar, Schema.org nos artigos — não é um site descuidado.
Os problemas são basicamente em **performance de imagens** (maior dor), alguns **meta tags sociais faltando** e uns ajustes de acessibilidade.

Pontuação estimada por categoria:

| Categoria          | Score |
|--------------------|-------|
| Estrutura HTML5    | 9/10  |
| SEO on-page        | 8/10  |
| Acessibilidade     | 7/10  |
| Performance        | 6/10  |
| Segurança          | 9/10  |
| Design/UX          | 9/10  |
| Responsividade     | 9/10  |
| **GERAL**          | **8/10** |

---

## 1. Performance (maior problema)

### 1.1 Imagens sem lazy loading

Todas as 89 imagens do site carregam na hora que a página abre, mesmo as que estão lá embaixo que o usuário talvez nunca veja.

**Onde está:** toda `<img>` em `index.html` e nos artigos.

**Como resolver:** adicionar `loading="lazy"` em todas as imagens que não estejam no topo da página (above the fold). O avatar do hero pode ficar sem, as demais todas ganham.

```html
<!-- antes -->
<img src="../imagens/rclone-config-b2.png" alt="...">

<!-- depois -->
<img src="../imagens/rclone-config-b2.png" alt="..." loading="lazy">
```

Também vale adicionar `decoding="async"` e declarar `width` e `height` para evitar layout shift (CLS):

```html
<img src="..." alt="..." loading="lazy" decoding="async" width="800" height="450">
```

**Impacto:** alto. Pode economizar vários MB de download na primeira visita.
ok
---

### 1.2 Imagem gigante de 2,2MB

O arquivo `imagens/cedula-dinheiro-hong-kong.png` tem **2,2MB**. Nenhuma imagem num blog precisa ser tão grande assim.

**Como resolver:** redimensionar para no máximo 1200px de largura e salvar como WebP ou JPEG com qualidade ~80%. Ferramentas: `cwebp`, `squoosh.app`, `imagemagick`.

```bash
# exemplo com imagemagick
convert cedula-dinheiro-hong-kong.png -resize 1200x -quality 80 cedula-dinheiro-hong-kong.webp
```

**Impacto:** alto. Sozinha essa imagem pesa mais que o HTML+CSS inteiro do site.
ok
---

### 1.3 Sem WebP

O site usa apenas PNG e JPEG. WebP entrega imagens ~30-40% menores com mesma qualidade visual.

**Como resolver:** converter as imagens para WebP e usar `<picture>` com fallback:

```html
<picture>
  <source srcset="../imagens/aws-ccp-certificado.webp" type="image/webp">
  <img src="../imagens/aws-ccp-certificado.png" alt="Certificado AWS CCP" loading="lazy">
</picture>
```

Não precisa fazer de uma vez — dá pra ir convertendo conforme for editando os artigos.
ok
---

### 1.4 CSS não minificado (44KB)

O `style.css` com 1002 linhas não está minificado. Minificado ficaria em torno de 20-25KB.

**Como resolver:** passar pelo [cssnano](https://cssnano.co/) ou pelo [cleancss](https://github.com/clean-css/clean-css) numa pipeline simples. Não precisa de webpack nem nada — um script npm basta:

```bash
npx cleancss -o style.min.css style.css
```

Aí no HTML troca o link para `style.min.css`. Mantém o `style.css` para edição.

**Impacto:** médio. ~20KB de economia, mas como o CSS é cacheado depois da primeira visita, o ganho real é só no primeiro load.

---

### 1.5 Iframes do YouTube sem lazy loading

Os artigos que embarcam vídeos do YouTube (como `como-chip-feito.html`) não usam lazy loading no iframe.

```html
<!-- antes -->
<iframe src="https://www.youtube.com/embed/k4mM8X2LCI0" ...>

<!-- depois -->
<iframe src="https://www.youtube.com/embed/k4mM8X2LCI0" loading="lazy" ...>
```

Melhor ainda: usar a técnica de façade (lite-youtube ou `youtube-nocookie.com`) para não carregar o iframe até o usuário clicar. Mas pelo menos o `loading="lazy"` já ajuda bastante.
ok
---

### 1.6 Fontes Google externas (3 fontes)

O `@import` no topo do CSS chama três fontes do Google Fonts: Space Mono, M PLUS Rounded 1c e JetBrains Mono.

O `display=swap` já está configurado (bom!), mas ainda depende de DNS lookup + conexão TLS com o Google.

**Opção 1 (fácil):** substituir o `@import` por um `<link rel="preconnect">` + `<link rel="stylesheet">` no HTML, com `rel="preload"` para adiantar:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Space+Mono...&display=swap">
```

**Opção 2 (melhor, mas mais trabalho):** hospedar as fontes localmente com [google-webfonts-helper](https://gwfh.mranftl.com/fonts). Elimina o request externo e melhora o score do Lighthouse.
ok
---

## 2. SEO

### 2.1 Falta og:image

O `index.html` tem Open Graph (og:title, og:description, og:url, og:site_name) mas não tem `og:image`. Quando alguém compartilha o site no Discord, Telegram, WhatsApp, LinkedIn — não aparece nenhuma imagem no preview.

**Como resolver:** criar uma imagem de capa (1200×630px) e adicionar:

```html
<meta property="og:image" content="https://cth.jp/imagens/og-cover.png">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:image:alt" content="CTH.JP — Blog do Celso Hamasaki">
```

Os artigos também deveriam ter og:image próprios, preferencialmente uma imagem relacionada ao conteúdo.

**Impacto:** alto para distribuição social. É a diferença entre um link pelado e um card bonito no Discord.

---

### 2.2 Falta Twitter Card

Nenhuma página tem as meta tags do Twitter/X. O resultado é que links compartilhados lá ficam sem preview.

```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:creator" content="@seu_twitter_aqui">
<meta name="twitter:title" content="Celso Takeshi Hamasaki | Dev Java & Cloud">
<meta name="twitter:description" content="Desenvolvedor Backend Java...">
<meta name="twitter:image" content="https://cth.jp/imagens/og-cover.png">
```

---

### 2.3 Schema.org Person poderia ser mais rico

O Schema.org do `index.html` está bom mas pode crescer:

```json
{
  "@context": "https://schema.org",
  "@type": "Person",
  "name": "Celso Takeshi Hamasaki",
  "url": "https://cth.jp",
  "image": "https://cth.jp/imagens/HFxhilNbIAAJyib.jpeg",
  "jobTitle": "Desenvolvedor Backend Java",
  "knowsAbout": ["Java", "Linux", "AWS", "Cloud Computing", "Segurança Digital"],
  "sameAs": [
    "https://www.linkedin.com/in/celso-hamasaki",
    "https://github.com/seu-usuario"
  ]
}
```

O campo `image` e `knowsAbout` ajudam o Google a montar o Knowledge Panel do autor.

---

### 2.4 Nomes de arquivo com espaços (URL encoding)

Vários arquivos de imagem têm espaços no nome, o que vira `%20` na URL:

```
../imagens/Pasted%20image%2020251130222709.png
```

Funcionalmente tudo bem, mas é feio e pode causar problemas em alguns contextos. O ideal é renomear para kebab-case na próxima edição dos artigos.
ok
---

### 2.5 Meta description um toque longa

A meta description do index está com 170 caracteres. O Google geralmente corta em ~155-160. Não é crítico (o Google pode usar o que quiser de qualquer forma), mas manter em até 160 garante que nada seja cortado.

---

## 3. Acessibilidade

### 3.1 Sem `:focus-visible`

O CSS define hover states mas não define estilos explícitos de foco para teclado. Quem navega só com Tab fica sem indicação visual clara de onde está.

```css
/* adicionar em style.css */
:focus-visible {
  outline: 2px solid var(--pink);
  outline-offset: 3px;
  border-radius: var(--r);
}
```

O `:focus-visible` só aparece quando o usuário está de fato navegando por teclado (não aparece ao clicar com mouse), então não estraga a estética.
ok
---

### 3.2 Sem skip link

Usuários de leitor de tela ou teclado precisam passar por toda a navegação antes de chegar no conteúdo. Um skip link resolve isso:

```html
<!-- logo depois do <body> -->
<a href="#main" class="skip-link">Pular para o conteúdo</a>
```

```css
.skip-link {
  position: absolute;
  top: -100%;
  left: 0;
  background: var(--pink);
  color: white;
  padding: 8px 16px;
  z-index: 9999;
}
.skip-link:focus { top: 0; }
```

Invisível para quem não usa teclado, útil para quem usa.
ok
---

### 3.3 Imagens sem dimensões declaradas (CLS)

Imagens sem `width` e `height` no HTML causam layout shift enquanto carregam — o conteúdo "pula" quando a imagem aparece. Isso prejudica o Core Web Vitals (CLS score).

```html
<!-- adicionar width e height (ou ratio via CSS) -->
<img src="..." alt="..." loading="lazy" width="800" height="450">
```

Não precisa ser o tamanho exato — o que importa é manter o aspect ratio correto para o browser reservar o espaço.
ok
---

### 3.4 Hierarquia de headings rasa

Os artigos usam H1 e H2 mas nunca H3+. Em artigos mais longos e com subseções, isso deixa o outline do documento plano. Não é um erro, mas H3 para subseções dentro de H2 seria mais correto semanticamente.
ok
---

## 4. Segurança

O site está bem. É estático, sem formulários, sem processamento server-side. Os riscos são mínimos. Só dois pontos menores:

### 4.1 rel="noreferrer" faltando

Os links externos têm `rel="noopener"` (protege contra tabnabbing — ótimo!), mas poderiam ter também `rel="noreferrer"`:

```html
<!-- antes -->
<a href="https://linkedin.com/..." target="_blank" rel="noopener">

<!-- depois -->
<a href="https://linkedin.com/..." target="_blank" rel="noopener noreferrer">
```

`noreferrer` evita que o site destino saiba de onde o usuário veio. Privacidade extra, sem custo.
ok
---

### 4.2 CSP via GitHub Pages / Cloudflare

Como o site é estático no GitHub Pages, não dá para adicionar headers HTTP diretamente. Mas se passar por Cloudflare (que já pode estar configurado pro domínio `cth.jp`), dá pra adicionar uma Content Security Policy simples via Transform Rule:

```
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src https://fonts.gstatic.com; img-src 'self' data: https:; frame-src https://www.youtube.com;
```

Não é urgente num site pessoal estático, mas ajuda no score de segurança de ferramentas como SecurityHeaders.com.

---

## 5. Outros (pequenos)

### 5.1 Vídeo MP4 na pasta imagens

Tem um arquivo `.mp4` dentro de `/imagens/` (`Ser_que_o_5G_pode_mudar_a_humanidade...mp4`). Se não está sendo usado em nenhum artigo, ocupa espaço desnecessário no repositório. Git não é bom para arquivos binários grandes — considerar remover ou mover para um host de vídeo externo.
ok
---

### 5.2 Scrollbar customizado só funciona no Chrome/Edge

```css
::-webkit-scrollbar { ... }
```

Firefox não respeita `::-webkit-scrollbar`. Tem uma propriedade padrão mais nova:

```css
/* padrão (Firefox + futuros browsers) */
* {
  scrollbar-width: thin;
  scrollbar-color: var(--border) var(--bg);
}
```

As duas podem coexistir — webkit para Chrome, a propriedade padrão para Firefox.
ok

---

### 5.3 Syntax highlighting nos code blocks

Os blocos de código têm boa estética (fonte JetBrains Mono, cor via CSS) mas o conteúdo é texto puro — sem coloração por token. Em artigos técnicos isso ajuda bastante na leitura.

Opção leve: [highlight.js](https://highlightjs.org/) (~50KB, ou só a linguagem necessária ~10KB). Dá pra incluir só no template dos artigos.

Alternativa sem JS: [Prism.js](https://prismjs.com/) que também pode ser configurado pra carregar só as linguagens usadas.

Sendo um blog retro, dá pra escolher um tema que combine com a paleta — o Prism tem temas customizáveis.
ok
---

### 5.4 PDF na pasta imagens

Tem PDFs (`celso-takeshi-hamasaki-*.pdf`) dentro de `/imagens/`. Funcionalmente ok, mas semanticamente faz mais sentido uma pasta `/docs/` ou `/assets/`.

---

## Resumo de Prioridades

### Fazer logo (alto impacto, baixo esforço)
1. `loading="lazy"` em todas as `<img>` dos artigos
2. `loading="lazy"` nos `<iframe>` do YouTube
3. Adicionar `og:image` no index.html e nos artigos
4. Adicionar Twitter Card tags
5. Comprimir a imagem de 2,2MB

### Fazer quando tiver tempo (médio impacto)
6. `scrollbar-width: thin` para Firefox
7. `:focus-visible` para acessibilidade de teclado
8. `rel="noreferrer"` nos links externos
9. `og:image` customizado por artigo
10. Converter imagens grandes para WebP

### Backlog (baixo impacto ou muito esforço)
11. Minificar CSS
12. Self-host das fontes Google
13. Syntax highlighting nos code blocks
14. Skip link de acessibilidade
15. CSP via Cloudflare
16. Renomear arquivos com espaço no nome
