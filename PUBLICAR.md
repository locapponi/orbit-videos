# Como publicar / atualizar os vídeos

Página: https://locapponi.github.io/orbit-videos/ (repo público `locapponi/orbit-videos`, GitHub Pages na branch `main`, pasta raiz, com `.nojekyll`).

O repositório é **público**: nada de nomes de clientes reais, marcas de parceiros, e-mails ou dados pessoais em vídeos, arquivos ou commits. Na dúvida, o vídeo fica fora.

## Como a página funciona

- `index.html` não lista vídeos: ele lê `videos.json`. **Para adicionar ou tirar um vídeo, mexa só no `videos.json`** (e nos arquivos em `videos/` e `posters/`).
- A página mostra no topo a obra mais recente (pela `data`), depois uma barra de seções fixa e, em cada seção, uma linha por obra. Tocar abre o player em tela cheia, com as versões em botões, setas para o vídeo anterior/próximo e link direto `#id` (dá para mandar `…/orbit-videos/#institucional`).
- `videos.json` tem duas listas:
  - `secoes`: ordem e nome das seções (`marca`, `produto`, `lancamentos`, `teasers`). Seção sem obra não aparece.
  - `obras`: um objeto por filme, com as versões dele dentro (narrado / sem voz, com legenda / sem legenda, horizontal / vertical):
    ```json
    {
      "id": "temas-plugins",            // vira o link #temas-plugins; sem espaço nem acento
      "titulo": "Temas e Plugins",
      "resumo": "…",                     // opcional, uma frase
      "secao": "lancamentos",
      "data": "2026-11-17",              // AAAA-MM-DD; a mais recente vai para o destaque
      "duracao": "0:50",
      "formato": "16:9",                 // 16:9, 9:16, 4:3, 3:2, 21:9… (proporção do player)
      "audio": "narrado",                // "narrado" ou "sem voz"
      "credito": "Música: …",            // opcional (obrigatório para músicas CC BY)
      "versoes": [
        { "rotulo": "Narrado", "audio": "narrado", "arquivo": "videos/x_narracao.mp4", "poster": "posters/x_narracao.jpg" },
        { "rotulo": "Sem voz", "audio": "sem voz", "arquivo": "videos/x.mp4", "poster": "posters/x.jpg" }
      ]
    }
    ```
  - Uma versão pode sobrescrever `audio`, `formato` e `duracao` da obra (ex.: o teaser vertical 9:16). A primeira versão é a que abre e dá a capa da lista.

## Critério do que entra

Só a versão final/aprovada de cada vídeo (narrada e sem voz quando houver as duas; vertical quando houver). Nada de backups (`_v1`, `_vN` antigas), prévias ou cortes de teste. Fora por ser público: vídeos com cliente real ou marca de parceiro (ex.: restaurante com a Cantina Borgonovo, Síndico e Condomínio com o logo da F&W).

## Passo a passo para adicionar um vídeo

1. ffmpeg do venv:
   ```bash
   FF=$(/home/locapponi/orbit-video-clientes/.venv/bin/python -c "import imageio_ffmpeg;print(imageio_ffmpeg.get_ffmpeg_exe())")
   cd ~/orbit-videos
   ```
2. Copiar com faststart (remux, sem recodificar) e sem metadados:
   ```bash
   $FF -y -i /caminho/do/final.mp4 -map 0:v:0 -map 0:a:0 -c copy -map_metadata -1 -movflags +faststart videos/NOME.mp4
   ```
   Se o áudio for 96 kHz ou o vídeo for 4K, recodificar (abaixo). 4K é pesado demais no celular: reduzir para 1920 de largura.
3. Conferir tamanho: **cada arquivo < 95 MB** (limite do GitHub é 100 MB) e **site inteiro < 800 MB** (`du -sh videos posters`; o limite do Pages é 1 GB). Se passar, recodificar:
   ```bash
   $FF -y -i /caminho/do/final.mp4 -vf "scale='min(1920,iw)':-2" -c:v libx264 -crf 23 -preset slow -pix_fmt yuv420p -c:a aac -b:a 160k -map_metadata -1 -movflags +faststart videos/NOME.mp4
   ```
   Se o total do site passar de 800 MB: recodificar os maiores em 1080p CRF 26, ou deixar só a versão narrada.
4. Poster (quadro em ~30% da duração, 960 px de largura):
   ```bash
   D=$($FF -i videos/NOME.mp4 2>&1 | sed -n 's/.*Duration: \([0-9:.]*\).*/\1/p' | awk -F: '{print ($1*3600+$2*60+$3)*0.3}')
   $FF -y -ss $D -i videos/NOME.mp4 -frames:v 1 -vf "scale=960:-2" -q:v 4 posters/NOME.jpg
   ```
   Abrir o JPG e conferir que o quadro diz algo (se cair numa transição, trocar o tempo).
5. Adicionar a versão (ou a obra nova) no `videos.json` (modelo acima). Validar:
   ```bash
   python3 -c "import json;json.load(open('videos.json'))" && echo ok
   ```
6. Commit com o e-mail noreply (já configurado no repo local; confira **antes**):
   ```bash
   git config user.name    # locapponi
   git config user.email   # 223686191+locapponi@users.noreply.github.com
   git add videos/NOME.mp4 posters/NOME.jpg videos.json
   git commit -m "Adiciona NOME" && git push
   ```
   **Nunca commitar com o e-mail pessoal.** Se o clone for novo, configure os dois valores acima antes do commit. Upload grande: commits pequenos (alguns vídeos por vez).
7. Esperar o build do Pages (gh em `~/.local/bin/gh`):
   ```bash
   gh api repos/locapponi/orbit-videos/pages/builds/latest --jq .status   # repetir até "built"
   ```
8. Conferir (tudo 200):
   ```bash
   B=https://locapponi.github.io/orbit-videos
   curl -s -o /dev/null -w "%{http_code} index\n" $B/
   curl -s -o /dev/null -w "%{http_code} json\n" $B/videos.json
   curl -s -o /dev/null -w "%{http_code} NOME\n" -I $B/videos/NOME.mp4
   ```
   O CDN do Pages pode levar alguns minutos para refletir; no celular, recarregar a página.
