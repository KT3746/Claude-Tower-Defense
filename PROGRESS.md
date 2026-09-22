# Muralha — status do projeto

Tower defense medieval, arquivo único autocontido (`index.html`: HTML+CSS+JS
inline, sem build, sem dependências além da folha de fontes do Google Fonts).

Duas formas de jogar, ambas sempre na versão mais recente:
- **Claude Artifact** (exige login na conta Claude):
  https://claude.ai/code/artifact/b0742332-e8be-417e-a8fa-2e358a728366
- **GitHub Pages** (público, sem login — pedido do usuário pra abrir no Safari
  do celular sem precisar de conta): https://kt3746.github.io/Claude-Tower-Defense/
  Publicado por `.github/workflows/pages.yml`, que roda a cada push em
  `claude/progress-md-context-kj12ok` (não mexe em `main`). Ver a seção
  "GitHub Pages" mais abaixo antes de mexer nesse workflow.

Todo o histórico abaixo está mesclado na `main`. Se você (uma sessão nova) foi
chamado para continuar este projeto, comece lendo `index.html` inteiro — são
~2900 linhas, um único IIFE comentado por seções — e depois volte aqui.

## O que já existe (v1 + v2, incrementos 1 a 6)

- **v1**: jogo completo e jogável — grade 18×10, estrada única sinuosa,
  3 torres (Arqueiro/Canhão/Mago), 4 inimigos (Trasgo/Orc/Cavaleiro
  Renegado/Chefe de Guerra), 15 ondas, economia de ouro, upgrade/venda de
  torre, pausa, velocidade ×2, som via WebAudio (sem arquivos de áudio),
  recorde salvo em `localStorage`.
- **Incremento 1**: framework de animação (torres e inimigos animados via
  parâmetro de fase), pool fixo de partículas (faíscas, explosões, números de
  dano, brasas/névoa ambiente — nunca aloca, descarta silenciosamente quando
  cheio), tochas com brilho piscante (sprites de brilho pré-renderizados +
  `globalCompositeOperation='lighter'`, nunca `shadowBlur`), sombra de nuvens
  à deriva sobre o terreno.
- **Incremento 2 + 3b**: os 4 inimigos viraram personagens humanoides de
  verdade (cabeça/tronco/braços/pernas articulados com joelho/cotovelo,
  sombreamento em gradiente + contorno, armas/adereços distintos), pintados
  uma vez em folhas de sprite (6 quadros do ciclo de passada) na
  inicialização e só `drawImage`'d em tempo real — mais barato que desenhar
  ao vivo.
- **Incremento 3**: paisagem reescrita em `paintBackground()` — grama
  orgânica (manchas + gradiente direcional + vinheta, não mais xadrez),
  árvores/pedras/tufos/lagoa espalhados fora da estrada, estrada com trilha
  desgastada/cascalho/rachaduras. Tudo pré-renderizado uma vez, zero custo
  por frame.
- **Incremento 3c**: 3 bugs corrigidos (espada do cavaleiro que tinha
  sumido numa reescrita, número de dano na altura errada, upgrade/venda
  funcionando depois do jogo já ter acabado).
