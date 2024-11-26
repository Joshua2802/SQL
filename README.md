SELECT *
FROM Project1.dbo.CovidDeaths
WHERE continent IS NOT NULL 
ORDER BY 3,4

SELECT Location, date, total_cases, new_cases, total_deaths, population
FROM Project1.dbo.CovidDeaths
WHERE continent IS NOT NULL 
ORDER BY 1,2


---- Total Cases vs Total Deaths
---- Likelihood of dying if you come in contact with covid in your country

SELECT Location, date, total_cases,total_deaths, CAST(total_deaths*100 AS FLOAT)/CAST(total_cases AS FLOAT) AS Death_Percentage
FROM Project1.dbo.CovidDeaths
WHERE location='India'
AND total_deaths IS NOT NULL 
ORDER BY 1,2


---- Total Cases vs Population
---- Percentage of population infected with Covid

SELECT Location, date, Population, total_cases, CAST(total_cases*100 AS FLOAT)/CAST(population AS FLOAT) AS Percent_Population_Infected
FROM Project1.dbo.CovidDeaths
WHERE location='India'
ORDER BY 1,2


---- Countries with Highest Infection Rate compared to Population

SELECT Location, Population, MAX(total_cases) AS HighestInfectionCount
, MAX(CAST(total_cases AS FLOAT)/CAST(population AS FLOAT) * 100) AS Percent_Population_Infected
FROM Project1.dbo.CovidDeaths
Where continent IS NOT NULL
GROUP BY Location, population
ORDER BY Percent_Population_Infected DESC


---- Countries with Highest Death Count per Population

SELECT Location, MAX(CAST(Total_deaths AS INT)) AS Total_Death_Count
FROM Project1.dbo.CovidDeaths
---WHERE location='India'
WHERE continent IS NOT NULL 
GROUP BY Location
ORDER BY Total_Death_Count DESC


---- Showing contintents with the highest death count per population

SELECT continent, MAX(CAST(Total_deaths AS INT)) AS Total_Death_Count
FROM Project1.dbo.CovidDeaths
--WHERE location='India'
WHERE continent IS NOT NULL 
GROUP BY continent
ORDER BY Total_Death_Count DESC



---- GLOBAL NUMBERS

SELECT SUM(new_cases) AS total_cases, SUM(CAST(new_deaths AS INT)) AS total_deaths, SUM(CAST(new_deaths*100 AS FLOAT))/SUM(New_Cases) AS DeathPercentage
FROM Project1.dbo.CovidDeaths
WHERE continent IS NOT NULL 
--GROUP BY date
ORDER BY 1,2



---- Total Population vs Vaccinations
---- Shows Percentage of Population that has recieved at least one Covid Vaccine

SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(INT,vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
FROM Project1.dbo.CovidDeaths dea
JOIN Project1.dbo.CovidVaccinations vac
	ON dea.location = vac.location
	AND dea.date = vac.date
WHERE dea.continent IS NOT NULL 
ORDER BY 2,3


---- Using CTE to perform Calculation on Partition By in previous query

WITH PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
AS
(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(INT,vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM Project1.dbo.CovidDeaths dea
JOIN Project1.dbo.CovidVaccinations vac
	ON dea.location = vac.location
	AND dea.date = vac.date
WHERE dea.continent IS NOT NULL 
--ORDER BY 2,3
)
SELECT *, CAST(RollingPeopleVaccinated AS FLOAT)/CAST(Population AS FLOAT)*100 AS RPV_Perctage
FROM PopvsVac



---- Using Temp Table to perform Calculation on Partition By in previous query

DROP TABLE IF EXISTS #PercentPopulationVaccinated
CREATE TABLE #PercentPopulationVaccinated
(
Continent NVARCHAR(255),
Location NVARCHAR(255),
Date DATETIME,
Population NUMERIC,
New_vaccinations NUMERIC,
RollingPeopleVaccinated NUMERIC
)

INSERT INTO #PercentPopulationVaccinated
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(INT,vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
FROM Project1.dbo.CovidDeaths dea
JOIN Project1.dbo.CovidVaccinations vac
	ON dea.location = vac.location
	AND dea.date = vac.date
--WHERE dea.continent IS NOT NULL 
--ORDER BY 2,3

Select *, (RollingPeopleVaccinated/Population)*100
From #PercentPopulationVaccinated




---- Creating View to store data for visualizations

--CREATE VIEW PercentPopulationVaccinated AS
--SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
--, SUM(CONVERT(INT,vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
----, (RollingPeopleVaccinated/population)*100
--FROM Project1.dbo.CovidDeaths dea
--JOIN Project1.dbo.CovidVaccinations vac
--	ON dea.location = vac.location
--	AND dea.date = vac.date
--WHERE dea.continent IS NOT NULL
