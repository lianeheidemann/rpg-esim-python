Você já está com o Python 3.13.3 e as dependências instaladas. No terminal do VS Code, aberto na pasta do projeto, rode:

```powershell
python tools/make_test_sequence.py --output demo_seq --frames 200
```

Depois execute o simulador:

```powershell
python -m esim.cli --input demo_seq --output demo_out --contrast-threshold 0.2
```

Para gerar uma imagem com o resultado:

```powershell
python -m esim.viz demo_out --save-to preview.png
```

Isso deve criar:

- `demo_seq/`: imagens de entrada sintéticas;
- `demo_out/events.npz`: eventos simulados;
- `demo_out/events.txt`: eventos em texto;
- `demo_out/frames/`: frames com motion blur;
- `preview.png`: visualização dos eventos.

Para rodar os testes:

```powershell
python -m pytest tests -q
```

Na minha verificação, 25 testes passaram. Os outros falharam somente porque meu ambiente restrito não conseguiu escrever na pasta temporária do Windows — no seu terminal normal isso provavelmente não acontecerá.

Caso queira preparar um ambiente virtual do zero:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

O passo a passo completo também está no [README em português](C:/Users/Lily/Documents/GitHub/rpg_esim_python/README_Pt-br.md).