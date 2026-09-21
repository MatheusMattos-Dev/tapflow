# TapFlow

Página de vendas do método de produção e venda de placas NFC.

Arquivo único: `index.html`. Sem build, sem dependência. Abre com duplo clique e
sobe em qualquer host estático (Vercel, Netlify, Hostinger) arrastando a pasta.

```
tapflow/
  index.html          toda a página: HTML + CSS + JS
  assets/
    favicon.svg
    noise.png         textura de ruído, usada no escuro e no claro
    images/
      hero-grid.webp  grade de blueprint, fundo do topo
      numeros-bg.webp arte de placas — sem uso no momento
```

As fontes vêm do Google Fonts, não há woff2 local.

## De onde veio

- **Estrutura e arquitetura de classes**: `hyrox-page` — o mesmo fluxo de seções
  (hero → público → prova → oferta → números → método → especialista → cases →
  FAQ → CTA), os mesmos nomes de classe e a mesma mecânica de countdown, vagas e
  barra fixa de compra.
- **Pele**: `portfolio` — fundo escuro com textura de ruído, cards de vidro,
  linhas finas de contorno.
- **Tipografia**: `hyrox-page` — Big Shoulders Display nos títulos (caixa alta,
  entrelinha curta), IBM Plex Sans no corpo, IBM Plex Mono nos miúdos.
- **Paleta**: TapFlow, no lugar do verde do portfolio.

| Cor | Hex | Uso |
| --- | --- | --- |
| Preto profundo | `#05070B` | fundo principal |
| Azul-marinho | `#0B1630` | fundo secundário, cards, áreas internas |
| Azul elétrico | `#146CFF` | botões, destaques, elementos de ação |
| Ciano neon | `#00E5FF` | bordas, ícones NFC, efeitos de luz |
| Branco | `#F7F9FC` | textos principais |
| Cinza metálico | `#A9B2C3` | textos secundários |

Todas estão em `:root`, no topo da `<style>`. Mudar ali muda a página inteira.

Há um tom derivado que não está na paleta original: `--azul-escuro` `#0B4FC4`.
Ele existe porque o azul elétrico puro sobre fundo claro dá 4.31:1 e reprova no
mínimo de acessibilidade; essa versão dá 6.80:1. É usado só em texto pequeno
sobre o claro e no selo "mais completo".

## As imagens de fundo

Só uma está em uso: `hero-grid.webp`, no `.topo`.

`numeros-bg.webp` continua na pasta mas **não é referenciada por nada**. Ela
esteve no fundo de `#resultados` e foi removida. Para usá-la de novo em alguma
seção, é uma regra:

```css
.section--art {
  background-image: url("assets/images/numeros-bg.webp");
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
}
```

```html
<section class="section section--art" id="cases">
```

Repare que ela usa `cover`, não `repeat`. É uma **composição** — os desenhos
ficam nos cantos e o miolo é vazio de propósito, pra deixar o texto legível por
cima. Repetir duplicaria os cantos. A grade do topo é o caso oposto: textura,
com bordas que casam.

Ambas vieram de PNGs de quase 1 MB e foram convertidas para WebP, onde pesam
13 KB e 15 KB, com erro médio abaixo de 1 em 255 — imperceptível em arte assim,
de linhas finas sobre fundo chapado. Os PNGs originais foram apagados; se
precisar deles de novo, é só reexportar.

## O fundo do topo

`assets/images/hero-grid.webp` é a grade de blueprint atrás do hero. Ela
**repete** em vez de
esticar (`background-repeat: repeat`, tamanho fixo de 1672×941): as bordas da
imagem casam entre si, então as células ficam quadradas em qualquer largura,
inclusive num monitor ultrawide, onde `cover` deformaria.

`.topo` é um invólucro, e existe para que a fronteira da grade seja **uma linha
só** no arquivo: para a textura cobrir mais seções, mova o `</div>` do `.topo`
para depois da seção que quiser incluir. Se cada seção declarasse a imagem por
conta própria, o ladrilho reiniciaria em cada fronteira e apareceria uma emenda
horizontal — é isso que o invólucro evita.

Por cima da grade há um esmaecimento curto no fim do hero, só para a imagem não
parecer cortada a faca na borda do bloco seguinte. As duas camadas moram na
mesma regra `.topo`, sem pseudo-elemento — o `::before` do hero já é dos halos
azul e ciano, que ficam acima da grade.

Para trocar a imagem: gere o WebP e mantenha o nome, ou ajuste o
`background-size` para as dimensões reais do arquivo novo. Se a imagem nova não
for tileável, troque `repeat` por `cover` e tire o tamanho fixo.

## Claro e escuro

Três seções são claras, para quebrar o ritmo: **oferta**, **método** e **FAQ**.
O resto é escuro.

Uma seção clara não repinta componente por componente — ela só redefine, em
`.section--light`, os mesmos tokens que todo componente já lê (`--ink`,
`--muted`, `--line`, `--surface-glass`…), e tudo dentro inverte sozinho. Para
tornar outra seção clara, basta trocar a classe:

```html
<section class="section section--light" id="cases">
```

As variantes disponíveis são `section--light` (branco), `section--navy`
(marinho) e `section--deep` (quase preto).