- **Incremento 4a**: **Corvo de Guerra** (inimigo voador) + **Balista**
  (torre nova anti-aérea), implementados juntos porque um sem o outro não
  dá pra testar de verdade.
  - `ENEMY_TYPES.raven` tem `flying:true` e `flyHeight` (deslocamento
    vertical); `drawEnemy` lê `def.flying` pra levantar o sprite e desenhar
    uma sombra separada na posição real (no chão), em vez de assar a sombra
    no sprite como os inimigos terrestres fazem via `groundShadow()`.
  - `paintRavenFrame` é um painter novo (asas batem com `Math.sin(phase)`
    em vez de ciclo de passada com perna/joelho) registrado em
    `ENEMY_PAINTERS.raven` — usa a mesma folha de sprites de 6 quadros do
    Incremento 2, só que sem `groundShadow()` embutido (pelo motivo acima).
  - Corvo liberado a partir da onda 4 em `generateWave` (`if(n>=4)
    available.push("raven")`), misturado aleatoriamente com os outros
    tipos como orc/cavaleiro já eram.
  - `TOWER_TYPES.ballista` tem `canHitFlying:true`. **Decisão de design
    tomada nesta sessão** (não estava no combinado anterior): a Balista
    também atinge inimigos terrestres — não é exclusiva pra voadores. A
    ideia foi evitar uma torre "morta" em ondas sem corvo; ela compensa
    isso custando mais (110) e tendo o maior alcance do jogo (150/175).
    Ainda não foi validado com o usuário — se ele preferir Balista
    exclusiva pra voadores, é só tirar o fallback terrestre de
    `findTarget`/`applyHit`.
  - `findTarget` e `applyHit` ganharam a checagem `eDef.flying &&
    !def.canHitFlying` pra pular voadores quando a torre/projétil não tem
    a capacidade — inclui o caso do canhão (splash) acertar de raspão um
    corvo que caiu dentro do raio da explosão.
  - Tecla `4` arma a Balista (`renderTowerCards` já gera o `kbd` certo
    sozinho, por ordem de inserção em `TOWER_TYPES`).
  - Verificado: `node --check` limpo, Playwright rodou 6 ondas em
    velocidade ×2 sem nenhum erro de console (fora o
    `ERR_CONNECTION_RESET` esperado da fonte do Google Fonts, que não
    carrega no sandbox de teste por falta de rede — não é bug do jogo).
    Confirmado visualmente por screenshot: Balista com zoom (base +
    dois braços curvos + virote) e Corvo de Guerra voando sobre a estrada
    com a sombra separada do corpo.

- **Incremento 4b — celular**: o tabuleiro deixou de ser um bloco de tamanho
  fixo e passou a ser encaixado na viewport por um `layout()` que roda na
  carga e a cada mudança de viewport (girar o aparelho, barra do navegador
  aparecer/sumir, redimensionar a janela).
  - **Retrato**: o mundo gira 90° (`ROT`). O `render()` aplica
    `setTransform` + `translate(viewW,0)` + `rotate(90°)` uma vez por quadro,
    então todo o resto do desenho continua no mesmo espaço de 900×500 de
    sempre — nada de coordenadas duplicadas. Tudo que tem "pra cima"
    (personagens, torres, árvores, castelo, portão, chama das tochas,
    números de dano) contra-gira com `upright()`; o que é chão (estrada,
    grama, sombras, círculos de alcance) gira junto. Num iPhone típico o
    tabuleiro salta de ~362×201 para ~331×596 — quase 3× a área, e os
    quadradinhos passam de ~21px para ~34px, que é a diferença entre
    conseguir e não conseguir acertar o toque.
  - A rotação é escolhida sozinha: `layout()` calcula a escala nas duas
    orientações e fica com a maior (com margem de 10% pra não ficar virando
    à toa). Tablet em retrato, por exemplo, continua sem girar.
  - `v2w()` (ponto da vista → mundo) e `sv()` (vetor de tela → mundo) mantêm
    entrada, brasas, névoa, gravidade das faíscas e o recuo da boca de fogo
    coerentes com a tela em qualquer orientação.
  - O fundo assado é **reassado** quando a orientação muda (pras árvores e o
    castelo ficarem em pé). Pra isso o cenário virou determinístico: as
    funções de assar usam um RNG semeado (`srand`/`rnd`) em vez de
    `Math.random`, então o mapa é sempre o mesmo — antes ele sorteava
    decoração nova a cada carga.
  - O buffer do canvas agora acompanha o tamanho exibido (`cssW*dpr`) em vez
    de 900×500×dpr fixos: no celular são ~2,4× menos pixels por quadro.
  - Cromo compacto por classe no `<body>` (`is-compact`/`is-stack`/`is-side`,
    postas pelo `layout()` — as media queries saíram porque JS e CSS
    precisam concordar): topo fino, legenda escondida, cartas de torre em
    linha (retrato) ou 2×2 (paisagem), botões de ação de 42px, rótulos
    curtos (`refreshActionLabels()`), e as seções arsenal/torre-selecionada
    com altura fixa pra selecionar uma torre não redimensionar o tabuleiro.
  - Toque: `<meta viewport>` (faltava no arquivo solto), `touch-action`,
    sem seleção de texto, sem *pull-to-refresh*, e o alvo de construção
    aparece no dedo enquanto ele está na tela via eventos de ponteiro.
  - Verificado com Playwright em 1440×900, 844×390, 568×320, 390×844,
    360×640, 320×568 e 768×1024: nenhuma rolagem em nenhum eixo, nada
    estourando a tela, painel de torre nunca cortado, construir/vender/
    selecionar funcionando pelo mapeamento girado, e 6 ondas em ×2 sem
    nenhum erro de console.

