# ESIM (port em Python): um simulador de câmera de eventos

<p align="left">
  <img src="https://img.shields.io/badge/Python-3.8%2B-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python 3.8+">
  <img src="https://img.shields.io/badge/NumPy-1.21%2B-013243?style=for-the-badge&logo=numpy&logoColor=white" alt="NumPy">
  <img src="https://img.shields.io/badge/OpenCV-4.5%2B-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white" alt="OpenCV">
  <img src="https://img.shields.io/badge/Matplotlib-3.4%2B-11557C?style=for-the-badge" alt="Matplotlib">
</p>

[![Generate Demo](https://github.com/lianeheidemann/rpg_esim_python_v2/actions/workflows/demo.yml/badge.svg)](https://github.com/lianeheidemann/rpg_esim_python_v2/actions/workflows/demo.yml)
[![Process Video](https://github.com/lianeheidemann/rpg_esim_python_v2/actions/workflows/video-demo.yml/badge.svg)](https://github.com/lianeheidemann/rpg_esim_python_v2/actions/workflows/video-demo.yml)

*[Read in English](README.md)*

> Origem: este código é uma migração (de C++ para Python) e adaptação do núcleo de geração de eventos do ESIM, publicado originalmente por Henri Rebecq, Daniel Gehrig e Davide Scaramuzza (Robotics and Perception Group, Universidade de Zurique), no artigo "ESIM: an Open Event Camera Simulator" (CoRL 2018) — repositório original em uzh-rpg/rpg_esim. O modelo de simulação de eventos (o algoritmo, os limiares de contraste, o período refratário etc.) é dos autores originais. A reescrita em Python puro, a estrutura do pacote, o CLI, as ferramentas auxiliares (src/tools/) e os testes são trabalho feito para este repositório.

<p align="left">
  <img src="assets/event_camera_hand-2.gif">
</p>

Um port em Python puro do núcleo de geração de eventos do [ESIM](https://github.com/uzh-rpg/rpg_esim), um simulador open-source para câmeras de eventos (sensores classe DVS/DAVIS). Dado uma pasta de imagens de intensidade com timestamp, ele reproduz o modelo de eventos original por pixel — incluindo ruído no limiar, período refratário e geração de frames com motion blur — sem nenhuma dependência de ROS, catkin ou toolchain C++.

```bibtex
@Article{Rebecq18corl,
  author        = {Henri Rebecq and Daniel Gehrig and Davide Scaramuzza},
  title         = {{ESIM}: an Open Event Camera Simulator},
  journal       = {Conf. on Robotics Learning (CoRL)},
  year          = 2018,
  month         = oct
}
```

O artigo está disponível [aqui](http://rpg.ifi.uzh.ch/docs/CORL18_Rebecq.pdf). Se você usar este código, cite a publicação acima.

## O que este projeto é (e não é)

Este repositório porta apenas o **pipeline de geração de eventos** do projeto C++/ROS original — a parte que transforma uma sequência de imagens de intensidade em eventos. Ele **não** inclui os renderizadores de cena do original (planar, panorama, OpenGL, UnrealCV), a simulação de trajetória/IMU, nem a publicação em tópicos ROS/gravação de rosbag. Se você precisar dessas partes, use o [ESIM C++/ROS original](https://github.com/uzh-rpg/rpg_esim) ou os [bindings Python acelerados por GPU](https://github.com/uzh-rpg/rpg_vid2e), que envolvem a mesma implementação de referência.

Na prática, isso significa: **você fornece as imagens** (renderizadas do jeito que preferir, ou uma sequência real de vídeo/fotos), e esta ferramenta simula o que uma câmera de eventos teria capturado.

## Funcionalidades

- Port fiel do modelo de eventos em C++: limiarização por intensidade logarítmica ou linear, limiares de contraste positivo/negativo separados (C+/C-), ruído gaussiano aditivo sobre os limiares e período refratário por pixel
- Síntese de frames com motion blur através de um tempo de exposição finito, junto com o fluxo de eventos
- Entrada simples baseada em pasta (`images.csv` + arquivos de imagem) e saída em arquivos (eventos `.npz` / `.txt`, sequência de frames em PNG)
- Uma API Python pequena e com poucas dependências (`esim.EventSimulator`, `esim.CameraSimulator`) utilizável fora do CLI
- Um utilitário de visualização (`esim.viz`) para renderizar a imagem de eventos acumulados e o gráfico de taxa de eventos
- Apenas NumPy, OpenCV e Matplotlib como dependências — sem ROS, sem extensões compiladas, roda em qualquer lugar onde o Python roda (Windows, macOS, Linux)

## Arquitetura

```
images.csv + frames ──▶ FolderImageSource ──▶ EventSimulator ──▶ events.npz / events.txt
                                          └──▶ CameraSimulator ──▶ frames/ (PNGs com blur)
```

- **`FolderImageSource`** ([src/esim/data_provider.py](src/esim/data_provider.py)) lê uma sequência de imagens com timestamp a partir do disco.
- **`EventSimulator`** ([src/esim/event_simulator.py](src/esim/event_simulator.py)) compara o sinal de intensidade (log) contra os limiares de contraste por pixel e emite eventos, respeitando o ruído no limiar e o período refratário.
- **`CameraSimulator`** ([src/esim/camera_simulator.py](src/esim/camera_simulator.py)) integra a intensidade ao longo de uma janela de exposição para sintetizar frames convencionais com motion blur.
- **`esim.cli`** ([src/esim/cli.py](src/esim/cli.py)) conecta os três módulos na ferramenta de linha de comando `python -m esim.cli`.
- **`esim.writers`** ([src/esim/writers.py](src/esim/writers.py)) e **`esim.viz`** ([src/esim/viz.py](src/esim/viz.py)) cuidam da E/S de saída e da visualização.

## Estrutura do repositório

| Caminho | Conteúdo |
| --- | --- |
| [`src/esim/`](src/esim) | O pacote do simulador: tipos, simuladores de evento/câmera, provedor de dados, CLI, writers, visualização |
| [`src/tests/`](src/tests) | Testes unitários e ponta a ponta (`unittest`) |
| [`src/tools/`](src/tools) | Scripts independentes: gerador de sequência de teste sintética, construtor de `images.csv`, extrator de frames de vídeo |
| [`src/requirements.txt`](src/requirements.txt) | As três dependências de execução |

## Requisitos

- Python 3.8+
- `numpy`, `opencv-python`, `matplotlib` (veja [src/requirements.txt](src/requirements.txt))
- Roda em Windows, macOS e Linux — não precisa de ROS, catkin, vcstool ou ferramentas de build em C++
- Funciona dentro de um `venv` comum ou de um ambiente conda; nada aqui exige conda especificamente

## Instalação

```bash
conda create -n esim python=3.10
conda activate esim
cd src
pip install -r requirements.txt
```

(Um `venv` comum funciona da mesma forma — troque as duas primeiras linhas por `python -m venv .venv` e a ativação dele.)

Todos os comandos abaixo (rodar o simulador, as ferramentas, os testes) devem ser executados de dentro de `src/`.

## Preparando a entrada

O simulador lê uma pasta contendo um índice `images.csv` e os arquivos de imagem que ele referencia:

```
seq/
├── images.csv
├── frame_000000.png
├── frame_000001.png
└── ...
```

O `images.csv` tem um par `timestamp_ns,filename` por linha (linhas começando com `#` ou `%` são comentários):

```
# timestamp_ns, image
0,frame_000000.png
1000000,frame_000001.png
```

Dois utilitários são fornecidos:

- **`tools/generate_stamps_file.py`** constrói o `images.csv` para uma pasta de imagens que você já tem, a uma taxa de frames fixa:
  ```bash
  python tools/generate_stamps_file.py -i path/to/frames -r 1000
  ```
- **`tools/make_test_sequence.py`** renderiza uma grade sintética em translação, do início ao fim — útil para uma demonstração rápida ou para testes, já que produz um fluxo de eventos denso e previsível:
  ```bash
  python tools/make_test_sequence.py --output demo_seq --frames 200
  ```
- **`tools/premiere_video.py`** extrai os frames de um vídeo (`.mp4` e outros formatos suportados pelo OpenCV) e já gera o `images.csv` correspondente:
  ```bash
  python tools/premiere_video.py -i video/video.mp4 -o video_input
  ```
  Veja [doc/converter_video.md](doc/converter_video.md) para o passo a passo completo, do vídeo até os frames de evento.

## Executando o simulador

```bash
python -m esim.cli --input demo_seq --output demo_out --contrast-threshold 0.2
```

(O limiar de contraste padrão de `1.0` é ajustado para renderizações de faixa dinâmica completa; a grade sintética de demonstração acima tem um contraste modesto, então é preciso um limiar menor, como `0.2`, para de fato gerar eventos. Ajuste-o conforme o contraste da sua própria sequência de imagens.)

Os argumentos também podem ser mantidos em um arquivo e carregados com `@`, uma flag por linha (isso espelha os flagfiles usados pela ferramenta C++ original):

```bash
python -m esim.cli @cfg/my_run.conf
```

### Flags

| Flag | Padrão | Descrição |
| --- | --- | --- |
| `-i`, `--input` | *(obrigatório)* | Pasta contendo `images.csv` e as imagens |
| `-o`, `--output` | *(obrigatório)* | Pasta onde os resultados serão gravados |
| `--contrast-threshold` | — | Define C+ e C- de uma vez (sobrepõe as duas flags abaixo) |
| `--contrast-threshold-pos` | `1.0` | Limiar de contraste positivo (ON), C+ |
| `--contrast-threshold-neg` | `1.0` | Limiar de contraste negativo (OFF), C- |
| `--contrast-threshold-sigma-pos` | `0.0` | Desvio padrão do ruído gaussiano somado a C+ |
| `--contrast-threshold-sigma-neg` | `0.0` | Desvio padrão do ruído gaussiano somado a C- |
| `--refractory-period-ns` | `0` | Tempo mínimo entre dois eventos no mesmo pixel |
| `--no-log-image` | desativado | Aplica os limiares sobre a intensidade bruta em vez da logarítmica |
| `--log-eps` | `0.001` | Epsilon somado antes do log, para estabilizar pixels escuros |
| `--random-seed` | — | Seed para o ruído do limiar (não determinístico se não definido) |
| `--exposure-time-ms` | `10.0` | Tempo de exposição usado para sintetizar o motion blur |
| `--no-blurred-frames` | desativado | Não gera nenhum frame com motion blur |
| `--no-txt` | desativado | Não gera o arquivo `events.txt` (ainda grava `events.npz`) |
| `--quiet` | desativado | Suprime a saída de progresso |

`--contrast-threshold-sigma-pos/neg` aqui têm padrão `0` em vez do `0.021` do original, o que corresponde a todas as configurações que o ESIM original distribui: o ruído no limiar esvazia o fluxo de eventos quando a variação de intensidade por frame é muito menor que o próprio ruído.

### Saída

```
demo_out/
├── events.npz          # x, y, t (ns), pol — veja esim.writers.load_events_npz
├── events.txt           # "t x y pol" por linha, t em segundos (omitido com --no-txt)
└── frames/              # frames com blur + images.csv (omitido com --no-blurred-frames)
```

## Visualizando os resultados

```bash
python -m esim.viz demo_out/events.npz
python -m esim.viz demo_out --save-to preview.png   # sem interface gráfica, grava um PNG em vez de abrir uma janela
```

Isso renderiza a imagem de eventos acumulados (azul = predominância de ON, vermelho = predominância de OFF) ao lado do gráfico de taxa de eventos ao longo do tempo.

<p align="left">
  <img src="assets/viz_preview.png" alt="Prévia dos eventos acumulados e da taxa de eventos">
</p>

Para transformar `events.txt` em uma sequência de frames (azul = ON, vermelho = OFF),
agrupando os eventos em janelas de 10 ms:

```bash
python -m esim.event_frames demo_out/events.txt --output demo_out/event_frames --window-ms 10
```

O comando grava `frame_000000.png`, `frame_000001.png`, etc., além de um
`images.csv` com o timestamp de cada frame. Use uma janela menor para mais detalhe
temporal ou uma janela maior para acumular mais eventos em cada imagem.

## Usando como biblioteca

```python
from esim import EventSimulator, EventSimConfig, CameraSimulator

sim = EventSimulator(EventSimConfig(Cp=0.2, Cm=0.2, random_seed=0))
camera = CameraSimulator(exposure_time_ms=10.0)

for stamp_ns, image in my_image_sequence:       # image: array 2D em [0, 1]
    events = sim.image_callback(image, stamp_ns)  # array estruturado, esim.EVENT_DTYPE
    frame = camera.image_callback(image, stamp_ns)  # None até que uma janela de exposição seja preenchida
```

## Executando os testes

```bash
python -m pytest tests/ -q
```

## Agradecimentos

Este é um port do [ESIM](https://github.com/uzh-rpg/rpg_esim), do Robotics and Perception Group (Universidade de Zurique). Todo o crédito pelo modelo de geração de eventos subjacente é dos autores originais; veja a citação acima.

## Licença

Distribuído sob a Licença MIT. Veja [LICENSE](LICENSE).
