# Covid-19-dataexploration-
Data Exploration of covid-19 Dataset in PostgreSQL serevr
----(1) WE LOOKING AT TOTAL CASES VS TOTAL DEATHS in "Nigeria" 
SELECT LOCATION, DATE, TOTAL_CASES, TOTAL_DEATHS, POPULATION, (TOTAL_DEATHS/TOTAL_CASES)* 100 as DEATHPERCENTAGE
FROM COVID.DEATHS 
WHERE LOCATION = 'Nigeria'
ORDER BY 1,2
--this shows the likelihood of dying if you contract covid in "Nigeria"
 

-----(2) LOOKING AT THE TOTAL CASES VS THE POPULATION
SELECT LOCATION, DATE, TOTAL_DEATHS, POPULATION, TOTAL_CASES,(TOTAL_CASES/POPULATION)* 100 as PERCENTAGEPOPULATIONINFECTED
FROM COVID.DEATHS 
WHERE LOCATION = 'Nigeria'
ORDER BY 1,2
-- SHOWS WHAT PERCENTAGE OF POPULATION GOT COVID


----(3) COUNTRIES WITH THE HIGHEST INFECTION RATE PERPOPULATION   
SELECT LOCATION,POPULATION, MAX(TOTAL_CASES)as highestinfectioncount,MAX((TOTAL_CASES/POPULATION))* 100 as PERCENTAGEPOPULATIONINFECTED
FROM COVID.DEATHS 
GROUP BY LOCATION,POPULATION
ORDER BY PERCENTAGEPOPULATIONINFECTED desc


----(4) COUNTRY WITH THE HIGHEST TOTAL_DEATH
SELECT LOCATION, MAX(TOTAL_DEATHS)as highestdeathscount
FROM COVID.DEATHS
where continent is not null and total_deaths is not null
GROUP BY LOCATION
ORDER BY 2 desc


----(5) SHOWING CONTINENTS WITH HIGHEST DEATH 
SELECT LOCATION, MAX(TOTAL_DEATHS)as highestdeathscount
FROM COVID.DEATHS
where continent is null
GROUP BY LOCATION
ORDER BY 1 


----(6) SUM OF NEWCASES AND NEWDEATHS PER DAY
SELECT DATE, SUM(NEW_CASES), SUM(NEW_DEATHS)
FROM COVID.DEATHS 
WHERE CONTINENT IS NOT NULL
GROUP BY DATE
ORDER BY 1,2


----(7) HERE WE WANT % OF NEW DEATHS AND NEW CASES 
SELECT DATE, SUM(NEW_CASES) AS TOTALCASES, SUM(NEW_DEATHS) AS TOTALDEATHS,  SUM(NEW_DEATHS)/SUM(NEW_CASES) * 100 as Deathpercentage 
FROM COVID.DEATHS 
WHERE CONTINENT IS NOT NULL
GROUP BY DATE
ORDER BY 1,2


---(8) WE WANT TO FIND OUT WHAT'S THE TOTAL NUMBER OF PEOPLE IN THE WORLD THAT HAS BEEN VACCINATED 
-- IST THING IS WE JOIN BOTH TABLES 
SELECT * 
FROM COVID.VACCINATION V 
JOIN COVID.DEATHS D 
ON 
V.LOCATION = D.LOCATION AND V.DATE = D.DATE 


------ LOOKING AT TOTAL POPULATION VS VACCINATION --WE'D USE THE RUNNING TOTAL CONCEPT HERE 
SELECT D.CONTINENT, D.LOCATION, D.DATE, D.POPULATION ,V.NEW_VACCINATIONS, SUM(V.NEW_VACCINATIONS) OVER (PARTITION BY D.LOCATION ORDER BY D.LOCATION,D.DATE) AS ROLLINGPEOPLEVACCINATED
FROM COVID.DEATHS D JOIN COVID.VACCINATION V 
ON 
V.LOCATION = D.LOCATION AND V.DATE = D.DATE 
WHERE D.CONTINENT IS NOT NULL
ORDER BY 2,3

-- NOW WE NEED TO USE THAT NEWLY CREATED COLUMN ALIASED "ROLLINGPEOPLEVACCINATED" TO PERFORM AN OPS((ROLLINGPEOPLEVACCINATED/POPULATION)*100)
USING A TEMP TABLE (TEMPORARY TABLE)
CREATE TABLE PERCENTPOPULATIONVACCINATED
(CONTINENT VARCHAR(100),
LOCATION VARCHAR(100),
DATE DATE,
POPULATION REAL,
NEW_VACCINATIONS REAL,
ROLLINGPEOPLEVACCINATED REAL)

INSERT INTO PERCENTPOPULATIONVACCINATED
SELECT D.CONTINENT, D.LOCATION, D.DATE, D.POPULATION ,V.NEW_VACCINATIONS, SUM(V.NEW_VACCINATIONS) OVER (PARTITION BY D.LOCATION ORDER BY D.LOCATION,D.DATE) AS ROLLINGPEOPLEVACCINATED
FROM COVID.DEATHS D JOIN COVID.VACCINATION V 
ON 
V.LOCATION = D.LOCATION AND V.DATE = D.DATE 

--- THEN (ROLLINGPEOPLEVACCINATED/POPULATION)*100 
select *, (ROLLINGPEOPLEVACCINATED/POPULATION)*100 as new
from PERCENTPOPULATIONVACCINATED

--- CREATE A VIEW TO STORE DATA FOR LATER VISUALIZATION 
CREATE VIEW COVID.PERCENTPOPULATIONVACCINATED
AS 
SELECT D.CONTINENT, D.LOCATION, D.DATE, D.POPULATION ,V.NEW_VACCINATIONS, SUM(V.NEW_VACCINATIONS) OVER (PARTITION BY D.LOCATION ORDER BY D.LOCATION,D.DATE) AS ROLLINGPEOPLEVACCINATED
FROM COVID.DEATHS D JOIN COVID.VACCINATION V 
ON 
V.LOCATION = D.LOCATION AND V.DATE = D.DATE 
WHERE D.CONTINENT IS NOT NULL

SELECT * FROM COVID.PERCENTPOPULATIONVACCINATED
