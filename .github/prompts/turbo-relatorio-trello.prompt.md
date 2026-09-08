---
name: "turbo-relatorio-trello"
description: "Gera relatório semanal do quadro Trello em Markdown e salva em .github/ do projeto atual. Triggers: 'gera relatório', 'relatório trello', 'relatório do board', 'gera o relatório semanal'."
tools: [execute, todo]
---

# Prompt: Gerar Relatório Trello

Gerar relatório Markdown do board Trello e salvar em `.github/relatorio-YYYY-MM-DD.md` do workspace atual. Executar os 3 passos em sequência.

---

## Passo 1 — Credenciais e seleção de board

```powershell
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$OutputEncoding = [System.Text.Encoding]::UTF8
Set-StrictMode -Off

$key = $env:TRELLO_KEY; $token = $env:TRELLO_TOKEN; $base = "https://api.trello.com/1"

if (-not $key -or -not $token) {
    Write-Output "ERRO: Configure TRELLO_KEY e TRELLO_TOKEN como variaveis de ambiente do Windows."
    Write-Output '[System.Environment]::SetEnvironmentVariable("TRELLO_KEY","<key>","User")'
    return
}

$boardId = $env:TRELLO_BOARD_ID
if ($boardId) {
    $b = Invoke-RestMethod "$base/boards/$boardId?key=$key&token=$token&fields=name"
    Write-Output "BOARD_DEFINIDO: $($b.name) [ID: $boardId]"
} else {
    $boards = Invoke-RestMethod "$base/members/me/boards?key=$key&token=$token&fields=name,id&filter=open"
    $i = 1; $boards | ForEach-Object { Write-Output "$i. $($_.name) [ID: $($_.id)]"; $i++ }
}
```

Se `BOARD_DEFINIDO` aparecer no output, use o ID informado e vá direto ao Passo 2. Caso contrário, pergunte ao usuário qual board usar e aguarde a resposta antes de continuar.

---

## Passo 2 — Coletar, processar e gerar rascunho

Substitua `{BOARD_ID}` pelo ID escolhido e execute:

```powershell
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$OutputEncoding = [System.Text.Encoding]::UTF8
Set-StrictMode -Off

$key = $env:TRELLO_KEY; $token = $env:TRELLO_TOKEN; $base = "https://api.trello.com/1"
$boardId = "{BOARD_ID}"

# Emojis por nome de lista
function Get-Emoji($n) {
    if ($n -like "*Backlog*")                               { return "📦" }
    if ($n -like "*Planejad*" -and $n -notlike "*amento*")  { return "📐" }
    if ($n -like "*Planejamento*")                          { return "📋" }
    if ($n -like "*Desenvolv*")                             { return "🔨" }
    if ($n -like "*Revis*" -or $n -like "*Review*")         { return "🔍" }
    if ($n -like "*Finaliz*" -or $n -like "*Done*")         { return "✅" }
    return "📋"
}

# Coletar tudo em memoria — sem arquivos intermediarios
$boardInfo = Invoke-RestMethod "$base/boards/$boardId?key=$key&token=$token&fields=name"
$listas    = Invoke-RestMethod "$base/boards/$boardId/lists?key=$key&token=$token&filter=open"
$cards     = Invoke-RestMethod "$base/boards/$boardId/cards?key=$key&token=$token&filter=open&fields=name,idList,due,shortUrl,dateLastActivity"

# Agrupar cards por lista
$hoje = Get-Date
$h = @{}; $listas | ForEach-Object { $h[$_.id] = [System.Collections.Generic.List[object]]::new() }
$cards | ForEach-Object { if ($h.ContainsKey($_.idList)) { $h[$_.idList].Add($_) } }

# Metricas
$listaFinal      = $listas | Where-Object { $_.name -like "*Finaliz*" -or $_.name -like "*Done*" } | Select-Object -First 1
$totalCards      = $cards.Count
$cardsConcluidos = if ($listaFinal) { $h[$listaFinal.id].Count } else { 0 }
$percentual      = if ($totalCards -gt 0) { [math]::Round($cardsConcluidos / $totalCards * 100, 1) } else { 0 }

# Cards concluidos nos ultimos 7 dias (na lista Finalizado com atividade recente)
$limite7d    = $hoje.AddDays(-7)
$proximo7d   = $hoje.AddDays(7)
$cardsConcluidos7d = if ($listaFinal) {
    $h[$listaFinal.id] | Where-Object { $_.dateLastActivity -and [datetime]::Parse($_.dateLastActivity).ToLocalTime() -ge $limite7d }
} else { @() }

# Cards com previsao de entrega nos proximos 7 dias (excluindo lista finalizado)
$idListaFinal = if ($listaFinal) { $listaFinal.id } else { $null }
$cardsPrevisto7d = $cards | Where-Object {
    $_.due -and
    $_.idList -ne $idListaFinal -and
    [datetime]::Parse($_.due).ToLocalTime() -ge $hoje -and
    [datetime]::Parse($_.due).ToLocalTime() -le $proximo7d
}
$barraPreench    = [math]::Round($percentual / 100 * 20)
$barra           = ([string][char]0x2588) * $barraPreench + ([string][char]0x2591) * (20 - $barraPreench)
$atrasados       = 0

# Tabela de progresso
$tabela = "| Coluna | Cards |`n|--------|-------|"
foreach ($l in $listas) { $tabela += "`n| $(Get-Emoji $l.name) $($l.name) | $($h[$l.id].Count) |" }

