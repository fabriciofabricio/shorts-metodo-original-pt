---
name: cartoon-3d-short
description: Faz um short vertical de ~55-70 s contando uma história no estilo "personagens 2D num mundo 3D colorido" (personagens planos com cabeça de feijão e contorno de tinta, cidade/mundo 3D pintado com linhas pretas de tinta, céu saturado, fundos com profundidade de campo, câmeras deslizando, narração, efeitos sonoros sintetizados + música estilo marcha), renderizado com three.js dentro do Remotion. Testado em "The Bridge That Fell for a Song" (2026-10-31). Use quando o usuário pedir outro vídeo "como o da ponte", "como o short dos soldados/da ponte", "naquele estilo cartoon 3D", um short de história/ciência com personagens de desenho em 3D, ou disser "use a skill de short cartoon".
---

# Short de história cartoon 3D (three.js + Remotion)

Exemplo testado: **The Bridge That Fell for a Song**. Um short original de 67 s sobre a ponte de Broughton (1831) e
74 soldados marchando no mesmo passo.
- Código: `src/videos/bridge/` do seu projeto Remotion (composição `BridgeShort`, 1080x1920, 30 fps).
- Roteiro, voz e som: `~/bridge-short/`.
- Pesquisa de estilo: `references/`.

A referência de estilo é um canal estudado em particular (`references/`: STYLE_BIBLE.md, reports/*.md, frames/).
- Esses vídeos são material de estudo privado. Nunca envie, publique ou mande para alguém.
- Nunca copie roteiros, personagens ou planos deles. História, personagens e voz são sempre NOSSOS.
- Nunca diga o nome do canal na narração ou na descrição.

Regras do usuário que valem:
- Sem legendas nem textos no vídeo.
- Só em inglês.
- Voz: um tom constante.
- As piadas são fatos reais pesquisados.
- Mixagem padrão: música +3 dB, efeitos -6 dB, voz sem alteração.
- Exportação final 2K vertical: 1440x2560, `--scale 1.3333`.
- Opus no esforço máximo confere cada checagem.
- Salvar depois de cada passo.
- "Pode fazer" depois que o roteiro for aprovado.

## Pipeline
1. **Tema e fatos.** Escolha uma história verdadeira e surpreendente com causa e efeito físicos (precisa ser visual),
   mais uma analogia simples do dia a dia que a explique.
   - Escreva os fatos com a fonte no topo do `SCRIPT.md`.
2. **Roteiro** (~3,7 palavras/s, 55-70 s). Fórmula de batidas do short da ponte:
   - **Gancho:** o fato absurdo numa linha ("Setenta e quatro soldados derrubaram uma ponte de ferro só andando sobre ela.").
   - **Cenário:** ano, lugar, orgulho.
   - **Ação:** escalada em três ou quatro batidas, cada uma um pouco maior.
   - **Estalo:** o momento em que quebra.
   - **Linha dupla seca:** "Por sorte ... / Por azar ...".
   - **Explicação:** uma ideia de física através de uma analogia do dia a dia (a criança no balanço).
   - **Consequência:** a regra que surgiu.
   - **Reviravolta de retorno:** o final.
   - Nunca descreva na narração o que o espectador já está vendo.
3. **Voz.** Grave ou gere `voice.wav` e depois os tempos das palavras `words.json` (qualquer transcrição com tempo por palavra).
   - Coloque na pasta do projeto, ex.: `~/bridge-short/voice/`.
   - Corrija os tempos das palavras à mão. Tudo é sincronizado pelo words.json.
4. **Plano de cenas** em `story.ts`: copie `templates/bridge/` (o código final da ponte, igual ao `src/videos/bridge/` do
   seu projeto Remotion) para `src/videos/<nome>/`, registre uma composição nova ao lado de `BridgeShort` em
   `src/Root.tsx` e renomeie.
   - `SHOTS`: id, frames a/b, luz day/golden, modo de multidão, `tau` (tempo da história).
   - `XFADE`: fusões nas mudanças de lugar/tempo.
   - `FLASH_AT`: um flash branco depois do gancho.
   - `CAMS`: chaves por plano `{f, dur?, cut?, s: f => {pos, look, fov, roll}}`.
   - `KICKS`: tremidas de impacto.
   - `DOF`/`dofAt`: desfoque de lente por batida.
   - `FOREGROUND`: objetos de primeiro plano para enquadrar.
   - Toda reação visual cai 0-3 frames antes da sua palavra.
5. **Mundo.**
   - `lights.ts`: LIGHTS mais a paleta LOOK de céu/rio/nuvem por luz.
   - `shade.ts`: sombreamento em dois tons e `untone()`.
   - `build.ts`: cidade, casas e objetos dos kits CC0 da Kenney, recoloridos, com remoção por linha de visão de
     tudo que bloquear uma câmera.
   - `paint.ts`: texturas pintadas em HD com Canvas2D, com as juntas pintadas como tons escuros.
   - `world.ts`: shader do céu, neblina = cor do horizonte, rio, pose por frame, sol congelado por plano e relativo à câmera.
6. **Personagens.**
   - `toon.ts`: a API de desenho `Pen` em Canvas2D, tela de desenho 600x1000, solas em y = 975, cabeças de feijão,
     membros de palito, contorno ameixa de largura variável, biblioteca de rostos.
   - `cast.ts`: billboards virados para a câmera, multidões, cartões de efeito, piscadas, ciclos de caminhada.
     Todos os billboards ficam na camada `NOINK`.
   - `cardshade.ts`: cartões iluminados pela luz da cena, sem emissão.
7. **Pós-produção** (`Bridge.tsx` + `ink.ts` + `lensblur.ts` + `grade.ts`).
   - Ordem do compositor: tinta → lente pré → lente principal → bloom → tone map NEUTRAL → correção de cor (MSAA 4).
   - Todo parâmetro em pixels escala com a altura de saída / 1920.
8. **Som** (`~/bridge-short/audio/`: timeline.py, sfx_gen.py, sfx.py, music.py, instruments.py, dsp.py, mix.py).
   - Tabela de eventos → efeitos sintetizados mais uma trilha original abaixada sob a voz (ducking).
   - Saída: mix.wav (48 kHz, duração exata), stems e cues.md.
   - Trave as constantes de tempo da imagem antes de fazer o som. Qualquer mudança de tempo depois obriga a refazer o som.
9. **Checagem do look.** Rode `node scripts/bridge-stills.mjs out/<nome>/x_ 100,400 0.5`.
   - Defina `BRIDGE_ENTRY=src/sandboxes/<n>/index.tsx` para renderizar uma cópia sandbox.
   - Monte comparações lado a lado com frames de referência: `python scripts/sbs_sheet.py out.jpg --ours ... --ref ...`.
   - Mande para um verificador Opus max: 1-3 imagens, veredito de ≤200 palavras.
10. **Render final.**
    - `npx remotion render src/index.ts <Comp> out/<nome>/video.mp4 --codec h264 --crf 16 --scale 1.3333`
    - Junte o áudio com `-map 0:v:0 -map 1:a:0` (o Remotion grava uma faixa de áudio muda).
    - QA: `python scripts/short_qa.py final.mp4 out/<nome>/qa shots.json` confere cada frame procurando quadros pretos,
      brancos, congelados e "pops", além de loudness e folhas de contato.
    - Corrija o que aparecer e depois rode uma auditoria profunda com Opus.
11. **Entrega.** Envie o mp4 final ao usuário (SendUserFile), atualize a memória, escreva as notas.

## Valores finais do look (ponte, depois de 4 rodadas de verificação; todos em `templates/bridge/`)
Comece todo short novo a partir de `templates/bridge/` (copie para `src/videos/<nome>/`): o look mora nesses valores, a
história mora em story.ts / layout do build.ts / papéis do cast.ts. Notas do Opus max no fim: cor dia 7, golden 5-6,
iluminação 5-6, tinta 6, texturas 5-6, personagens 7, profundidade 6-7, composição 6-7 (eram 5/3/4/5/4/7/4/6 no começo).
- **Presets de luz** (`lights.ts`, sol relativo à câmera e congelado por plano):
  - day: az 75 / el 48, lit [1.58,1.5,1.33], shade [.30,.29,.36], lift [.05,.06,.11] (tom frio SÓ na sombra), céu
    topo #3e7fd2 → horizonte #b8d0ec, neblina = horizonte, 25→300 m, uSat 1.08, ~15 % de névoa azul além de 60 m.
  - golden: az 6 / el 7 (sol baixo por trás), luz principal para #FFB36B ×1.5, sombra azul-violeta (fachada ≈ #444380),
    céu #ffc47c → #8a6f8a, neblina pêssego só além de 50 m, disco do sol #ffcf9e + bloom, manchas de contato + sombras
    reais dos cartões.
- **Albedos** (`paint.ts`/`build.ts`): casas de tijolo apagado / creme / ocre / terracota com rodapé, cornija, peitoris,
  marquises, degrau da porta, chaminé + vaso, alturas 7 / 8,4 / 9,8 m; pedra cinza/bege quente (saturação −35 % em
  relação a "livro de histórias"); deque de madeira ocre C* ~51-58 com juntas quase pretas; ferro escuro e
  dessaturado. Nunca amarelo-limão, nunca rostos iluminados em lavanda.
- **Tinta** (`ink.ts`): #030303, 5,1 px perto (6 m) → 3,3 px longe (90 m) em 1080x1920, vincos ×0,8 a partir de 35°,
  saltos de profundidade a partir de 1 %, tom de neblina 0,3 só ao longe; cartões de personagem na camada `NOINK`
  (eles têm a própria tinta ameixa).
- **Lente** (`lensblur.ts` + `dofAt` em story.ts): cInf px em 1080: gancho/close do rio 15-16, closes 12-14, plano médio
  10-12, parque 7, balanço golden 5, gerais/aéreos 0. Foco nos olhos da primeira fila.
- **Pós** (`Bridge.tsx`): tinta → lente pré → lente principal → bloom (thr 1.0, int .5, 6 níveis, raio .6) → tone map
  NEUTRAL → correção de cor; MSAA 4. O grão de filme existe mas fica DESLIGADO (a referência não tem grão; ele
  quadruplica o tamanho do arquivo).
- **Personagens** (`toon.ts`/`cast.ts`/`cardshade.ts`): cabeças de feijão h/w .94, tinta ameixa #241726, olhos a .26 W
  de distância, sem blush, 14 expressões; adultos com 3,5 cabeças de altura; balanço da caminhada 5 %, inclinação 1-7°,
  passada plantada de 0,8 m; piscadas a cada 0,5-2,3 s; sombra de contato justa ~.5 sob os pés + cauda na direção do sol
  de dia; contraluz golden só por dentro (nunca uma cópia deslocada por fora).
- **Câmera** (`story.ts`): cortes nos saltos grandes, deslizes smootherstep de 18-40 f, todo plano parado avança devagar
  (nenhum plano estático ≥4 f), tremidas que invertem e decaem em 7 f nos impactos, rolagens de 4-4,6°, objetos de
  primeiro plano (`FOREGROUND`) em ~1/3 dos planos.

## Metas de look (medidas na referência; ver references/reports/)
- **Tinta:**
  - Linhas do mundo: cerca de 3 px em 1080 de largura, quase preto #0b0a0c, quase constante com a distância. Só
    silhuetas, saltos de profundidade e vincos de normal. Sem linha nas junções parede/chão nem no horizonte. Os cartões
    dos personagens nunca recebem contorno da tinta do mundo.
  - Tinta dos personagens: ameixa escuro #201628-#2f1d28, mais grossa no lado de baixo/sombra.
- **Cor:**
  - Luma mediana de dia ~.57. p5 fora da tinta ~.23 (sem pretos esmagados). Croma iluminado C* p90 ~55. Topo do céu de
    dia ~#5e93d9, horizonte 1,28x mais claro.
  - Golden: horizonte pêssego #ffd08a, sombras azul-violeta (nunca marrons), sol por trás.
- **Profundidade:**
  - DOF em cerca de 1/3 do tempo de tela (closes, planos médios, inserts), cInf 10-16 px em 1080, foco nos olhos.
    Planos de ambientação e gerais ficam nítidos.
  - Neblina = cor do horizonte, 25→420 m de dia, 20→360 m no golden.
  - Bloom só em emissivos HDR: threshold 1.0.
- **Câmera:**
  - Cortes secos nos saltos grandes. O resto é deslize smootherstep de 18-40 f, pico ≤ 25 px/f em 720p.
  - Todo plano parado avança 0,15-0,35 %/f, com 0,6 %/f nas palavras de impacto.
  - Rolagens de 4-5°.
  - Nada de câmera na mão flutuando. Tremidas de impacto invertem o sinal a cada frame e decaem em 7 f.
- **Personagens:**
  - Cerca de 3,5 cabeças de altura, pernas 1,4 cabeça.
  - Membros de palito com cerca de 0,055 da largura da cabeça, mãos de ponto de tinta.
  - Cabeça de feijão (h/w ~0,93, queixo mais estreito, topo mais pontudo).
  - Sem blush nem brilho nos olhos. Rostos com brilho ~.93.
  - Balanço da caminhada 3-7 %, inclinação 1-7°, pés plantados.
  - Piscadas a cada 0,5-2,3 s, durando 4 f.

## Look-match loop (como o look da ponte foi alcançado; reutilize em qualquer passada de "deixe parecido com X")
1. **Meça a referência primeiro.** Um agente de análise por tema (movimento, profundidade, texturas, personagens,
   iluminação, contornos, cor), cada um escrevendo números, não adjetivos, em `references/reports/<tema>.md`.
   Briefing comum: `compare/CONTEXT.md`. Os relatórios são a fonte da verdade para todo agente depois.
2. **Sandboxes, um dono por arquivo.** Copie a pasta do projeto para `src/videos/<nome>_{cam,world,mat,char,post}` com
   entradas `src/sandboxes/<n>/index.tsx` (o mesmo `index.tsx` do principal, só muda o caminho do import). Divisão usada
   na ponte:
   - cam: story.ts (chaves de câmera, cortes, tremidas, tabela de DOF, objetos de primeiro plano)
   - world: world.ts, lights.ts, shade.ts, grade.ts (sol, céu, neblina, rio, cor, correção)
   - mat: paint.ts, build.ts (texturas pintadas, prédios, objetos)
   - char: toon.ts, cast.ts, cardshade.ts (desenho dos personagens, poses, luz dos cartões, cartões de efeito)
   - post: ink.ts, Bridge.tsx, lensblur.ts (passe de contorno, compositor, desfoque de lente, bloom)
   Os agentes só editam os próprios arquivos no próprio sandbox e renderizam stills com
   `BRIDGE_ENTRY=src/sandboxes/<n>/index.tsx node scripts/bridge-stills.mjs out/<nome>/<n>/x_ <frames> 0.5`.
3. **Junte** com `python scripts/bridge_merge.py [cam,world,mat,char,post]` (faz backup do principal em
   `backup/bridge_<hora>/`, copia só os arquivos de cada dono). Depois `npx tsc --noEmit` (filtrando a pasta do projeto)
   e **ressincronize todo sandbox a partir do principal** (`cp -f src/videos/<nome>/* src/videos/<nome>_<n>/`) antes da
   próxima rodada.
4. **Verifique** com um agente Opus max novo: folhas lado a lado (`python scripts/sbs_sheet.py out.jpg --ours a b --ref
   r1 r2`), nota de 0 a 10 por área (cor dia / golden, iluminação, tinta, texturas, personagens, profundidade,
   composição) mais uma lista numerada de correções com números de frame. Ligue cada correção ao dono do arquivo e
   retome o agente desse dono com SendMessage (mantém o contexto). Notas da ponte: rodada 1 5/3/4/5/4/7/4/6 → rodada 2
   5/4/5/6/4/7/6/6 → rodada 3 ver notas.
5. Os contratos entre donos ficam em constantes exportadas (ex.: camada `NOINK` do ink.ts, `dofAt`/`FOREGROUND` do
   story.ts, `LIGHTS`/`LOOK` do lights.ts). Mudar um contrato significa avisar todo dono que o lê.

## Armadilhas
- Os materiais do nature-kit da Kenney são verde-azulados: recolora pelo nome do material em build.ts.
- Câmeras acabam dentro das casas: o removedor por linha de visão em build.ts/world.ts tira tudo que bloqueia qualquer
  raio de câmera.
- Tinta no mesmo passe de efeito que o DOF é sobrescrita. A tinta precisa ser um passe próprio, antes do desfoque.
- Parâmetros de pós em pixels precisam escalar com a resolução: stills são em 0.5, o final em 1.333. O R3F limita o
  dpr a ≥ 1, então stills com `--scale 0.5` ainda renderizam um buffer de 1080x1920.
- **Disco:** todo bundle do Remotion copia `public/` (~1,3 GB) para `pasta temporária\remotion-webpack-bundle-*` e
  nunca apaga. Mais de 60 GB encheram o disco do sistema durante as passadas da ponte e um render morreu com ENOSPC.
  Sempre renderize com `export TEMP='<disco grande>/remotion-tmp' TMP='<disco grande>/remotion-tmp'` (o
  bridge-stills.mjs já define isso) e pergunte ao usuário antes de apagar bundles antigos na pasta temporária
  compartilhada (o modo automático bloqueia a limpeza). `<disco grande>/remotion-tmp` também enche (~500 pastas / 110 GB
  depois das 4 rodadas da ponte): quando um projeto terminar e nada estiver renderizando, apague as pastas
  `remotion-webpack-bundle-*` e `puppeteer_dev_chrome_profile-*` de lá.
- Em strings JS escreva `"<disco grande>/remotion-tmp"`, nunca `"<disco grande>\remotion-tmp"` (`\r` vira retorno de carro).
- O Remotion grava uma faixa de áudio muda: junte com `-map 0:v:0 -map 1:a:0`.
- Subagentes não conseguem gravar arquivos de relatório neste ambiente: peça a eles uma mensagem final de ≤150
  palavras e salve você mesmo em `out/<nome>/<n>/REPORT.md`.
- Arquivos `.ts` de rascunho dentro de `out/` quebram o `tsc`: use a extensão `.txt`.
- Renders 2K rodam a ~4 fps (≈10 min para 67 s). Stills em 0.5 levam 3-8 s cada depois de um bundle de ~40 s.
- Grão de filme em 2,5 % deixou o arquivo 2K com 910 MB em vez de ~250 MB e vira papa no YouTube; a referência não tem
  grão. Deixe desligado.

## Armadilhas de leitura que o verificador pegou (confira em todo objeto/plano novo)
- **Um balanço desenhado com uma corda só que some atrás da criança parece um ENFORCAMENTO.** Sempre duas cordas até
  uma tábua larga com a criança entre elas (`swingKid` em toon.ts), e uma estrutura cujas pernas fiquem fora do arco
  (cavalete em A em cada ponta).
- Objetos ou destroços a menos de ~2 m da lente viram bolhas enormes cortadas na borda do quadro. Quando uma câmera se
  move, confira de novo todo objeto perto da nova posição (as câmeras do rio entraram nos chapéus boiando).
- Um arbusto de primeiro plano cortado pela borda do quadro parece uma "bola verde"; um objeto de primeiro plano precisa
  ser reconhecível ou sair.
- Bandeiras: vermelho + estrela/losango amarelo parece a do Vietnã. Use uma bandeira da época (cruz de São Jorge para a
  Inglaterra de 1831).
- Placas pintadas iluminadas por sol lateral ficam cinza-azuladas e ilegíveis: dê à face da placa um material sem luz.
- Contraluz golden feito como cópia deslocada por fora transforma todo chapéu numa moldura: só contraluz interno, só
  cabeça/tronco.
- Corrimãos de sacada/grade na altura do peito atravessando os personagens: coloque a lente ~2,5 m acima para que os
  corrimãos cruzem as pernas, não os troncos.
- Planos parados parecem congelamentos no QA (trecho congelado ≥12 f): todo plano parado precisa de um avanço lento.
