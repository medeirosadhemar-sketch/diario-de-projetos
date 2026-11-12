# Dragon: um app de música próprio em cima do Jellyfin

**Projeto:** Dragon · **Data:** 12/11/2026 · **Série:** parte 1

Dragon é a interface de música que montei em cima do Jellyfin do meu servidor de casa. Começou como um tema em CSS, ganhou um site complementar para explorar a biblioteca e um app Android, e na versão 2.0 virou um app só, só de música. Este texto é a visão geral da série. Os detalhes de arquitetura, do tema e do script de aplicação ficam para os próximos posts.

## Contexto

A meta era ter a biblioteca organizada e com uma interface boa, aproveitando o que funciona em Spotify, Apple Music, Tidal, Plexamp e YouTube Music. A regra visual é preto e branco, com destaque em branco e nunca em azul. A única exceção são os mapas, que usam cor por gênero.

As artes de fundo e das capas são de domínio público. Primeiro usei o dragão de Kanō Hōgai, depois gravuras em madeira com anjos e, na 2.0, gravuras de Doré e Dürer.

## As três peças antes da 2.0

### 1. Jellyfin com tema próprio

O Jellyfin só aceita CSS no painel de Branding. O tema fica no `branding.xml`. Organizei as 20 melhorias em 4 levas de 5:

| Leva | Foco | Exemplos |
|---|---|---|
| 1 | Início | play redondo que sobe sobre a capa, capas arredondadas com sombra |
| 2 | Álbum/artista | cabeçalho com capa grande, selos com contorno para faixas/duração/ano |
| 3 | Player | "Tocando agora" com fundo desfocado, letra grande com linha atual em branco, fila em cartões |
| 4 | Celular/TV | layout a 400 px, contorno de foco para TV/teclado, coração que pula ao favoritar |

Três problemas desse estágio:

- O Jellyfin 12.1 mistura o layout antigo com componentes React/MUI. O cabeçalho e o avatar só mudaram com seletores `header.MuiAppBar-root`.
- Uma linha cortava a tela. A causa era `background-attachment: fixed`, que saiu do tema base.
- O item 19, um brilho de carregamento nas capas, usava `!important` e cobria as capas, que ficavam pretas. Ficou desligado.

### 2. Site complementar (FastAPI em container)

O site tem visual próprio e chegou a 28 melhorias. A parte de exploração da biblioteca é a mais interessante do ponto de vista de interface:

- **Mapa:** grafo da biblioteca no estilo do Obsidian. Cada música liga ao seu artista, artistas que cantam juntos ficam ligados, artistas parecidos aparecem em tracejado e as playlists são quadrados. Dá para arrastar, dar zoom e procurar um nó, e o duplo clique encaixa o grafo. Existe também um mapa por gênero (`#mapa/rock`).
- **Trilha:** fluxograma deitado. Começa numa música conhecida e oferece 3 prévias de 30 s. A escolhida continua a linha, e o resultado pode virar playlist.
- **Blocos** (`#blocos`): escolho gêneros e recebo os blocos de cada um, as pontes entre eles e a mistura.
- **Montar** (`#montar`): arrasto gêneros, artistas e músicas da coluna para a mesa, reordeno, tiro e toco.

O código fica dividido assim:

```text
Busca/
  app.py            # servidor FastAPI
  static/
    index.html
    partes.js       # Mapa e Trilha
    blocos.js       # Blocos e Montar
    acabamento.css/.js
    submenus.css/.js
```

### 3. App Android

O app é uma WebView de 17 kB (pacote `br.dragon.app`), compilada sem Gradle:

```text
aapt2 → javac → d8 → apksigner
```

A versão 1.6 passou a levar a interface do site dentro do APK. Ela abre na hora e se atualiza em segundo plano. As capas ficam em cache de até 200 MB, apagando as menos usadas.

A versão 1.7 transformou o aviso em player e levou os controles para a tela de bloqueio, o fone Bluetooth e o relógio. A ponte funciona assim: um script injetado em toda página olha `navigator.mediaSession` e o `<audio>` a cada 1 s e chama `DragonNativo.estado(...)` só quando algo muda. Os botões do aviso voltam para a página por `__dragonFazer(acao)`.

