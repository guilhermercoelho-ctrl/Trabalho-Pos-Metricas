[readme.md](https://github.com/user-attachments/files/22710258/readme.md)
# 1322004 - Métricas de Software - Trabalho Prático

Universidade Federal de Juiz de Fora - Outubro 2025

Pós-Graduação em Gerência de Projetos de Software na Era de Dados de Sensores e IA

## Anexo Técnico e Códigos Utilizados

Caso deseje usar uma versão em PDF, utilize o link [aqui](https://github.com/guilhermercoelho-ctrl/Trabalho-Pos-Metricas/blob/main/1322004-2025.1-A%20-%20Anexo%20T%C3%A9cnico.pdf).

## Anexos Disponíveis

O projeto demonstrativo foi criado utilizando-se componentes como Git, jscpd e Radon.
Os arquivos gerados por estas ferramentas estão disponíveis para conferência no diretório
“analysis_data”:

```bash
• commits.json
• contributors_last_year.json
• issues_closed.json
• radon_cc.json
• radon_mi.json
• radon_raw.json
• jscpd-report/ jscpd-report.json
```

Adicionalmente o arquivo “metrics_summary.json” sumariza o resultado dos cálculos com
base nos arquivos brutos acima. Os códigos Python de sumarização podem ser conferidos
em “Analise Metricas.ipynb”

##  Guia de Execução

Os seguintes aplicativos são necessários para executar o código de exemplo:

```bash
• Windows 10/11 (64-bit)
• Git for Windows
• Python 3.8+ com pip
• Node.js + npm (para jscpd)
• Conta GitHub e Personal Access Token (https://github.com/settings/tokens.)
• Jupyter Notebook: Pode ser instalado via comando PIP (pip install notebook). Mais
instruções podem ser encontradas em: https://jupyter.org/install. Alternativamente o
jupyter pode ser carregado ainda via Docker ou instalado junto do pacote Anaconda
(https://www.anaconda.com/download)
```

## Baixar o repositório

Antes de mais nada é necessário baixar o repositório para um diretório local a gosto.

```powershell
git clone https://github.com/scrapy/scrapy.git
```


## Extrair os Dados do Radon

```powershell
# Complexidade Ciclomática
radon cc -a -j . > analysis_data/radon_cc.json
# Dados brutos para análise dos comentários
radon raw -j . > analysis_data/radon_raw.json
# Dados de Manutenibilidade
radon mi -j . > analysis_data/radon_mi.json
```

## Coleta de Dados de Duplicações com o jscpd

```powershell
npx jscpd . --reporters json --output analysis_data/jscpd-report --min-tokens 50 --ignore
"**/scrapy/tests/**"
```

## Estatísticas de Atividade do Repositório (Git API)

Importante notar que o uso da API requer o Token que pode ser obtido em https://github.com/settings/tokens.

```powershell
# Configuração do Token do GitHub
$env:GITHUB_TOKEN = "seu_token_aqui"


# exporta commits com hash, autor, email, data (ISO)
# tem de ser executada dentro da pasta to Scrapy -> cd Scrapy
$lines = git log --all --pretty=format:'%H|%aN|%aE|%aI'
$objects = @()
foreach ($l in $lines -split "`n") {
  $p = $l -split '\|'
  $objects += [PSCustomObject]@{hash=$p[0]; author=$p[1]; email=$p[2]; date=$p[3]}
}
$objects | ConvertTo-Json -Depth 4 > ../analysis_data/commits.json

# Configurações do Nome do Repositório, download issues fechados
$owner = "scrapy"
$repo  = "scrapy"
$per_page = 100
$page = 1
$all = @()
$headers = @{'User-Agent'='PS'}
if ($env:GITHUB_TOKEN) { $headers['Authorization'] = "token $env:GITHUB_TOKEN" }

do {
  $url = "https://api.github.com/repos/$owner/$repo/issues?state=closed&per_page=$per_page&page=$page"
  $resp = Invoke-RestMethod -Uri $url -Headers $headers
  $all += $resp
  $page++
} while ($resp.Count -eq $per_page)

# filtra só issues (remove PRs)
$issues = $all | Where-Object { -not $_.pull_request }
$issues | ConvertTo-Json -Depth 10 > ../analysis_data/issues_closed.json

# Contribuições no último ano
$since = (Get-Date).AddYears(-1).ToString('yyyy-MM-dd')
$short = git shortlog -sne --since="$since"
$objs = @()
foreach ($line in $short -split "`n") {
  if ($line -match '^\s*(\d+)\s+(.*)\s+<(.*)>$') {
    $objs += [PSCustomObject]@{commits=[int]$matches[1]; author=$matches[2]; email=$matches[3]}
  }
}
$objs | ConvertTo-Json -Depth 4 > ../analysis_data/contributors_last_year.json

```


## Resultados

Para uma extração individual dos resultados compilados no relatório final favor verificar o PDF do anexo técnico ([aqui](https://github.com/guilhermercoelho-ctrl/Trabalho-Pos-Metricas/blob/main/1322004-2025.1-A%20-%20Anexo%20T%C3%A9cnico.pdf)).
