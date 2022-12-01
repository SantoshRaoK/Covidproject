# Covidproject
#Full Analysis of COVID-19 using SQL Server

SELECT *
FROM COVIDProject..COVIDDeaths
WHERE Continent IS NOT NULL
ORDER BY 3,4

--SELECT *
--FROM COVIDProject..COVIDVaccinations
--ORDER BY 3,4

-- Select Data that we are going to be using

SELECT location, date, total_cases, new_cases, total_deaths, population
FROM COVIDProject..COVIDDeaths
WHERE Continent IS NOT NULL
ORDER BY 1,2 


-- Looking at TOTAL CASES vs TOTAL DEATHS
-- Shows Likehood of dying if you contract COVID in your country

SELECT location, date, total_cases, total_deaths, (total_deaths / total_cases)*100 AS DeathPercentage
FROM COVIDProject..COVIDDeaths
WHERE Location Like '%States'
AND Continent IS NOT NULL
ORDER BY 1,2

-- Looking at TOTAL CASES vs POPULATION
-- Shows what percentage of population got COVID

SELECT location, date, population, total_cases,  (total_cases / population)*100 AS PercentPopulationInfected
FROM COVIDProject..COVIDDeaths
-- WHERE Location LIKE '%States'
ORDER BY 1,2

-- Looking at Countries With Highest Infection Rate Compared to Population

SELECT location, population, MAX(total_cases) AS HighestInfectionCount,  MAX((total_cases / population))*100 AS PercentPopulationInfected
FROM COVIDProject..COVIDDeaths
-- WHERE Location LIKE '%States'
GROUP BY population, location
ORDER BY PercentPopulationInfected DESC

-- Showing countries with the highest death count per population
SELECT location, MAX(cast(total_deaths as int)) AS TotalDeathCount
FROM COVIDProject..COVIDDeaths
-- WHERE Location LIKE '%States'
WHERE Continent IS NOT NULL
GROUP BY population, location
ORDER BY TotalDeathCount DESC

-- LET'S BREAK THING DOWN BY CONTINENT

-- Showing continents with the highest Death Count per population

SELECT continent, MAX(cast(total_deaths as int)) AS TotalDeathCount
FROM COVIDProject..COVIDDeaths
-- WHERE Location LIKE '%States'
WHERE Continent IS NOT NULL
GROUP BY continent
ORDER BY TotalDeathCount DESC


-- GLOBAL NUMBERS

SELECT SUM (new_cases) as TotalCases, SUM (cast (New_deaths as int)) AS TotalDeaths, 
SUM (cast(new_deaths as int)) / SUM (new_cases) * 100 AS DeathPercentage
FROM COVIDProject..COVIDDeaths
--WHERE Location Like '%States'
WHERE Continent IS NOT NULL
--GROUP BY date
ORDER BY 1,2


-- Looking at TotalPopulation VS Vaccination

SELECT DEA.continent, DEA.location, DEA.date, DEA.population, VAC.new_vaccinations
, SUM(Cast(VAC.new_vaccinations AS bigint)) OVER (Partition By DEA.location ORDER BY DEA.location, 
DEA.date) AS RollingPeopleVaccinated
--, (RollingPeopleVaccinated / Population) * 100
FROM COVIDProject..COVIDDeaths DEA 
JOIN COVIDProject..COVIDVaccinations VAC
ON DEA.location = VAC.location
AND DEA.date = VAC.date
WHERE DEA.continent IS NOT NULL
ORDER BY 2,3 

-- USE CTE

WITH PopvsVac (Continent, Location, Date, Population, NewVaccinations, RollingPeopleVaccinated)
AS
(
SELECT DEA.continent, DEA.location, DEA.date, DEA.population, VAC.new_vaccinations
, SUM(Cast(VAC.new_vaccinations AS bigint)) OVER (Partition By DEA.location ORDER BY DEA.location, 
DEA.date) AS RollingPeopleVaccinated
--, (RollingPeopleVaccinated / Population) * 100
FROM COVIDProject..COVIDDeaths DEA 
JOIN COVIDProject..COVIDVaccinations VAC
ON DEA.location = VAC.location
AND DEA.date = VAC.date
WHERE DEA.continent IS NOT NULL
--ORDER BY 2,3 
)
SELECT * , (RollingPeopleVaccinated /Population) * 100
FROM PopvsVac


-- TEMP TABLE

DROP TABLE PercentPopulationVaccinated IF EXISTS
CREATE TABLE PercentPopulationVaccinated
(
Continent nvarchar (255),
Location nvarchar (255),
Date datetime, 
Population numeric,
NewVaccinations numeric, 
Rollingpeoplevaccinated numeric)


INSERT INTO PercentPopulationVaccinated
SELECT DEA.continent, DEA.location, DEA.date, DEA.population, VAC.new_vaccinations
, SUM(Cast(VAC.new_vaccinations AS bigint)) OVER (Partition By DEA.location ORDER BY DEA.location, 
DEA.date) AS RollingPeopleVaccinated
--, (RollingPeopleVaccinated / Population) * 100
FROM COVIDProject..COVIDDeaths DEA 
JOIN COVIDProject..COVIDVaccinations VAC
ON DEA.location = VAC.location
AND DEA.date = VAC.date
WHERE DEA.continent IS NOT NULL
--ORDER BY 2,3 

SELECT * , (RollingPeopleVaccinated /Population) * 100
FROM PercentPopulationVaccinated

- Creating View to store data for later visualizations

Create View PercentPopulationVaccinated AS
SELECT DEA.continent, DEA.location, DEA.date, DEA.population, VAC.new_vaccinations
, SUM(Cast(VAC.new_vaccinations AS bigint)) OVER (Partition By DEA.location ORDER BY DEA.location, 
DEA.date) AS RollingPeopleVaccinated
--, (RollingPeopleVaccinated / Population) * 100
FROM COVIDProject..COVIDDeaths DEA 
JOIN COVIDProject..COVIDVaccinations VAC
ON DEA.location = VAC.location
AND DEA.date = VAC.date
WHERE DEA.continent IS NOT NULL
--ORDER BY 2,3 


SELECT * 
FROM PercentPopulationVaccinated
