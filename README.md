# Shorts animados com Claude — método original

Uma skill do Claude Code que produz shorts verticais de 55-70 s contando uma história: personagens 2D planos num
mundo 3D colorido, com contorno de tinta, cidade pintada à mão, desfoque de lente, câmeras que deslizam, trilha de
marcha e efeitos sonoros. Tudo é renderizado com **three.js dentro do Remotion**, no seu próprio PC, **sem gerador de
vídeo por IA**: a IA escreve o código que desenha cada frame.

Exemplo pronto: **The Bridge That Fell for a Song**, 67 s sobre a ponte de Broughton (1831), que caiu com 74 soldados
marchando no mesmo passo. Todo o código dele está em `templates/`.

## Como funciona, em uma frase

O Claude segue o `SKILL.md` passo a passo: pesquisa fatos → escreve o roteiro → você gera a voz → ele monta as cenas
em código (`story.ts`) a partir do template da ponte → renderiza stills → agentes comparam com frames de referência e
corrigem → render final 2K + mixagem de som sintetizado.

## O pipeline

| # | Etapa | O que acontece | Arquivos |
|---|---|---|---|
| 1 | Tema e fatos | Uma história real e surpreendente, com causa e efeito físicos e uma analogia do dia a dia | `SCRIPT.md` |
| 2 | Roteiro | ~3,7 palavras/s. Fórmula: gancho → cenário → escalada → estalo → "por sorte/por azar" → explicação → consequência → reviravolta | `templates/SCRIPT_example.md` |
| 3 | Voz | Você grava ou gera `voice.wav`; uma transcrição com tempo por palavra vira `words.json`. **Tudo no vídeo é sincronizado por ele** | `voice/` |
| 4 | Plano de cenas | Copia `templates/bridge/` e reescreve `story.ts`: planos, câmeras, cortes, tremidas, desfoque por batida | `story.ts` |
| 5 | Mundo | Luz, céu, neblina, casas, objetos dos kits Kenney (CC0) recoloridos, texturas pintadas em Canvas2D | `lights.ts`, `build.ts`, `paint.ts`, `world.ts`, `shade.ts` |
| 6 | Personagens | Desenhados em Canvas2D (cabeça de feijão, membros de palito, 14 expressões) e colocados como billboards virados para a câmera | `toon.ts`, `cast.ts`, `cardshade.ts` |
| 7 | Pós-produção | Contorno de tinta → desfoque de lente → bloom → tone map → correção de cor | `Bridge.tsx`, `ink.ts`, `lensblur.ts`, `grade.ts` |
| 8 | Som | Efeitos e música **sintetizados em Python** a partir de uma tabela de eventos, com ducking sob a voz e loudness final | `templates/audio/` |
| 9 | Checagem do look | Stills em meia resolução, folhas lado a lado com os frames de referência, verificação por Opus max | `templates/scripts/` |
| 10 | Render e QA | Render 2K (1440x2560), junção do áudio, QA automático frame a frame (pretos, brancos, congelados, "pops", loudness) | `short_qa.py` |

## Como o código do vídeo é organizado

- **A história** fica em `story.ts` (tabela `SHOTS`, câmeras `CAMS`, fusões `XFADE`, tremidas `KICKS`, desfoque
  `dofAt`, objetos de primeiro plano `FOREGROUND`), no layout do `build.ts` e nos papéis do `cast.ts`.
- **O look** fica nos valores já ajustados dos outros arquivos. Por isso todo vídeo novo começa copiando o template:
  você muda a história e herda o visual.
- **Contratos entre arquivos** são constantes exportadas: a camada `NOINK` (personagens não recebem a tinta do mundo,
  eles têm a própria), `LIGHTS`/`LOOK`, `dofAt`.
- Todo parâmetro em pixels escala com a altura de saída, então stills em 0,5 e o final em 1,333 têm o mesmo look.

## O "look-match loop" (como o visual foi calibrado)

A parte mais poderosa, e mais cara, do método:

1. **Medir a referência.** Um agente por tema (movimento, profundidade, texturas, personagens, iluminação, contornos,
   cor) mede 20 shorts de referência e escreve **números**, não adjetivos, em `references/reports/`.
2. **Sandboxes com dono.** O projeto é copiado em 5 cópias (`cam`, `world`, `mat`, `char`, `post`); cada agente só
   mexe nos próprios arquivos e renderiza os próprios stills.
3. **Juntar** com `bridge_merge.py`, checar tipos com `tsc` e ressincronizar as cópias.
4. **Verificar** com um agente Opus max novo: nota de 0 a 10 por área e lista de correções com número de frame, que
   volta para o agente dono de cada arquivo.

Na ponte foram 4 rodadas; as notas subiram de 5/3/4/5/4/7/4/6 para ~7/5-6/5-6/6/5-6/7/6-7/6-7.

## O que tem em `references/`

| Caminho | Conteúdo |
|---|---|
| `STYLE_BIBLE.md` | Fórmula de roteiro e de imagem medida em 20 shorts de referência |
| `reports/` | Sete relatórios com medidas: movimento, profundidade, texturas, personagens, iluminação, contornos, cor |
| `frames/` | 38 frames de referência (540x960) para comparar com os seus |
| `sheets/` | 20 folhas de contato, uma por short de referência, 60 frames cada |

Esse material é só para estudo do look: nunca copie roteiros, personagens ou planos, e não publique os frames.

## Instalação

1. Copie esta pasta para `~/.claude/skills/cartoon-3d-short/`.
2. Instale Node, Python 3 (`numpy`, `scipy`) e ffmpeg.
3. Crie um projeto [Remotion](https://www.remotion.dev/) com `@remotion/three`, `three`, `@react-three/fiber` e
   `@react-three/postprocessing`. Baixe os kits CC0 da [Kenney](https://kenney.nl) usados em `build.ts` para `public/`.
4. Defina `TEMP`/`TMP` (ou `REMOTION_TMP`) para uma pasta num disco grande: cada bundle do Remotion copia `public/`
   inteiro (~1,3 GB) para a pasta temporária e nunca apaga.
5. No Claude Code: *"Use a skill de short cartoon e faça um short sobre …"*.

## Comandos principais

```bash
# stills em meia resolução (frames 100 e 400)
node scripts/bridge-stills.mjs out/<nome>/x_ 100,400 0.5

# comparação lado a lado com a referência
python scripts/sbs_sheet.py out.jpg --ours a.jpg b.jpg --ref r1.jpg r2.jpg

# som (dentro da pasta audio/)
python mix.py

# render final 2K
npx remotion render src/index.ts <Composicao> out/<nome>/video.mp4 --codec h264 --crf 16 --scale 1.3333

# juntar áudio (o Remotion grava uma faixa muda)
ffmpeg -i video.mp4 -i mix.wav -map 0:v:0 -map 1:a:0 -c:v copy -c:a aac final.mp4

# QA frame a frame
python scripts/short_qa.py final.mp4 out/<nome>/qa shots.json
```

## Custo

O método foi feito para rodar com **Opus no esforço máximo** em todas as checagens, com vários subagentes em paralelo
(um por área e um verificador por rodada). Dá o melhor resultado, mas consome muito limite, principalmente num plano
Pro. Para gastar menos, veja o repositório `shorts-metodo-economico`.

## Tempos de referência

- Stills em 0,5: 3-8 s cada, depois de um bundle de ~40 s.
- Render 2K: ~4 fps (≈10 min para 67 s).
- Arquivo final 2K: ~250 MB (sem grão de filme).
