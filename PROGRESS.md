# Muralha — status do projeto

Tower defense medieval, arquivo único autocontido (`index.html`: HTML+CSS+JS
inline, sem build, sem dependências além da folha de fontes do Google Fonts).
Publicado como Claude Artifact: https://claude.ai/code/artifact/b0742332-e8be-417e-a8fa-2e358a728366

Todo o histórico abaixo está mesclado na `main`. Se você (uma sessão nova) foi
chamado para continuar este projeto, comece lendo `index.html` inteiro — são
~2070 linhas, um único IIFE comentado por seções — e depois volte aqui.

## O que já existe (v1 + v2, incrementos 1 a 3c)

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
- Balanceamento de custo/dano/HP do conteúdo novo e da curva de ondas.
- Micro-interações de interface (hover/transição, feedback de ouro
  insuficiente).

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