## Cor por gênero no Mapa

Na contagem de 04/11, a biblioteca tinha 429 músicas e 528 artistas:

| Gênero | Músicas | Artistas |
|---|---|---|
| Rock | 24,0% | 59 |
| Funk | 17,7% | 172 |
| Rap/Hip Hop | 12,6% | 70 |
| Eletrônica | 11,0% | 68 |
| Alternativo | 10,3% | 42 |
| Pop | 9,8% | 49 |
| Sertanejo | 2,3% | 3 |
| MPB | 1,9% | 9 |
| Outros | 10,3% | 56 |

A paleta categórica foi validada em modo escuro contra o fundo preto. Ela passa nos pares vizinhos, mas com as 8 cores juntas alguns pares se confundem: rosa e verde-água para daltonismo, vermelho e laranja e também lilás e azul até para visão normal.

Por isso a cor não carrega a informação sozinha. No mapa geral, cada gênero fica na sua região, os pares parecidos ficam em lados opostos e o nome do gênero vai escrito. O mapa de um único gênero usa uma cor só.

## Dragon 2.0

Em 09/11 o app virou um só, só de música e sem nada por cima:

- **Início**, **Buscar** e **Biblioteca**;
- tela **Tocando** com letra, fila e timer para dormir.

A versão antiga continua acessível em `/antigo/`. As mudanças são publicadas com `Busca/publicar-app-novo.sh`.

Em 10/11 as funções antigas voltaram numa aba **Explorar**: Mapa, Trilha, Blocos, Montar e Pedidos.

Vários elementos da tela Tocando vieram de uma pesquisa em apps Flutter, principalmente o Finamp e os exemplos oficiais do Flutter. Entre eles:

- a capa que passa do mini player para a tela cheia (View Transitions);
- arrastar para baixo para fechar, com fling pela velocidade;
- deslizar a capa para pular a música;
- tirar da fila com Desfazer;
- timer que baixa o volume devagar antes de pausar;
- continuar de onde parou.

Os selos de qualidade mostram o kbps calculado pelo tamanho do arquivo. Em teoria, a taxa média é:

$$\text{taxa (kbps)} = \frac{8 \cdot \text{tamanho (bytes)}}{1000 \cdot \text{duração (s)}}$$

Essa conta inclui o peso de capa e tags embutidas no arquivo, então é uma estimativa. O selo de FLAC/kbps ainda não foi conferido com o servidor.

## Números

| Item | Valor |
|---|---|
| Melhorias no tema do Jellyfin | 20, em 4 levas |
| Melhorias do site complementar | 28 |
| Melhorias vindas de apps Flutter | 20 |
| Tamanho do APK inicial | 17 kB |
| Cache de capas no app | até 200 MB |
| Fundo do tema (PNG → JPG) | 3,9 MB → 424 KB |

A troca do fundo veio de um travamento. O Jellyfin enviava o PNG original de 3200×2139 sem reduzir, e ele era baixado e decodificado várias vezes. Troquei por um JPG de 1920 px.

## Limitações e próximos passos

- **Celular de verdade:** ainda não testei. Vibração, puxar para atualizar e controles na tela de bloqueio estão compilados, mas não foram conferidos no aparelho.
- **Emulador:** caiu 4 vezes na primeira carga com a guarda local ligada. Com a guarda desligada, ou no Chrome do emulador, não caiu. Depois passou duas vezes sem cair. A causa não está confirmada.
- **Memória:** o container do site encostou no limite de 512 MB, com o swap do servidor em 100%, e a busca parou de responder. Reiniciar libera a memória. A suspeita é um cache no `app.py`, ainda não confirmada.
- **Letra:** na aba controlada por automação, as páginas do Jellyfin rodavam a cerca de 1 quadro/s, e a letra não rolava até a linha atual. Falta ver se isso acontece no uso normal.
- Importar uma playlist e criá-la no Dragon ainda não foi testado de ponta a ponta.

---
`#Jellyfin` `#FastAPI` `#Android` `#CSS` `#SelfHosted`
