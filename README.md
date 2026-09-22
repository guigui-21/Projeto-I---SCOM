# Omnitrix Codex

Front-end da disciplina de Sistemas Computacionais (SCOM 2026) — Trabalho
Individual I, com base no guia `Omnitrix Codex - Guia de Projeto e Passo a
Passo (SCOM 2026).pdf`.

## Como executar

O projeto não depende de servidor, build ou internet: é HTML/CSS/JS puro.

1. Extraia o `.zip`.
2. Dê duplo clique em `index.html` (ou abra pelo navegador com `Ctrl+O`).
3. Pronto — funciona 100% offline, em Chrome, Firefox e Edge.

## Estrutura de arquivos

```
omnitrix-codex/
├── index.html          → HTML5 semântico (header, nav, main, section, article, aside, footer)
├── css/
│   └── styles.css      → CSS externo único, com :root de variáveis e media queries
├── js/
│   ├── aliensData.js   → "banco de dados" dos 4 Omnitrix e seus aliens (camada de dados)
│   ├── app.js          → lógica de interação, disco giratório, pop-up e acessibilidade
│   └── sound.js        → controla os 4 efeitos sonoros (com bipe sintetizado de reserva)
├── assets/
│   ├── gwen.svg, max.svg, kevin.svg, azmuth.svg   → emblemas originais dos aliados
│   ├── audio/           → os 4 efeitos sonoros do Omnitrix (mp3 + m4a)
│   ├── omnitrix/        → imagens reais dos 4 relógios + símbolo central (enviadas por você)
│   └── aliens/          → pasta reservada caso queira substituir os glifos por artes próprias
└── README.md
```

## O que já está pronto

- **Estrutura semântica completa**: `header`, `nav`, `main`, `section`, `article`,
  `aside`, `figure`/`figcaption`, `footer`; um único `h1`; hierarquia `h2 > h3 > h4`
  consistente.
- **4 eras do Omnitrix** (Clássico 2005, Força Alienígena 2008, Supremacia
  Alienígena 2010, Omniverse 2013), cada uma com paleta própria via variáveis CSS
  e a imagem real do relógio daquela era ao lado do nome/descrição.
- **60 aliens no total** (19 Clássico, 13 Força Alienígena, 11 Supremacia
  Alienígena, 17 Omniverse), com espécie, planeta, habilidades e feito marcante
  — conteúdo fornecido por você em `js/aliensData.js`.
- **Símbolo do Omnitrix real** no centro do disco giratório (a mesma imagem
  para as 4 eras, já que é o símbolo genérico do dispositivo).
- **Disco giratório "visto de cima"**: o anel de marcadores gira (CSS
  `transform: rotate()`) conforme você passeia pelos aliens com as setas ‹ ›
  (ou as setas do teclado). Cada giro toca `som_troca_omnitrix`.
- **4 efeitos sonoros reais** (`assets/audio/`), tocados nos momentos certos:
  | Ação do usuário | Som tocado |
  |---|---|
  | Abrir/trocar de era (aba do Omnitrix) | `som_abertura_omnitrix` |
  | Girar o disco para o alien anterior/próximo | `som_troca_omnitrix` |
  | Clicar num alien (no disco ou na lista) para vê-lo | `som_escolha_omnitrix` |
  | Fechar o pop-up de detalhes | `som_descarregou_omnitrix` |

  Cada som tem um `<audio>` com duas fontes (`.mp3` e `.m4a`), para que o
  navegador escolha o formato que suportar. Se, por algum motivo, nenhum dos
  dois tocar, um bipe sintetizado (Web Audio API) entra como reserva — a
  interação nunca fica muda. Há também um botão de mudo, e nenhum som toca
  sozinho no carregamento da página (evita bloqueio de autoplay do navegador).
- **Pop-up de detalhes (`<dialog>`)**: nome, espécie, planeta, feito marcante
  e habilidades do alien escolhido, com foco preso dentro do pop-up, fechamento
  por Esc/clique fora/botão ✕, e foco devolvido a quem abriu o pop-up ao fechar.
- **Acessibilidade**: navegação por teclado (setas nas abas de era e no disco),
  foco visível (`:focus-visible`), skip link, `aria-live`, padrão ARIA
  `tablist`/`tab`/`tabpanel`, respeito a `prefers-reduced-motion`.
- **Responsivo mobile-first** com 3 breakpoints (mobile / tablet / desktop),
  Flexbox + CSS Grid.
- **Boas práticas de imagem**: `alt` descritivo, `loading="lazy"` e `width`/`height`
  explícitos nos emblemas dos personagens.

## Performance — otimizações aplicadas

Uma primeira auditoria Lighthouse (mobile) apontou Performance = 49, com
Acessibilidade, Best Practices e SEO em 100. A causa raiz, segundo o próprio
relatório, era o peso e o formato das imagens (não a estrutura HTML/CSS/JS).
Sem alterar nenhuma seção, layout ou funcionalidade do site, foram feitas
apenas otimizações técnicas de arquivo:

1. **Redimensionamento das imagens para o tamanho real de exibição.** Várias
   fotos de personagens/aliens/vilões estavam com até 1843 px de altura sendo
   exibidas em apenas 160 px — todas foram redimensionadas para ~360 px de
   altura (nítido em telas retina, sem excesso de dado).
