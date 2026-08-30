# MuleSoft Notes

Estudos técnicos e notas práticas sobre engenharia de software, arquitetura,
segurança, performance, integrações e MuleSoft, publicados no GitHub Pages com
MkDocs.

## Executar localmente

Crie e ative um ambiente virtual do Python e instale as dependências:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Inicie o servidor local:

```powershell
mkdocs serve
```

O site ficará disponível em `http://127.0.0.1:8000`.

## Gerar o site

```powershell
mkdocs build --strict
```

Os arquivos gerados ficam no diretório `site/`, que não é versionado.
