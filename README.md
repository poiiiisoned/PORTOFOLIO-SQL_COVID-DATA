# PORTOFOLIO-SQL_COVID-DATA
FIRST GUIDED PORTOFOLIO OF DATA MINING ON SQL
-- to join data from CovidDeaths and Vaksin, join on the location and date
SELECT *
FROM vaksin as vak
JOIN CovidDeaths as cov
	ON vak.location= cov.location
	and vak.date = cov.date

-- to see how  many total population and total vaccinated
SELECT  vak.location,max(cov.population) as populasi, max(people_vaccinated) as divaksin
FROM vaksin as vak
JOIN CovidDeaths as cov
	ON vak.location= cov.location
	and vak.date = cov.date
where vak.continent is not null 
group by vak.location
order by divaksin

-- to see how many people get vaccinated daily 
SELECT  cov.continent, cov.location, cov. population,cov.date, vak.new_vaccinations, 
sum(cast(vak.new_vaccinations as bigint)) over (partition by cov.location order by cov.location, cov.date ROWS UNBOUNDED PRECEDING) as pertambahanVaksin
FROM vaksin as vak
JOIN CovidDeaths as cov
	ON vak.location= cov.location
	and vak.date = cov.date
where cov.continent is not null

-- to see how  many people get vaccinated (percentages) with cte
with cte_JumlahVaksinasi(continent,location,date,population,new_vaccinations,pertambahanVaksin)
as  
(SELECT  cov.continent, cov.location, cov.date,cov.population, vak.new_vaccinations, 
sum(cast(vak.new_vaccinations as bigint)) over (partition by cov.location order by cov.location, cov.date ROWS UNBOUNDED PRECEDING) as pertambahanVaksin
FROM vaksin as vak
JOIN CovidDeaths as cov
	ON vak.location= cov.location
	and vak.date = cov.date
where cov.continent is not null
)
select*, (pertambahanVaksin/population)*100 as persentase
from cte_JumlahVaksinasi

-- to see how  many people get vaccinated (percentages) with tempatable
drop table if exists #temp_PopvsVac
create table #temp_PopvsVac(
continent varchar(255),
location varchar(255),
date datetime,
population numeric,
new_vaccinated numeric,
pertambahanVaksin numeric
)
insert into #temp_PopvsVac
SELECT  cov.continent, cov.location, cov.date,cov.population, vak.new_vaccinations, 
sum(cast(vak.new_vaccinations as bigint)) over (partition by cov.location order by cov.location, cov.date ROWS UNBOUNDED PRECEDING) as pertambahanVaksin
FROM vaksin as vak
JOIN CovidDeaths as cov
	ON vak.location= cov.location
	and vak.date = cov.date
where cov.continent is not null

select location,population,new_vaccinated,pertambahanVaksin, (pertambahanVaksin/population)*100 as persentase
from #temp_PopvsVac

--cretate view for data visualisation
create view PPV AS
SELECT  cov.continent, cov.location, cov.date,cov.population, vak.new_vaccinations, 
sum(cast(vak.new_vaccinations as bigint)) over (partition by cov.location order by cov.location, cov.date ROWS UNBOUNDED PRECEDING) as pertambahanVaksin
FROM vaksin as vak
JOIN CovidDeaths as cov
	ON vak.location= cov.location
	and vak.date = cov.date
where cov.continent is not null

--TO SHOW THE STORED DATA IN NEWEST VIEW
SELECT *
FROM PPV