# Secoes de cards
$secoes = ""
foreach ($lista in $listas) {
    $cl = $h[$lista.id]
    $secoes += "`n## $(Get-Emoji $lista.name) $($lista.name) ($($cl.Count))`n`n"
    if ($cl.Count -eq 0) { $secoes += "_Nenhum card nesta coluna._`n"; continue }
    foreach ($c in $cl) {
        $pfx = "- "
        if ($c.due -and [datetime]::Parse($c.due).ToLocalTime() -lt $hoje -and $lista.name -notlike "*Finaliz*" -and $lista.name -notlike "*Done*") {
            $pfx = "- ⚠️ "; $atrasados++
        }
        $secoes += "$pfx[$($c.name)]($($c.shortUrl))`n"
    }
}

# Dados para o resumo executivo
$concluidos7dStr = if ($cardsConcluidos7d.Count -gt 0) { ($cardsConcluidos7d | Select-Object -ExpandProperty name) -join "; " } else { "nenhum" }
$previsao7dStr   = if ($cardsPrevisto7d.Count -gt 0) {
    ($cardsPrevisto7d | ForEach-Object { "$($_.name) (vence $(([datetime]::Parse($_.due).ToLocalTime()).ToString('dd/MM'))" + ")" }) -join "; "
} else { "nenhum" }

$rodape = if ($atrasados -gt 0) { "`n> ⚠️ **$atrasados card(s) com prazo vencido** — atencao necessaria.`n" } else { "" }

# Montar rascunho com placeholder para o resumo executivo
$rascunho  = "# Relatorio do Projeto — $($boardInfo.name)`n"
$rascunho += "`n> Gerado em **$(Get-Date -Format 'dd/MM/yyyy')**`n`n---`n"
$rascunho += "`n## 🎯 Visao Executiva`n`n{{RESUMO_EXECUTIVO}}`n`n---`n"
$rascunho += "`n## Progresso Geral`n`n``$barra`` **$percentual%**`n`n$tabela`n`n---`n"
$rascunho += "$secoes`n---`n$rodape"
$rascunho += "`n_Gerado via GitHub Copilot | [Board no Trello](https://trello.com/b/$boardId)_"

$rascunho | Out-File "$env:TEMP\trello_rascunho.md" -Encoding UTF8

# Output compacto para o Copilot usar no resumo executivo
Write-Output "BOARD: $($boardInfo.name)"
Write-Output "STATS: Total=$totalCards | Fin=$cardsConcluidos | Perc=$percentual% | Atrasados=$atrasados"
Write-Output "CONCLUIDOS_7D: $concluidos7dStr"
Write-Output "PREVISAO_7D: $previsao7dStr"
Write-Output "RASCUNHO: $env:TEMP\trello_rascunho.md"
```

---

## Passo 3 — Escrever resumo executivo e salvar

Com base nas linhas `CONCLUIDOS_7D` e `PREVISAO_7D` do output acima, escreva o resumo executivo em **texto narrativo em linguagem de negócios** (sem termos técnicos), seguindo este modelo:

- **Últimos 7 dias:** descreva em 1-2 frases o que foi entregue. Se `CONCLUIDOS_7D` for "nenhum", informe que não houve entregas no período.
- **Próximos 7 dias:** descreva em 1-2 frases o que está previsto para entrega. Se `PREVISAO_7D` for "nenhum", informe que não há previsões para o período.

O resumo substituirá completamente o conteúdo da seção `🎯 Visao Executiva`. Execute o script abaixo substituindo `{RESUMO_AQUI}` pelo texto gerado:

```powershell
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
Set-StrictMode -Off

$resumo = "{RESUMO_AQUI}"

$conteudo = (Get-Content "$env:TEMP\trello_rascunho.md" -Raw) -replace '{{RESUMO_EXECUTIVO}}', $resumo

$raiz = (git rev-parse --show-toplevel 2>$null); if (-not $raiz) { $raiz = $PWD.Path }
$pastaGithub = Join-Path $raiz ".github"
if (-not (Test-Path $pastaGithub)) { New-Item -ItemType Directory -Path $pastaGithub | Out-Null }
$destino = Join-Path $pastaGithub "relatorio-$(Get-Date -Format 'yyyy-MM-dd').md"

$conteudo | Out-File $destino -Encoding UTF8
Write-Output "SAVED: $destino"
```

---

## Regras

- Nunca exiba `$key` ou `$token` em nenhum output.
- Se qualquer chamada à API do Trello falhar, pare e informe o erro — não gere relatório com dados parciais.
- O resumo executivo deve refletir os cards reais — não invente tarefas.
- Se não houver cards em uma lista, a seção já conterá `_Nenhum card nesta coluna._`.
- Confirme ao usuário o caminho `SAVED:` ao final.

