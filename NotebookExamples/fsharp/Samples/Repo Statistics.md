---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.3.3+dev
  kernelspec:
    display_name: .NET (F#)
    language: F#
    name: .net-fsharp
---

# dotnet/fsharp Github Dashboard <img src ="https://upload.wikimedia.org/wikipedia/commons/thumb/3/38/Jupyter_logo.svg/207px-Jupyter_logo.svg.png" width="60px" alt="dotnet bot in space" align ="right">

### Add NuGet package references

```fsharp
#r "nuget:Octokit, 0.32.0"
#r "nuget:NodaTime, 2.4.6"

open Octokit
open NodaTime
open NodaTime.Extensions
open XPlot.Plotly
```

### Setup

Create a GitHub public API client

```fsharp
let organization = "dotnet"
let repositoryName = "fsharp"
let options = ApiOptions()
let gitHubClient = GitHubClient(ProductHeaderValue("notebook"))
```

[Generate a user token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) to get rid of public [api](https://github.com/octokit/octokit.net/blob/master/docs/getting-started.md) throttling policies for anonymous users 

```fsharp
// let tokenAuth = Credentials("YOUR-TOKEN-HERE")
// gitHubClient.Credentials <- tokenAuth
```

```fsharp
let today = SystemClock.Instance.InUtc().GetCurrentDate()
let startOfTheMonth = today.With(DateAdjusters.Month(1))
let startOfTheYear = LocalDate(today.Year, 1, 1).AtMidnight()

let since t = Nullable(DateTimeOffset(t))
```

Query GitHub for : 
- Issues created this month
- Issues closed this month
- Every issue this year

```fsharp
let createdIssuesRequest =
    RepositoryIssueRequest(
        Since = since (startOfTheMonth.ToDateTimeUnspecified()),
        Filter = IssueFilter.Created)
let closedIssuesRequest =
    new RepositoryIssueRequest(
                Since = since (startOfTheMonth.ToDateTimeUnspecified()),
                State = ItemStateFilter.Closed)
let thisYearIssuesRequest =
    RepositoryIssueRequest(
        Since = since (startOfTheYear.ToDateTimeUnspecified()),
        State = ItemStateFilter.All)
```

Start pulling data via the GitHub API

```fsharp
let createdThisMonthTask =
    async {
        return!
            gitHubClient.Issue.GetAllForRepository(organization, repositoryName, createdIssuesRequest)
            |> Async.AwaitTask
    }

let closedThisMonthTask =
    async {
        return!
            gitHubClient.Issue.GetAllForRepository(organization, repositoryName, closedIssuesRequest)
            |> Async.AwaitTask
    }

let thisYearIssuesTask =
    async {
        return!
            gitHubClient.Issue.GetAllForRepository(organization, repositoryName, thisYearIssuesRequest)
            |> Async.AwaitTask
    }

let results =
    [| createdThisMonthTask; closedThisMonthTask; thisYearIssuesTask |]
    |> Async.Parallel
    |> Async.RunSynchronously

let createdThisMonth = results.[0]
let closedThisMonth = results.[1]
let thisYearIssues = results.[2]
```

Group open and closed issues by month

```fsharp
let openSoFar =
    createdThisMonth
    |> Seq.sortBy (fun i -> i.CreatedAt)
    |> Seq.filter (fun i -> i.State.StringValue = "open")

let openByMonthOfCreation =
    openSoFar
    |> Seq.groupBy (fun i -> {| Year = i.CreatedAt.Year; Month = i.CreatedAt.Month |})
    |> Seq.map (fun (key, issues) -> {| Date = key; Count = issues.Count() |})
    
let closedSoFar =
    thisYearIssues
    |> Seq.sortBy (fun i -> i.CreatedAt)
    |> Seq.filter (fun i -> i.State.StringValue = "closed")

let closedByMonthOfClosure =
    closedSoFar
    |> Seq.groupBy (fun i -> {| Year = i.ClosedAt.Value.Year; Month = i.ClosedAt.Value.Month |})
    |> Seq.map (fun (key, issues) ->  {| Date = key; Count = issues.Count() |})

let openCountByMonth =
    let mutable runningTotal = thisYearIssues.Count
    
    closedSoFar
    |> List.ofSeq
    |> List.groupBy (fun i -> {| Year = i.CreatedAt.Year; Month = i.CreatedAt.Month |})
    |> List.map (fun (key, issues) ->
                   let dataPoint = {| Date = key; Count = issues.Count() |}
                   dataPoint)
```

Show issues opened this month grouped by day 

```fsharp
let createdThisMonthByDay =
    createdThisMonth
    |> Seq.groupBy (fun i -> DateTime(i.CreatedAt.Year,i.CreatedAt.Month, i.CreatedAt.Day))
    |> Seq.map (fun (date, issues) -> (date, issues.Count()))

createdThisMonthByDay
|> Chart.Line
|> Chart.WithTitle("Daily created issues over the past year")
```

Show open issues, in descending order. Limit to 10 to save screen space.

```fsharp
openSoFar
|> Seq.map (fun i -> {| CreatedAt = i.CreatedAt; Title = i.Title; State = i.State.StringValue; Number = i.Number |})
|> Seq.sortByDescending (fun d -> d.CreatedAt)
|> Seq.take 10 // Limiting the output to 10 here!
```

Let's see what issues still opened, grouped by month, looks like

```fsharp
openByMonthOfCreation
|> Seq.map (fun g -> (DateTime(g.Date.Year, g.Date.Month, 1), g.Count))
|> Chart.Line
|> Chart.WithTitle("Issues still opened, grouped by month")
```

Now let's look at idle vs active issues.

```fsharp
let idleByMonth =
    openSoFar
    |> Seq.filter (fun i -> i.Comments = 0)
    |> Seq.groupBy (fun i -> DateTime(i.CreatedAt.Year, i.CreatedAt.Month, 1))
    |> Seq.map(fun (key, issues) -> {| Date = key; Count = issues.Count() |})

let activeByMonth =
    openSoFar
    |> Seq.filter (fun i -> i.Comments > 0)
    |> Seq.groupBy (fun i -> DateTime(i.CreatedAt.Year, i.CreatedAt.Month, 1))
    |> Seq.map (fun (key, issues) -> {| Date = key; Count = issues.Count() |})

let idleSeries =
    Graph.Scattergl(
        name = "Idle",
        y = (idleByMonth |> Seq.map (fun g -> g.Count)),
        x = (idleByMonth |> Seq.map (fun g -> g.Date)))

let activeSeries =
    Graph.Scattergl(
        name = "Active",
        y = (activeByMonth |> Seq.map (fun g -> g.Count)),
        x = (activeByMonth |> Seq.map (fun g -> g.Date)))

[idleSeries; activeSeries]
|> Chart.Plot
|> Chart.WithTitle("Idle and active open issue report")
```

Now let's generate a report for the whole year.

```fsharp
let openDataPoints =
    openByMonthOfCreation
    |> Seq.map (fun g -> {| Date = DateTime(g.Date.Year, g.Date.Month, 1); Count = g.Count |})
    |> Seq.sortBy (fun d -> d.Date)

let closedDataPoints =
    closedByMonthOfClosure
    |> Seq.map (fun g -> {| Date = DateTime(g.Date.Year, g.Date.Month, 1); Count = g.Count |})
    |> Seq.sortBy (fun d -> d.Date)

let openCountByMonthDataPoints =
    openCountByMonth
    |> Seq.map (fun g -> {| Date = DateTime(g.Date.Year, g.Date.Month, 1); Count = g.Count |})
    |> Seq.sortBy (fun d -> d.Date)

let openSeries =
    Graph.Scattergl(
        name = "Created",
        x = (openDataPoints |> Seq.map (fun g -> g.Date)),
        y = (openDataPoints |> Seq.map (fun g -> g.Count)))

let closeSeries =
    Graph.Scattergl(
        name = "Closed",
        x = (closedDataPoints |> Seq.map (fun g -> g.Date)),
        y = (closedDataPoints |> Seq.map (fun g -> g.Count)))

let stillOpenSeries =
    Graph.Scattergl(
        name = "Open",
        x = (openCountByMonthDataPoints |> Seq.map (fun g -> g.Date)),
        y = (openCountByMonthDataPoints |> Seq.map (fun g -> g.Count)))

[openSeries; closeSeries; stillOpenSeries]
|> Chart.Plot
|> Chart.WithTitle("Issue report for the current year")
```

### How many times has dotnet/fsharp been forked?

```fsharp
let forks =
    async {
        return!
            gitHubClient.Repository.Forks.GetAll(organization, repositoryName)
            |> Async.AwaitTask
    } |> Async.RunSynchronously
```

```fsharp
let forksCreatedByMonth =
    forks
    |> Seq.groupBy (fun f -> DateTime(f.CreatedAt.Year, f.CreatedAt.Month,  f.CreatedAt.Day))
    |> Seq.map (fun (key, issues) -> {| Date = key; Count = issues.Count() |})
    |> Seq.sortBy (fun g -> g.Date)

let forksLastUpdateByMonth =
    forks
    |> Seq.groupBy (fun f -> DateTime(f.UpdatedAt.Year, f.UpdatedAt.Month,  f.UpdatedAt.Day))
    |> Seq.map (fun (key, issues) -> {| Date = key; Count = issues.Count() |})
    |> Seq.sortBy (fun g -> g.Date)

let forkCountByMonth =
    forksCreatedByMonth
    |> Seq.sortBy (fun g -> g.Date)
    |> Seq.map (fun g -> {| Date = g.Date; Count = g.Count |})

let forkCreationSeries =
    Graph.Scattergl(
        name = "created by month",
        x = (forksCreatedByMonth |> Seq.map (fun g -> g.Date)),
        y = (forksCreatedByMonth |> Seq.map (fun g -> g.Count)))

let forkTotalSeries =
    Graph.Scattergl(
        name = "running total",
        x = (forkCountByMonth |> Seq.map (fun g -> g.Date)),
        y = (forkCountByMonth |> Seq.map (fun g -> g.Count)))

let forkUpdateSeries =
    Graph.Scattergl(
        name = "last updated by month",
        x = (forksLastUpdateByMonth |> Seq.map (fun g -> g.Date)),
        y = (forksLastUpdateByMonth |> Seq.map (fun g -> g.Count)))

[forkCreationSeries; forkTotalSeries; forkUpdateSeries]
|> Chart.Plot
|> Chart.WithTitle("Fork activity")
```
