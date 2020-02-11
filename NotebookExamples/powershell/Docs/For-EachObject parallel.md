---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.3.3+dev
  kernelspec:
    display_name: .NET (PowerShell)
    language: PowerShell
    name: .net-powershell
---

# Using For-EachObject -Parallel in PowerShell 7 <img src="https://raw.githubusercontent.com/PowerShell/PowerShell/master/assets/Powershell_black_64.png" align="right"/>


If you are a PowerShell user you are likely already familiar with the For-EachObject command.
In the past you have used it with the -Proccess parameter.

```powershell
1..5 | ForEach-Object -Process {write-output "This is number $_"; sleep 1}
```

```powershell
(Measure-Command { 
    1..5 | ForEach-Object -Process {write-output "This is number $_"; sleep 1}
    }).Seconds
```

However, with the introduction of the -Parallel paramter in PowerShell 7 we can improve the performance of this command.

```powershell
(Measure-Command { 
    1..5 | ForEach-Object -Parallel {write-output "This is number $_"; sleep 1}
    }).Seconds
```

While this may seem trivial in the example above, you can begin applying this to larger tasks. 

```powershell
function FetchGitHubRepos {
    $orgs = @(
        'PowerShell',
        'Microsoft',
        'dotnet'
    )

    Measure-Command -Expression {
        foreach($org in $orgs) {
            1..10 | ForEach-Object -ThrottleLimit 3 -Parallel {
                (Invoke-RestMethod "https://api.github.com/orgs/$using:org/repos?page=$_") |
                Select-Object -Property full_name,stargazers_count |
                Export-Csv ./repo_stats.csv -Append
            }
        }
    } | Select-Object -Property Minutes, Seconds
}
FetchGitHubRepos
```

If you'd like to see the result of that cell, just run this:

```powershell
Get-Content repo_stats.csv |
    ConvertFrom-Csv |
    Sort-Object { $_.stargazers_count -as [int] } -Descending |
    Select-Object -First 20
```
