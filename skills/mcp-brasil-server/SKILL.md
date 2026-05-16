---
name: mcp-brasil-server
description: MCP Server connecting AI agents to 70+ Brazilian public APIs for government data, economy, legislation, transparency, judiciary, elections, environment, health, education, and more
triggers:
  - how do I connect to Brazilian government APIs
  - set up mcp-brasil server for Claude
  - query Brazilian public data with AI
  - integrate Brazilian APIs into my MCP client
  - access IBGE TCU TSE data through MCP
  - use mcp-brasil with Claude Desktop
  - query Brazilian legislative or economic data
  - connect to transparency portal API through MCP
---

# mcp-brasil-server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

MCP Server providing 533 tools across 70+ Brazilian public data sources including economy (Banco Central, IBGE), legislation (Câmara, Senado), transparency (Portal da Transparência, TCU), judiciary (DataJud, STF), elections (TSE), environment (INPE, IBAMA), health (DataSUS, ANVISA), education (INEP), and more. Most APIs require no authentication; 4 features use free API keys. Includes local dataset caching with DuckDB for large datasets (TSE elections, SPU properties, ANP fuel prices, INEP school census).

## Installation

### pip/uv

```bash
pip install mcp-brasil
```

```bash
uv add mcp-brasil
```

### Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "mcp-brasil": {
      "command": "uvx",
      "args": ["--from", "mcp-brasil", "python", "-m", "mcp_brasil.server"],
      "env": {
        "TRANSPARENCIA_API_KEY": "your-transparencia-key",
        "DATAJUD_API_KEY": "your-datajud-key",
        "META_ACCESS_TOKEN": "your-meta-token",
        "MCP_BRASIL_DATASETS": "tse_candidatos,anp_precos"
      }
    }
  }
}
```

**Environment variables:**
- `TRANSPARENCIA_API_KEY` (optional) - Portal da Transparência API key
- `DATAJUD_API_KEY` (optional) - DataJud/CNJ API key  
- `META_ACCESS_TOKEN` (optional) - Meta Graph API token for electoral ads
- `MCP_BRASIL_DATASETS` (optional) - Comma-separated list of local datasets to enable: `spu_siapa`, `tse_candidatos`, `tse_bens`, `tse_votacao`, `tse_redes_sociais`, `tse_fefc`, `anp_precos`, `inep_censo_escolar`, `inep_enem`

### VS Code / Cursor

Create `.vscode/mcp.json`:

```json
{
  "servers": {
    "mcp-brasil": {
      "command": "uvx",
      "args": ["--from", "mcp-brasil", "python", "-m", "mcp_brasil.server"],
      "env": {
        "TRANSPARENCIA_API_KEY": "your-key-here"
      }
    }
  }
}
```

### Claude Code CLI

```bash
claude mcp add mcp-brasil -- uvx --from mcp-brasil python -m mcp_brasil.server
```

### HTTP Mode (for other MCP clients)

```bash
fastmcp run mcp_brasil.server:mcp --transport http --port 8000
# Server available at http://localhost:8000/mcp
```

## Key Features

### 533 Tools Across 70 Data Sources

The server auto-discovers features from the `features/` directory. Each feature provides multiple tools:

- **Economy**: `bacen` (9 tools), `bcb_olinda` (8), `bndes` (4), `ipeadata` (5), `b3` (6)
- **Geography**: `ibge` (9 tools for states, municipalities, statistics)
- **Legislative**: `camara` (11), `senado` (26), `governadores` (4)
- **Transparency**: `transparencia` (54), `tcu` (9), `tce_*` (multiple state courts)
- **Judiciary**: `datajud` (7), `jurisprudencia` (6)
- **Electoral**: `tse` (15), `anuncios_eleitorais` (6)
- **Environment**: `inpe` (4), `ana` (3), `ibama` (4)
- **Health**: `saude` (10), `opendatasus` (7), `anvisa` (10), `imunizacao` (10)
- **Education**: `inep` (4), `fnde` (4)
- **Security**: `atlas_violencia` (7), `sinesp` (6)
- **Utilities**: `brasilapi` (16), `dados_abertos` (4), `diario_oficial` (11)

### Smart Tool Discovery

The server uses BM25 search to filter tools based on context:

```python
# When user asks about "inflation and interest rates"
# Only economy-related tools (bacen, bcb_olinda) are shown
# 533 total tools filtered down to ~20 relevant ones
```

### Cross-Reference Planning

Use `planejar_consulta` to create multi-API execution plans:

```python
# Tool: planejar_consulta
# Input: {"objetivo": "Compare spending of deputy X with their voting record on health bills"}
# Returns: Execution plan combining camara tools for expenses + voting + propositions
```

### Batch Execution

Execute multiple queries in parallel:

```python
# Tool: executar_lote
# Input: {
#   "consultas": [
#     {"tool": "bacen_sgs_serie", "params": {"codigo": 433}},  # IPCA
#     {"tool": "bacen_sgs_serie", "params": {"codigo": 4189}}, # Selic
#     {"tool": "ibge_agregados", "params": {"indicador": "pib"}}
#   ]
# }
```

### Local Datasets with DuckDB

Enable large datasets via `MCP_BRASIL_DATASETS` environment variable:

```bash
export MCP_BRASIL_DATASETS="tse_candidatos,tse_votacao,anp_precos"
```

**Available datasets:**
- `spu_siapa` - 813k federal properties
- `tse_candidatos` - 4M+ candidate records (2014-2024)
- `tse_bens` - Candidate asset declarations
- `tse_votacao` - Vote counts by municipality/zone
- `tse_redes_sociais` - Candidate social media URLs
- `tse_fefc` - Electoral fund distributions
- `anp_precos` - Weekly fuel prices at gas stations
- `inep_censo_escolar` - 180k+ schools infrastructure data
- `inep_enem` - 3.9M+ ENEM exam records

First load downloads and ingests data (minutes); subsequent queries run in milliseconds.

## Common Usage Patterns

### Query Economic Indicators

```python
# Get Selic rate time series
tool: bacen_sgs_serie
params:
  codigo: 432
  data_inicio: "2024-01-01"
  data_fim: "2024-12-31"

