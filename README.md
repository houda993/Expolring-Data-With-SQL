# project 1
#data analytics portfolio
#guided by Alex The Analyst
#exploring covid 19 data and visualize it in Tableau 

Select *
From covid_deaths
Where continent is not null 
order by 3,4

-- Select Data that we are going to be starting with

Select Location, date, total_cases, new_cases, total_deaths, population
From covid_deaths
Where continent is not null 
order by 1,2


-- Total Cases vs Total Deaths
-- Shows likelihood of dying with covid in uae

Select Location, date, total_cases,total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From covid_deaths
Where location like '%emirates%'
and continent is not null
order by 1,2


-- Total Cases vs Population
-- Shows what percentage of population infected with Covid 

Select Location, date, Population, total_cases,(total_cases/population)*100 as PercentPopulationInfected
From covid_deaths
--Where location like '%emirates%'
order by 1,2 

-- looking at countries with highest ifection rate compared to population
Select location, population, MAX(total_cases) as maxtotalcases, MAX((total_cases/population))*100 as PercentPopulationInfected 
from covid_deaths
Where continent is not null 
group by location, population
order by PercentPopulationInfected desc

-- shwoing countries with highest death rates 

Select location, MAX(cast(total_deaths as int)) as maxtotaldeathes 
from covid_deaths
Where continent is not null 
group by location
order by maxtotaldeathes desc

-- showing continents with the highest death rates 
Select continent, MAX(cast(total_deaths as int)) as maxtotaldeathes 
from covid_deaths
Where continent is not null 
group by continent
order by maxtotaldeathes desc

-- global numbers 
-- used CAST to convert varchar to int to get the calculation right

Select sum(new_cases) as total_cases, sum(cast(new_deaths as int)) as total_deaths,
sum(cast(new_deaths as int))/sum(new_cases) * 100 as DeathPercentage
From covid_deaths
where continent is not null
--group by date
--order by 1,2

--- looking at total population vs vaccination 
-- by joining the tables
-- CTE 

With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From covid_deaths dea
Join covid_vaccination vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
)
Select *, (RollingPeopleVaccinated/Population)*100
From PopvsVac

-- temp table 
drop table if exists #percentpopulationvaccinated
create table #percentpopulationvaccinated
(
continent nvarchar(255),
location nvarchar(255),
date datetime,
population numeric, 
new_vaccinations numeric,
rollingpeoplevaccinated numeric
)
insert into #percentpopulationvaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From covid_deaths dea
Join covid_vaccination vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null

Select *, (RollingPeopleVaccinated/Population)*100
From #percentpopulationvaccinated


-- Creating View to store data for visualizations

Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
From covid_deaths dea
Join covid_vaccination vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 

select * from PercentPopulationVaccinated
