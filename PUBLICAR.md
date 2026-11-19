# Como publicar / atualizar os vídeos

Página: https://locapponi.github.io/orbit-videos/ (repo público `locapponi/orbit-videos`, GitHub Pages na branch `main`, pasta raiz).

O repositório é **público**: nada de nomes de clientes, e-mails ou dados pessoais em arquivos ou commits.

## Nomes exatos dos arquivos (em `videos/`)

| Arquivo | Seção da página |
|---|---|
| `videos/orbit-studio-eixo_narracao.mp4` | Novo: Studio Eixo · narrado |
| `videos/orbit-studio-eixo.mp4` | Novo: Studio Eixo |
| `videos/orbit-sistema-v3_narracao.mp4` | Versões anteriores: Sistema v3 · narrado |
| `videos/orbit-temas-plugins_narracao.mp4` | Versões anteriores: Temas e Plugins · narrado |

Enquanto os do Studio Eixo não existem, a página mostra "Em produção — volte mais tarde" (HEAD 404 → aviso). Ao subir o arquivo com o nome exato, ele aparece sozinho; não precisa mexer no `index.html`.

## Passo a passo

1. ffmpeg do venv:
   ```bash
   FF=$(/home/locapponi/orbit-video-clientes/.venv/bin/python -c "import imageio_ffmpeg;print(imageio_ffmpeg.get_ffmpeg_exe())")
   ```
2. Copiar com faststart (remux, sem recodificar) e sem metadados:
   ```bash
   cd ~/orbit-videos
   $FF -y -i /caminho/do/final.mp4 -c copy -map_metadata -1 -movflags +faststart videos/orbit-studio-eixo_narracao.mp4
   ```
3. Conferir tamanho: **cada arquivo < 95 MB** (limite do GitHub é 100 MB). Se passar, recodificar:
   ```bash
   $FF -y -i /caminho/do/final.mp4 -c:v libx264 -crf 23 -preset slow -pix_fmt yuv420p -c:a aac -b:a 160k -map_metadata -1 -movflags +faststart videos/NOME.mp4
   ```
4. Commit com o e-mail noreply (já configurado no repo local; confira):
   ```bash
   git config user.name    # locapponi
   git config user.email   # 223686191+locapponi@users.noreply.github.com
   git add videos/ && git commit -m "Adiciona Studio Eixo" && git push
   ```
   Nunca commitar com o e-mail pessoal. Se o clone for novo, configure os dois valores acima antes do commit.
5. Esperar o build do Pages (gh em `~/.local/bin/gh`):
   ```bash
   gh api repos/locapponi/orbit-videos/pages/builds/latest --jq .status   # repetir até "built"
   ```
6. Conferir:
   ```bash
   curl -sI https://locapponi.github.io/orbit-videos/videos/orbit-studio-eixo_narracao.mp4 | head -1   # 200
   ```
   O CDN do Pages pode levar alguns minutos para refletir; no celular, recarregar a página.