# Get IPCA inflation
tool: bacen_sgs_serie
params:
  codigo: 433
  
# Get official exchange rate (PTAX)
tool: bcb_olinda_ptax
params:
  data_cotacao: "2024-05-15"
  moeda: "USD"
```

### Search Legislative Data

```python
# Find bills about AI in the Chamber
tool: camara_proposicoes_buscar
params:
  keywords: "inteligência artificial"
  ano: 2024

# Get deputy expenses
tool: camara_deputados_despesas
params:
  deputado_id: "204554"
  mes: 5
  ano: 2024

# Search Senate matters
tool: senado_materias_buscar
params:
  assunto: "meio ambiente"
```

### Access Transparency Data

```python
# Search federal contracts
tool: transparencia_contratos
params:
  data_inicio: "2024-01-01"
  data_fim: "2024-12-31"
  valor_minimo: 1000000

# Get civil servant salaries
tool: transparencia_servidores
params:
  orgao: "Ministério da Saúde"
  mes: 5
  ano: 2024

# Query TCU court decisions
tool: tcu_acordaos
params:
  ano: 2024
  palavras_chave: "licitação fraude"
```

### Electoral Data Queries

```python
# Search candidates (requires tse_candidatos dataset)
tool: tse_candidatos_query
params:
  sql: "SELECT * FROM candidatos WHERE ano_eleicao = 2024 AND uf = 'SP' LIMIT 100"

# Get candidate assets (requires tse_bens dataset)
tool: tse_bens_query
params:
  sql: "SELECT * FROM bens WHERE sq_candidato = '123456' ORDER BY valor DESC"

# Electoral ads from Meta
tool: anuncios_eleitorais_buscar
params:
  eleicao: "2024"
  candidato: "João Silva"
```

### Health and Environment Data

```python
# Search health facilities
tool: saude_cnes_estabelecimentos
params:
  municipio: "São Paulo"
  tipo: "Hospital Geral"

# Get vaccination coverage
tool: imunizacao_cobertura_vacinal
params:
  uf: "SP"
  ano: 2024

# Query wildfire hotspots
tool: inpe_focos_queimadas
params:
  data_inicio: "2024-08-01"
  data_fim: "2024-08-31"
  bioma: "Amazônia"
```

### Geographic and Statistical Data

```python
# List municipalities in a state
tool: ibge_municipios
params:
  uf: "SP"

# Get statistical aggregates
tool: ibge_agregados
params:
  indicador: "população"
  nivel_territorial: "município"
  localidade: "3550308"  # São Paulo city code
```

### Fuel Prices (requires anp_precos dataset)

```python
tool: anp_precos_query
params:
  sql: """
    SELECT municipio, avg(preco_venda) as preco_medio
    FROM precos_gasolina
    WHERE uf = 'SP' AND data_coleta >= '2024-01-01'
    GROUP BY municipio
    ORDER BY preco_medio DESC
    LIMIT 10
  """
