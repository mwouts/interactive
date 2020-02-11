---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.3.3+dev
  kernelspec:
    display_name: .NET (C#)
    language: C#
    name: .net-csharp
---

# dotnet/interactive Github Dashboard <img src ="https://upload.wikimedia.org/wikipedia/commons/thumb/3/38/Jupyter_logo.svg/207px-Jupyter_logo.svg.png" width="60px" alt="dotnet bot in space" align ="right">


### Add NuGet package references 

```csharp
#r "nuget:Octokit, 0.32.0"
#r "nuget:NodaTime, 2.4.6"
using Octokit;
using NodaTime;
using NodaTime.Extensions;
using XPlot.Plotly;
```

### Setup
Create a GitHub public API client 

```csharp
var organization = "dotnet";
var repositoryName = "interactive";
var options = new ApiOptions();
var gitHubClient = new GitHubClient(new ProductHeaderValue("notebook"));

```

[Generate a user token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) to get rid of public [api](https://github.com/octokit/octokit.net/blob/master/docs/getting-started.md) throttling policies for anonymous users 

```csharp
// var tokenAuth = new Credentials("YOUR-CREDENTIALS-HERE");
// gitHubClient.Credentials = tokenAuth;
```

Create checkpoints for the:  current day(today), the start of the current month and, the start of the current year. 

```csharp
var today = SystemClock.Instance.InUtc().GetCurrentDate();
var startOfTheMonth = today.With(DateAdjusters.StartOfMonth);
var startOfTheYear = new LocalDate(today.Year, 1, 1).AtMidnight();
```

Query GitHub for : 
- Issues created this month
- Issues closed this month
- Every issue this year

```csharp
var createdIssuesRequest = new RepositoryIssueRequest
            {
                Since = startOfTheMonth.ToDateTimeUnspecified(),
                Filter = IssueFilter.Created
            };
var closedIssuesRequest = new RepositoryIssueRequest
            {
                Since = startOfTheMonth.ToDateTimeUnspecified(),
                State = ItemStateFilter.Closed
            };
var thisYearIssuesRequest = new RepositoryIssueRequest
            {
                State = ItemStateFilter.All,
                Since = startOfTheYear.ToDateTimeUnspecified()
            };
```

Start pulling data via the GitHub API

```csharp
var createdThisMonth = await gitHubClient.Issue.GetAllForRepository(organization, repositoryName, createdIssuesRequest);
var closedThisMonth = await gitHubClient.Issue.GetAllForRepository(organization, repositoryName, closedIssuesRequest);
var thisYearIssues = await gitHubClient.Issue.GetAllForRepository(organization, repositoryName, thisYearIssuesRequest);
```

Group open and closed issues by month 

```csharp
var openSoFar = createdThisMonth.OrderBy(i => i.CreatedAt).Where(i => i.State.StringValue == "open").ToArray();
var openByMonthOfCreation = openSoFar.GroupBy(i => new { i.CreatedAt.Year, i.CreatedAt.Month})
                .Select(g => new {Date = g.Key, Count = g.Count()}).ToArray();

var closedSoFar = thisYearIssues.OrderBy(i => i.CreatedAt).Where(i => i.State.StringValue == "closed").ToArray();
var closedByMonthOfClosure = closedSoFar.GroupBy(i => new { i.ClosedAt.Value.Year, i.ClosedAt.Value.Month})
                .Select(g => new {Date = g.Key, Count = g.Count()}).ToArray();
var totalOpenIssues = thisYearIssues.Count();
var openCountByMonth = closedSoFar.GroupBy(i => new { i.CreatedAt.Year, i.CreatedAt.Month})
                .Select(g => {
                    var count = g.Count();                    
                    var dataPoint = new  {Date = g.Key, Count = totalOpenIssues};
                    totalOpenIssues -= count;
                    return dataPoint;
                }).ToArray();
```

Show issues opened this month grouped by day 

```csharp
var createdThisMonthByDay = createdThisMonth.GroupBy(i => new DateTime(i.CreatedAt.Year,i.CreatedAt.Month, i.CreatedAt.Day)); 
var lineChart = Chart.Line(createdThisMonthByDay.Select(g => new Tuple<DateTime,int>(g.Key, g.Count())));
lineChart.WithTitle("Daily Issues");
display(lineChart);
```

```csharp
display(openSoFar.Select(i => new {i.CreatedAt, i.Title, State = i.State.StringValue,  i.Number}).OrderByDescending(d => d.CreatedAt));
```

```csharp
var lineChart = Chart.Line(openByMonthOfCreation.Select(g => new Tuple<DateTime, int>(new DateTime(g.Date.Year, g.Date.Month, 1),g.Count)));
lineChart.WithTitle("Issues still opened grouped by month");
display(lineChart);
```

```csharp
var idleByMonth = openSoFar.Where( i => i.Comments == 0).GroupBy(i => new DateTime( i.CreatedAt.Year, i.CreatedAt.Month, 1))
    .Select(g => new {Date = g.Key, Count = g.Count()}).ToArray();
var activeByMonth = openSoFar.Where( i => i.Comments > 0).GroupBy(i => new DateTime( i.CreatedAt.Year, i.CreatedAt.Month, 1))
                .Select(g => new {Date = g.Key, Count = g.Count()}).ToArray();

var idleSeries = new Graph.Scattergl
{
    name = "Idle",
    y = idleByMonth.Select(g => g.Count ).ToArray(),
    x = idleByMonth.Select(g => g.Date ).ToArray()
};

var activeSeries = new Graph.Scattergl
{
    name = "Active",
    y = activeByMonth.Select(g => g.Count ).ToArray(),
    x = activeByMonth.Select(g => g.Date ).ToArray()
};


var chart = Chart.Plot(new[] {idleSeries, activeSeries});
chart.WithTitle("Idle and active open issue report");
display(chart);
```

```csharp
var openDataPoints = openByMonthOfCreation
    .Select(g => new { y = g.Count, x = new DateTime(g.Date.Year, g.Date.Month, 1)} )
    .OrderBy(d => d.x).ToArray();


var closedDataPoints = closedByMonthOfClosure
    .Select(g => new { y = g.Count, x = new DateTime(g.Date.Year, g.Date.Month, 1)} )
    .OrderBy(d => d.x).ToArray();

var openCountByMonthDataPoints = openCountByMonth
    .Select(g => new { y = g.Count, x = new DateTime(g.Date.Year, g.Date.Month, 1)} )
    .OrderBy(d => d.x).ToArray();

var openSeries = new Graph.Scattergl
{
    name = "Created",
    y = openDataPoints.Select(g => g.y ).ToArray(),
    x = openDataPoints.Select(g => g.x ).ToArray()
};

var closeSeries = new Graph.Scattergl
{
    name = "Closed",
    y = closedDataPoints.Select(g => g.y ).ToArray(),
    x = closedDataPoints.Select(g => g.x ).ToArray()
};

var stillOpenSeries = new Graph.Scattergl
{
    name = "Open",
    y = openCountByMonthDataPoints.Select(g => g.y ).ToArray(),
    x = openCountByMonthDataPoints.Select(g => g.x ).ToArray()
};


var chart = Chart.Plot(new[] {openSeries, closeSeries, stillOpenSeries});
chart.WithTitle("Issue report for the current year");
display(chart);
```

### How many times has dotnet/interactive been forked ?

```csharp
var forks = await gitHubClient.Repository.Forks.GetAll(organization, repositoryName);
```

```csharp
var forkCreateByMonth = forks.GroupBy(f => new DateTime(f.CreatedAt.Year, f.CreatedAt.Month,  f.CreatedAt.Day) )
    .Select(g => new {Date = g.Key, Count = g.Count()}).OrderBy(g => g.Date).ToArray();
var forkLastUpdateByMonth = forks.GroupBy(f => new DateTime(f.UpdatedAt.Year, f.UpdatedAt.Month,  f.UpdatedAt.Day) )
    .Select(g => new {Date = g.Key, Count = g.Count()}).OrderBy(g => g.Date).ToArray();

var total = 0;
var forkCountByMonth = forkCreateByMonth.OrderBy(g => g.Date).Select(g => new {g.Date, Count = total += g.Count}).ToArray();

var forkCreationSeries = new Graph.Scattergl
{
    name = "created by month",
    y = forkCreateByMonth.Select(g => g.Count ).ToArray(),
    x = forkCreateByMonth.Select(g => g.Date ).ToArray()
};

var forkTotalSeries = new Graph.Scattergl
{
    name = "running total",
    y = forkCountByMonth.Select(g => g.Count ).ToArray(),
    x = forkCountByMonth.Select(g => g.Date ).ToArray()
};

var forkUpdateSeries = new Graph.Scattergl
{
    name = "last update by month",
    y = forkLastUpdateByMonth.Select(g => g.Count ).ToArray(),
    x = forkLastUpdateByMonth.Select(g => g.Date ).ToArray()
};



var chart = Chart.Plot(new[] {forkCreationSeries,forkTotalSeries,forkUpdateSeries});
chart.WithTitle("Fork activity");
display(chart);
```