- **Incremento 5a — reforma visual da interface**: emoji (🏰🔊) e glifos
  unicode soltos (⟲) viraram um sprite de `<symbol>` SVG único, reaproveitado
  via `<use>` em toda a página (sempre `currentColor`, acompanha a cor do
  botão/estado). Elevação em duas camadas (`--lift-1`/`--lift-2`: sombra de
  contato justa + sombra ambiente larga, no lugar de um `box-shadow` chapado
  único), friso interno de vidro (`--hairline-top/bottom`), grão de filme
  sutil (SVG `feTurbulence` embutido, sem asset externo) sobre o fundo
  inteiro. Botões com bisel (sobem 1px no `:active`) e layout ícone+rótulo;
  cartas de torre com faixa lateral na cor do tipo (`TOWER_ACCENT`, separada
  da cor do projétil — a do canhão é quase preta). Sidebar de 260→284px pra
  caber ícone+rótulo sem truncar; nos modos compactos pausa/velocidade viram
  só-ícone/só-número quando o painel é estreito demais (`is-side`).
- **Incremento 5b — cenário**: a estrada trocou pebbles soltos por juntas de
  laje de verdade (`tangentAtDistance()` dá a direção do trecho, a normal
  espalha as pedras pela largura da via em fileiras alternadas tipo tijolo —
  só contorno + friso claro no canto, sem preencher, senão fica parecendo
  tabuleiro de xadrez, o que já aconteceu numa primeira tentativa). Castelo
  redesenhado: duas torres redondas de telhado cônico flanqueando um portão
  em arco (era um bloco retangular único com uma porta preta chapada), toda
  a pedra com gradiente diagonal em vez de `fillStyle` sólido. Luz direcional
  suave (`soft-light`) por cima de tudo no fim de `paintBackground()`,
  pintada no espaço do mapa (sem `upright()`) — gira junto com o tabuleiro em
  retrato, como um raio de sol caindo sobre a mesa física. Copa das árvores
  com gradiente radial por bolha em vez de cor sólida.
- **Incremento 5c — torres e inimigos**: `drawTower()` (redesenhada todo
  quadro, não pode usar `shadowBlur`) ganhou gradientes diagonais por peça
  via um helper `grad()` local — trocou `fillStyle` sólido sem custo extra.
  `modeled()` (só roda uma vez, ao assar a folha de sprites dos inimigos)
  ganhou sombra suave por trás de cada peça (`shadowBlur`, de propósito
  proibido em qualquer coisa desenhada ao vivo neste arquivo, mas de graça
  aqui por rodar uma única vez) e uma linha de luz fina por dentro do
  contorno escuro — o par mais barato da mesma ideia.