2. **Conversão para JPEG das imagens com fundo "chapado".** Fotos com muitas
   cores/gradientes (ex.: `encanadores.png`, `skurd.png`, a arte da introdução)
   comprimem muito mal em PNG. Elas foram recompostas sobre a cor de fundo do
   card/pop-up (`#14211b`) ou da introdução (`#0a0f0d`) e salvas como `.jpg`
   — visualmente idênticas (o fundo da imagem já "some" no fundo do card),
   mas até 90% mais leves. Ilustrações com poucas cores (os glifos e os 60
   ícones `*CL.png` dos aliens) permaneceram em PNG, que comprime melhor esse
   tipo de imagem.
3. **`width`/`height` explícitos em todas as tags `<img>`.** O relatório
   apontava isso como a causa direta do CLS de 0.359 (o navegador não sabia
   reservar espaço antes da imagem carregar). Agora todo `<img>` (estático no
   HTML e os do pop-up, gerados via JS) carrega as dimensões reais do arquivo.
4. **Prioridade de carregamento da imagem da introdução (LCP).** Ela estava
   com `loading="lazy"` — o que atrasa justamente o maior elemento visível da
   primeira dobra. Trocado por `fetchpriority="high"`.
5. **Áudios com `preload="none"`.** Os 4 efeitos sonoros só são baixados
   quando tocam pela primeira vez, em vez de competir com o carregamento
   inicial da página.

Resultado esperado: peso total de imagens caiu de ~7,3 MB para ~2,3 MB, o que
deve levar o LCP para bem abaixo de 2,5 s e o CLS para próximo de 0 — dentro
das metas do Trabalho I (Performance ≥ 85). **Rode uma nova auditoria
Lighthouse (mobile) depois de publicar esta versão e atualize os números na
seção de resultados do seu relatório.**

O que foi **propositalmente não alterado**: minificação de CSS/JS. O
relatório aponta ~12 KiB de economia possível, mas isso exigiria remover os
comentários do código-fonte — e "comentários úteis" é um item avaliado no
Critério G (Qualidade do código). Como o ganho de Performance já vem quase
todo da otimização de imagens, manter o código comentado e legível pareceu a
troca certa; se quiser, você pode justificar essa decisão no relatório.

## Antes de entregar — pontos de atenção

1. **Conteúdo dos aliens** (`js/aliensData.js`): os textos (espécie, planeta,
   habilidades, feito marcante) foram escritos a partir de conhecimento geral
   da franquia e servem de ponto de partida. Confira e ajuste com a
   [Ben 10 Fandom Wiki (PT-BR)](https://ben10.fandom.com/pt-br/wiki/Categoria:Esp%C3%A9cies_do_Omnitrix)
   antes de entregar, como o próprio guia recomenda.
2. **Imagens**: os emblemas em `assets/` são ilustrações originais (SVG gerado),
   não arte oficial da série — isso evita qualquer risco de direitos autorais
   na entrega. Se quiser usar fan art ou capturas de tela oficiais, verifique
   a política de uso de imagens da disciplina antes de substituir.
3. **Áudios**: os 4 sons em `assets/audio/` são os arquivos que você enviou.
   Como são efeitos originais da série (não gerados por você), use-os apenas
   no seu projeto acadêmico/local — evite publicar o site com esses arquivos
   em um domínio público, para não ter problemas de direitos autorais.
4. **Suporte a `<dialog>` e a `.m4a`**: confirme o pop-up e os sons nos três
   navegadores exigidos (Chrome, Firefox, Edge) — ambos são bem suportados nas
   versões atuais, mas vale registrar isso na tabela de compatibilidade do
   relatório.
5. **Validação W3C**: rode `index.html` e `css/styles.css` nos validadores
   oficiais (validator.w3.org e jigsaw.w3.org/css-validator) e tire print do
   resultado "zero erros" para o relatório.
6. **Lighthouse**: abra o DevTools (F12) → aba Lighthouse → modo Mobile →
   rode Acessibilidade, Performance, Best Practices e SEO. Anexe o print.
7. **Compatibilidade**: teste manualmente em Chrome, Firefox e Edge e preencha
   a tabela do relatório.


## Declaração de uso de IA generativa (preencher no relatório)

| Item | Descrição |
|---|---|
| Ferramenta | Claude (Anthropic) e Gemini (Google)|
| Finalidade | Geração da estrutura de HTML/CSS/JS do front-end e integração dos efeitos sonoros/pop-up a partir do guia de projeto do estudante |
| Trecho produzido | Estrutura completa de `index.html`, `css/styles.css`, `js/app.js`, `js/aliensData.js`, `js/sound.js` |
| Modificações realizadas | *(preencher: o que você ajustou — dados dos aliens, cores, textos, sons etc.)* |
| Avaliação crítica | *(preencher: o que funcionou bem, o que você revisou/corrigiu, e por quê)* |


## Sobre o Trabalho Individual II (backend)

Este pacote cobre **apenas o front-end** (Trabalho I). Quando for evoluir
para o Trabalho II, a sugestão é:

- Trocar `js/aliensData.js` por chamadas `fetch()` a uma API própria
  (CRUD de aliens em PostgreSQL/MySQL).
- Adicionar autenticação (ex.: um perfil "curador" que pode cadastrar novos
  aliens/eras, e um perfil "visitante" que só visualiza).
- Manter a mesma camada de apresentação (`app.js`) — ela já está desacoplada
  dos dados, então trocar a origem dos dados exige poucas mudanças.