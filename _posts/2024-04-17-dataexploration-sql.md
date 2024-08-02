---
layout: post
title:  Data Exploration in SQL
categories: [SQL,Code, Project]
---

Project information: 
+ Language: SQL
+ Data: Covid-19
+ Objective: To explore the data behaviour, gain further insights and to understand the trends and patterns.


References: [AlexTheAnalyst](https://www.youtube.com/watch?v=qfyynHBFOsM&list=PLUaB-1hjhk8H48Pj32z4GZgGWyylqv85f)

Below is the raw snippet of code including the comments.

1) Select all data where the Continent is not empty/missing.
```sql  
select *
from PortfolioProject..CovidDeaths
where continent is not null
order by 3,4

--select *
-- from PortfolioProject..CovidVaccinations
-- order by 3,4
```


2) Select which data that we are going to use
```sql
select Location, date, total_cases, new_cases, total_deaths, population
from PortfolioProject..CovidDeaths
order by 1,2
```

a) Looking at Total Cases versus Total Deaths. This shows the likelihood of people dying if they contract covid in their country
```sql
Select Location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From PortfolioProject..CovidDeaths
where location like '%states%'
order by 1,2
```


b) Looking at Total Cases versus Population. This shows what percentage of population that got covid
```sql
Select Location, date, total_cases, Population, (total_cases/population)*100 as PercentPopulationInfected
From PortfolioProject..CovidDeaths
--where location like '%states%'
order by 1,2
```

c) Looking at countries with the highest infection rate compared to population. 
```sql
Select Location, Population, MAX(total_cases) as highestInfectionCount, MAX((total_cases/population))*100 as PercentPopulationInfected
From PortfolioProject..CovidDeaths
--where location like '%states%'
group by Location, Population
order by PercentPopulationInfected desc
```

d) Showing countries with highest death count per population.
```sql
Select Location, MAX(cast(total_deaths as int)) as TotalDeathCount
From PortfolioProject..CovidDeaths
--where location like '%states%'
where continent is not null
group by Location
order by TotalDeathCount desc
```

e) Let's break things down by continent.
```sql
Select continent, MAX(cast(total_deaths as int)) as TotalDeathCount
From PortfolioProject..CovidDeaths
--where location like '%states%'
where continent is not null
group by continent
order by TotalDeathCount desc
```

f) Showing continents with the highest death count per population.
```sql
Select continent, MAX(cast(total_deaths as int)) as TotalDeathCount
From PortfolioProject..CovidDeaths
--where location like '%states%'
where continent is not null
group by continent
order by TotalDeathCount desc
```

g) The global numbers.
```sql
Select date, sum(new_cases) as totalCases, sum(cast(new_deaths as int)) as totalDeaths, sum(cast(new_deaths as int))/sum(new_cases)*100 as DeathPercentage
From PortfolioProject..CovidDeaths
--where location like '%states%'
where continent is not null
group by date
order by 1,2

Select sum(new_cases) as totalCases, sum(cast(new_deaths as int)) as totalDeaths, sum(cast(new_deaths as int))/sum(new_cases)*100 as DeathPercentage
From PortfolioProject..CovidDeaths
--where location like '%states%'
where continent is not null
--group by date
order by 1,2
```

h) Check table named 'CovidVaccinations'
```sql
select *
from PortfolioProject..CovidVaccinations
```

i) Join two tables called 'CovidDeaths' and 'CovidVaccinations' based on location and date.
```sql
select *
from PortfolioProject..CovidDeaths dea
join PortfolioProject..CovidVaccinations vac
on dea.location = vac.location
and dea.date = vac.date
```

j) Use cte
```sql
with popsVsVac(continent, location, date, population, new_vaccinations, rollingPeopleVaccinated)
as
(
--looking at total population vs vaccinations
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
sum(convert(int,vac.new_vaccinations)) over (partition by dea.location order by dea.location, dea.date) as rollingPeopleVaccinated
--(rollingPeopleVaccinated/population)*100
from PortfolioProject..CovidDeaths dea
join PortfolioProject..CovidVaccinations vac
on dea.location = vac.location
and dea.date = vac.date
where dea.continent is not null
--order by 2,3
)
select * , (rollingPeopleVaccinated/population)*100
from popsVsVac
```

k) Temporary table
```sql
drop table if exists #percentPopulationVaccinated
create table #percentPopulationVaccinated
(
continent nvarchar(255),
location nvarchar (255),
date datetime,
population numeric,
new_vaccinations numeric,
rollingPeopleVaccinated numeric
)

insert into #percentPopulationVaccinated
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
sum(convert(int,vac.new_vaccinations)) over (partition by dea.location order by dea.location, dea.date) as rollingPeopleVaccinated
--(rollingPeopleVaccinated/population)*100
from PortfolioProject..CovidDeaths dea
join PortfolioProject..CovidVaccinations vac
on dea.location = vac.location
and dea.date = vac.date
--where dea.continent is not null
--order by 2,3

select * , (rollingPeopleVaccinated/population)*100
from #percentPopulationVaccinated
```

l) Create a view to store data for later visualisations
```
--drop table if exists percentPopulationVaccinated
create view percentPopulationVaccinated as
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
sum(convert(int,vac.new_vaccinations)) over (partition by dea.location order by dea.location, dea.date) as rollingPeopleVaccinated
--(rollingPeopleVaccinated/population)*100
from PortfolioProject..CovidDeaths dea
join PortfolioProject..CovidVaccinations vac
on dea.location = vac.location
and dea.date = vac.date
where dea.continent is not null
--order by 2,3
```

m) View the created view
```sql
select *
from percentPopulationVaccinated
```