- **Incremento 6 — caça a bugs + qualidade**: varredura completa do arquivo,
  cada bug reproduzido num navegador headless com uma cópia instrumentada
  (`window.__dbg`) antes e depois da correção. O que estava quebrado:
  - **Derrota virava vitória.** Morrer para o *último* inimigo da onda 15
    chamava `defeat()` dentro de `updateEnemies()` e, no mesmo quadro,
    `waveClearCheck()` via a lista vazia e chamava `victory()` por cima:
    "Cerco repelido!" com 0 vidas e ainda +230 de ouro de bônus. Agora
    `waveClearCheck()` começa com `if(ended) return;`.
  - **Tiro mirava no passado.** O projétil guardava `tx/ty` — a posição do
    alvo no instante do disparo — e voava pra lá; o campo `targetId` era
    gravado e nunca lido. Agora há `leadPoint()`: como todo inimigo anda
    sobre a trilha, dá pra prever exatamente onde ele estará (basta avançar
    `e.dist`; duas iterações convergem). Além disso, tiro de alvo único
    guarda a *referência* do inimigo e persegue enquanto ele vive
    (`p.target`); a bomba do canhão continua caindo no ponto previsto,
    senão a área deixa de ser a vantagem dela. Inimigos removidos ganham
    `e.dead = true` pro projétil saber que deve seguir até o último ponto
    conhecido.

    **Cuidado ao medir isto de novo**: a primeira medição desta sessão deu
    "27% de erro da balista contra corvos" e estava *errada* — era overkill,
    não mira. Com 6 balistas atirando nos mesmos corvos, build antigo e
    novo empatam em ~68% de acerto, porque várias torres disparam num alvo
    que morre antes dos tiros chegarem. Para medir mira é preciso **uma
    torre só**, inimigos bem espaçados, e a torre **longe da estrada** (o
    erro só existe perto do alcance máximo). Nessa montagem:

    | uma torre, sem overkill | distância | vel. do alvo | antigo | novo |
    |---|---|---|---|---|
    | Balista × **Corvo** | 114px | 85 px/s | **63%** | **100%** |
    | Balista × Trasgo | 114px | 62 px/s | 100% | 100% |
    | Arqueiro × Trasgo | 100px | 62 px/s | 100% | 100% |

    Ou seja: o bug existia, mas só contra o inimigo mais rápido do jogo a
    distância longa. É aritmética — tempo de voo × velocidade do alvo tem
    que passar do raio de acerto de 20px, e só o corvo (85 px/s) consegue.
  - **Lentidão do mago encolhia no ×2.** `e.slowUntil` usava `now`
    (relógio de parede) enquanto o movimento usava `dt` (tempo de jogo):
    no ×2 o inimigo percorria 226px sob lentidão contra 191px no ×1.
    Existem agora dois relógios de propósito: `now` só pra enfeite (tremular
    de tocha) e `gameTime`, que anda com `dt` — acelera no ×2/×3 e **congela
    na pausa**. Toda regra usa `gameTime`.
  - **Lentidão fraca aliviava a forte.** Um mago nível 1 acertando depois de
    um nível 2 rebaixava o fator de 0.35 pra 0.45. Agora fica sempre o fator
    mais forte e o prazo mais longo.
  - **"Melhorar" travado.** O botão só era reavaliado ao *selecionar* a
    torre: com a torre já selecionada ele continuava cinza mesmo depois do
    ouro chegar. `updateHUD()` agora reemite o painel da torre selecionada.
  - **Arrastar construía torre.** O `click` do canvas não tinha limiar de
    arrasto — passar o dedo pelo tabuleiro com uma torre armada gastava
    ouro. Agora `pointerdown` registra a origem e um deslocamento acima de
    12px invalida o clique.
  - **Prévia verde fora do tabuleiro.** A célula fora da grade era desenhada
    como válida e o toque não construía nada, sem explicação. A checagem de
    limites entrou na expressão de validade do `render()`.
  - **`card.style.opacity` inline** vencia o `.tower-card:disabled` do CSS —
    virou a classe `.unaffordable`.
  - **`dpr` era lido uma vez na carga**: mudar o zoom do navegador ou
    arrastar a janela pra outro monitor deixava o canvas borrado. Agora é
    reavaliado em cada `layout()`.
  - **`spawnFromQueue` descartava a sobra** (`spawnTimer = spawnInterval`
    em vez de `+=`) e só deixava nascer um inimigo por quadro — no ×2 isso
    esticava a onda. Virou laço com acumulação.

  E o que mudou pra melhor no jogo:
  - **Curva de dificuldade medida, não chutada.** Existe um harness de
    balanceamento no scratchpad (`balance.js`) que joga 15 ondas com um
    *jogador simulado competente*: espalha torres pela estrada, garante
    anti-aéreo antes dos corvos, e melhora quando sobra ouro. Três partidas
    por build, no ×3:

    | build | resultado |
    |---|---|
    | anterior (sorteio uniforme) | 3 vitórias — 18, 6 e 10 vidas |
    | pelotões, 1ª tentativa | 1 vitória — derrota na 9, vitória com 16, derrota na 13 |
    | pelotões, ajustado (atual) | 2 vitórias — derrota na 8, vitória com 8, vitória com 20 |

    O aprendizado: **o que endurece uma onda não é o total de inimigos, é a
    concentração.** Quatro cavaleiros chegando juntos valem muito mais que
    quatro espalhados, mesmo com HP total idêntico. A 1ª tentativa subia
    volume *e* concentração ao mesmo tempo e virou parede. O ajuste devolveu
    o volume ao nível anterior (`hpMul` 7%/onda e contagem `6+1,5n`, ambos
    idênticos ao build antigo) e deixou o pelotão fazer só o ritmo.
    Se precisar mexer nisso de novo, o parâmetro certo é o intervalo entre
    pelotões (`loose`), que controla concentração sem mexer no conteúdo.
    Não vale continuar afinando contra o bot: ele tem estratégia fixa, e
    2 de 3 já é uma curva melhor que o 3/3 folgado do build anterior.
  - **Ondas em pelotões.** `generateWave` sorteava uniformemente de um balde
    de tipos, o que deixava a curva chapada (da onda 6 em diante toda onda
    era a mesma sopa). Agora cada onda é montada em grupos do mesmo tipo,
    com peso variando ao longo do cerco (`waveWeights`): trasgo domina no
    começo e some no fim, orc e cavaleiro crescem, corvo vem em revoada de
    3–5. O intervalo passou a ser por inimigo (`gap`): rajada apertada
    dentro do pelotão, respiro entre pelotões. A onda 15 fecha com **dois**
    Chefes de Guerra.
  - **Anúncio da onda** diz a composição ("Onda 7 — orcs e corvos de
    guerra"), pra dar pra decidir onde gastar o ouro antes de convocar.
  - **Clarão vermelho** quando um inimigo atravessa o portão — perder vida
    era sinalizado só pelo número no topo, fácil de não ver no meio da
    briga. Respeita `prefers-reduced-motion`.
  - **Contador de restantes** no botão da onda em curso.
  - **Velocidade ×3** (era só ×1/×2) e atalhos de teclado novos: **Espaço**
    convoca a onda ou pausa, **U** melhora, **X** vende (1–4 e Esc já
    existiam). A legenda do rodapé lista os atalhos no desktop.

## O que falta (próximos incrementos combinados com o usuário)

**Ainda dentro do Incremento 4 — conteúdo novo:**
- **Torre de Óleo**: segunda torre nova, dano contínuo em área — reaproveita
  a mesma estrutura já usada pro efeito de lentidão do mago (duração +
  efeito aplicado ao inimigo), só que dano ao longo do tempo em vez de
  lentidão.
- **Segundo mapa**: `WAYPOINTS_GRID` vira uma tabela de mapas com 2 entradas;
  escolha na tela inicial, não é possível trocar de mapa em partida em
  andamento.

**Incremento 5 — polimento e balanceamento:**
- Ajuste fino de partículas/animação com base em teste real no celular.
- A legenda de inimigos e as descrições das cartas de torre ficam escondidas
  no modo compacto por falta de espaço — a dica de que só a Balista alcança
  voadores se perde no celular. Vale achar um lugar pra ela (talvez na tela
  inicial quando compacto).
- Balanceamento de custo/dano/HP do conteúdo novo e da curva de ondas.
- Micro-interações de interface (hover/transição, feedback de ouro
  insuficiente).
- Prioridade de alvo por torre (primeiro / mais forte / mais próximo):
  chegou a ser considerada no Incremento 6 e ficou de fora porque a linha
  extra no painel encolhe o tabuleiro no retrato — precisa de um lugar que
  não custe altura.
- O som ainda é um oscilador só por efeito; dá pra enriquecer sem sair do
  WebAudio procedural (ruído filtrado pro canhão, duas vozes na vitória).

## Coisas que uma sessão nova precisa saber

- **Regra de ouro do projeto**: sempre um único arquivo `index.html`
  autocontido. Nunca adicionar CDN, biblioteca externa, ou arquivo de
  imagem — regra do Artifact (CSP só permite a folha do Google Fonts) e
  decisão explícita do usuário (prefere vetor 100% em código a imagens,
  mesmo sabendo que isso tem teto visual — já discutimos e descartamos
  gerar imagens externas).
- **Verificação antes de publicar**: extrair o `<script>` do HTML e rodar
  `node --check`; idealmente também um teste headless via Playwright
  (`/opt/pw-browsers/chromium`, módulos em `/opt/node22/lib/node_modules`
  — usar `NODE_PATH=/opt/node22/lib/node_modules node script.js`) jogando
  algumas ondas pra checar erros de console antes de publicar o Artifact.
- **Publicar sempre no mesmo Artifact** (usar `url` na chamada da tool
  Artifact, não criar um novo) pra manter o link que o usuário já tem.
- **Processo combinado com o usuário**: incrementos pequenos e testáveis,
  ele joga o Artifact publicado e aprova antes do próximo passo — não
  implementar tudo de uma vez sem checkpoint.
- **Repositório**: `KT3746/Claude-Tower-Defense` (foi renomeado de `teste`
  durante o desenvolvimento — o remote local e as chamadas de ferramenta já
  usam o nome novo). Owner é o mesmo, histórico é o mesmo.
- Não fazer PR/merge sem o usuário pedir — mas ele já pediu (e aprovou) uma
  vez pra trazer tudo até aqui pra `main`, então commits diretos em `main`
  pra trabalho já testado e aprovado têm sido aceitos nesta conversa.

## GitHub Pages (acesso sem login)

O usuário pediu isso porque o Artifact exige conta Claude e ele queria abrir
no Safari do celular sem logar em nada. `.github/workflows/pages.yml` cobre
o caso, mas a configuração tem duas pegadinhas que já custaram 3 rodadas de
tentativa e erro — se for mexer nisso de novo, já sai sabendo:

1. **`actions/configure-pages@v5` não cria o site sozinho por padrão.** Sem
   `enablement: true`, ele só tenta *ler* um site que já existe e falha com
   404 se o Pages nunca foi ativado. Mesmo com `enablement: true`, a
   primeira ativação **não pode ser feita pelo `GITHUB_TOKEN` do Actions** —
   dá "Resource not accessible by integration" na criação, sempre, sem
   exceção. Alguém com acesso à conta precisa entrar uma vez em
   Settings → Pages → Build and deployment → Source e escolher
   "GitHub Actions" manualmente. Depois disso o token passa a bastar pra
   tudo (deploys seguintes funcionam sozinhos).
2. **O ambiente `github-pages` que o GitHub cria nesse passo vem com uma
   regra de proteção que só libera deploy a partir da branch padrão do
   repo** (`main`, aqui) — então mesmo com o Pages ativado, publicar a
   partir de `claude/progress-md-context-kj12ok` falhava na hora (sem
   runner, sem logs, só "Branch '...' is not allowed to deploy to
   github-pages due to environment protection rules" na página do run).
   Precisou de um segundo ajuste manual: Settings → Environments →
   github-pages → Deployment branches and tags → trocar pra
   "No restriction" (ou adicionar a branch explicitamente).

Os dois ajustes já foram feitos pelo usuário nesta conversa — não deveriam
ser necessários de novo, a menos que o repo seja recriado do zero ou o
workflow passe a rodar numa branch nova. Se `pages.yml` começar a falhar,
comece descartando essas duas causas antes de mexer no YAML.