Dentro do claro o `--ciano` é remapeado para `--azul-escuro`: ciano neon sobre
branco fica em 1.4:1 e some.

## O ritmo de fundos

A página alterna quatro tratamentos, nesta ordem:

| Trecho | Fundo |
| --- | --- |
| hero | grade de blueprint (`.topo`) |
| esteira + faixa de público + depoimentos | `.prova`, quase preto, com borda |
| oferta | `section--light`, claro |
| números, CTA, cases, CTA final | fundo liso da página |
| método, FAQ | `section--light`, claro |

As três dividem o elemento `.prova` pelo mesmo motivo do `.topo`: fundo
contínuo. Se cada uma declarasse o seu, a borda de uma encostaria na da outra e
apareceria risco duplo no meio do bloco.

A esteira de nichos **não é uma `<section>`** — é um `<div class="container
splitbar">` solto, primeiro filho do `.prova`. Ela nasceu dentro do hero e foi
movida; por isso perdeu a `margin-top` e a `border-top` (que encostariam na
borda do bloco) e ganhou `padding-top` no lugar.

## A faixa de público

Os sete nichos cabem numa linha só a partir de 1024px. Abaixo disso a faixa
quebra — e aí ela fica **centralizada** de propósito (`@media (max-width:
1179px)`), porque o que parecia defeito não era a quebra em si, era um item
pendurado sozinho à esquerda na última linha.

Se você acrescentar um oitavo nicho, ele vai empurrar a quebra para o desktop
de novo. A linha inteira precisa caber em ~1050px; hoje sobram 86px de folga.

## Contraste

A página inteira passa no WCAG AA (4.5:1 para texto normal, 3:1 para texto
grande), medido nos dois tamanhos de tela. Os pontos mais apertados:

| Combinação | Razão |
| --- | --- |
| `#fff` sobre azul elétrico (botões) | 4.55:1 |
| `--muted-2` `#7E8A9C` sobre o preto | 5.76:1 |
| `--azul-escuro` sobre o branco | 6.80:1 |
| `--muted` `#48505F` sobre o branco | 7.69:1 |

Três armadilhas que já custaram caro aqui, caso você mexa nas cores:

- O branco da paleta, `#F7F9FC`, **reprova** sobre o azul elétrico (4.31:1). Os
  botões usam `#fff` puro de propósito.
- O botão do Pacote tinha um gradiente terminando em ciano claro: branco sobre
  aquela ponta caía para 2.76:1. Hoje o gradiente fica na faixa azul.
- No menu do celular, `.mobile-drawer ul a` precisa do `:not(.btn)`. Sem ele a
  regra vence `.btn--primary` por especificidade e repinta o texto do botão.

## Antes de publicar

**1. Links de checkout.** Os 7 botões de compra estão com `href="#"`. Cada um
carrega o atributo `data-checkout-placeholder`:

```
grep -n "data-checkout-placeholder" index.html
```

Troque o `href` de cada um pelo link real. Enquanto não trocar, o clique rola
até a seção de oferta em vez de recarregar a página, e o console avisa quantos
faltam.

**2. Prova social.** Depoimentos, cases e o bloco do especialista são fictícios
— existem só para segurar o layout. Estão marcados com `PLACEHOLDER` em
comentário:

```
grep -n "PLACEHOLDER" index.html
```

Publicar depoimento inventado como se fosse real é propaganda enganosa. Troque
por conversas e resultados de verdade, com autorização de nome e imagem, ou
apague as seções.

**3. Pixel e UTM.** Há um ponto marcado no topo do arquivo para colar Meta
Pixel, Google Ads ou UTMify. O rastreio de `InitiateCheckout` já está escrito no
JS do rodapé, comentado — basta descomentar depois de colar o pixel.

**4. Prazo do lote.** A data aparece em um lugar só:

```html
<div class="offer-bundle__urgency-bar" data-bundle-urgency data-deadline="2026-09-30T23:59:59-03:00">
```

O contador regressivo e a barra fixa de compra leem dali. Passado o prazo, a
faixa de urgência some sozinha e a barra fixa passa a oferecer o Método.
O texto "até 30/09" aparece em três lugares e é escrito à mão — ajuste junto.

**5. Vagas.** `data-total` e `data-restantes` na mesma faixa. O contador é **por
visitante**, guardado no navegador de quem clicou: sem backend, o próximo
visitante começa de novo no total declarado. Nunca chega a zero.

**6. Preços.** São placeholders: R$ 19 (Guia), R$ 47 (Método), R$ 67 (Pacote,
de R$ 127). Cada botão repete o valor em `data-price` para o rastreio — trocar
o preço exige trocar os dois.

**7. Imagens.** Os mockups dos produtos e a foto do especialista são blocos
`.media-holder` com ícone e legenda. Substitua por `<img>` apontando para
`assets/images/`.

## Comportamento

- **Revelação no scroll** — desligada no celular e em "reduzir movimento".
- **Contador regressivo** — some sozinho quando o prazo passa.
- **Barra fixa de compra** — só no celular, aparece depois da esteira de nichos
  e some dentro da seção de oferta, onde os botões de verdade já estão na tela.
- **Carrossel de prints** — bolinhas de navegação que se recalculam no resize e
  somem quando tudo cabe na tela.
- **Lista longa do Método** — nasce cortada no celular, com botão "ver tudo".