```

### School Census (requires inep_censo_escolar dataset)

```python
tool: inep_censo_query
params:
  sql: """
    SELECT uf, count(*) as total_escolas
    FROM escolas
    WHERE tem_internet = 1 AND tem_biblioteca = 1
    GROUP BY uf
    ORDER BY total_escolas DESC
  """
```

## Resources and Prompts

The server provides 131 resources and 102 prompts for common tasks:

**Resources:**
- Dataset schemas and metadata
- Feature documentation
- API endpoint references

**Prompts:**
- "Analyze deputy spending patterns"
- "Compare state-level health indicators"
- "Track bill progress through legislature"
- "Find contract anomalies"

## Development and Extension

### Adding a New Feature

1. Create directory: `mcp_brasil/features/my_feature/`
2. Add `tools.py`:

```python
from mcp_brasil.core import ServerContext
from pydantic import BaseModel, Field

class MyToolInput(BaseModel):
    param1: str = Field(description="Parameter description")
    param2: int = Field(default=10)

async def my_tool(context: ServerContext, params: MyToolInput):
    """Tool description for AI agent."""
    async with context.http.get(
        f"https://api.example.gov.br/endpoint",
        params={"q": params.param1, "limit": params.param2}
    ) as response:
        return await response.json()
```

3. Register automatically via auto-discovery (zero configuration)

### Local Development

```bash
git clone https://github.com/Mcp-Brasil/mcp-brasil.git
cd mcp-brasil
uv venv
source .venv/bin/activate  # or .venv\Scripts\activate on Windows
uv pip install -e ".[dev]"

# Run server
python -m mcp_brasil.server

# Run tests
pytest tests/
```

### Rate Limiting

All HTTP requests use automatic rate limiting with exponential backoff:

```python
# Configured in ServerContext
http = httpx.AsyncClient(
    timeout=30.0,
    limits=httpx.Limits(max_keepalive_connections=10, max_connections=20)
)
# Rate limiting handled automatically per feature
```

## Troubleshooting

### API Keys Not Working

```bash
# Verify environment variables are set
echo $TRANSPARENCIA_API_KEY
echo $DATAJUD_API_KEY

# Most features work without keys - only 4 require them:
# - transparencia (54 tools)
# - datajud (7 tools)
# - anuncios_eleitorais (6 tools)
# - spu_geo (4 tools - uses TRANSPARENCIA_API_KEY)
```

### Dataset Download Fails

```bash
# Check disk space (datasets can be 100MB-1.6GB)
df -h

# Clear cache and retry
rm -rf ~/.mcp_brasil/cache/

# Enable specific datasets only
export MCP_BRASIL_DATASETS="tse_candidatos"  # Start with one
```

### Too Many Tools Shown

The BM25 search transform should filter automatically. If you see all 533 tools:

```bash
# Check MCP client supports search transforms
# Claude Desktop and modern MCP clients do this automatically

# Manually use planejar_consulta to scope your query
```

### HTTP Timeouts

```python
# Increase timeout in ServerContext if needed
# Default is 30s, some large queries may need more

# For dataset queries, ensure dataset is downloaded first
# First query triggers download (minutes), subsequent are instant
```

### Invalid SQL Queries on Datasets

```python
# Get schema first
tool: tse_candidatos_schema  # or other *_schema tool
# Then construct SQL based on actual column names

# Example correct query:
tool: tse_candidatos_query
params:
  sql: "SELECT nm_candidato, sg_partido FROM candidatos WHERE ano_eleicao = 2024 LIMIT 10"
```

## API Key Registration

**Portal da Transparência** (free):
1. Visit https://api.portaldatransparencia.gov.br/
2. Click "Solicitar chave"
3. Fill form with email
4. Receive key instantly

**DataJud/CNJ** (free):
1. Visit https://datajud-api.cnj.jus.br/
2. Register account
3. Generate API key in dashboard

**Meta Access Token** (free):
1. Create Facebook App at https://developers.facebook.com/
2. Get User Access Token
3. Use for electoral ads library

## License and Data Usage

- **Code**: MIT License
- **Data sources**: Each API has its own license/terms (see SOURCES.md in repo)
- **Acceptable Use**: Read ACCEPTABLE_USE.md before commercial/journalistic use
- **Not official**: Independent project, not affiliated with Brazilian government

## Resources

- Repository: https://github.com/Mcp-Brasil/mcp-brasil
- PyPI: https://pypi.org/project/mcp-brasil/
- Data Sources: See SOURCES.md in repository
- Issues: https://github.com/Mcp-Brasil/mcp-brasil/issues
