# Portfolio_Project-Indian-Census
--Inspecting Data
select* from Data1
select* from Data2

--no. of rows in our dataset
select count(*) from Data1
select count(*) from Data2

--dataset for Andhra Pradesh  & Rajasthan

select * from data1 where state in ('Andhra Pradesh', 'Rajasthan')

-- population of India

Select sum (population) as TotalPopulation from Data2

--avg growth 
select avg(growth) from Data1

--avg growth %

select avg(growth)*100 avg_growth from Data1
select state, avg(growth)*100 avg_growth from data1 group by state
select state, avg(growth)*100 avg_growth from Data1 group by state order by avg_growth desc

--avg sex ratio of the states

select state, round(avg(Sex_Ratio),0)sex_ratio from data1 group by state
select state, round(avg(sex_ratio),0) sex_ratio from data1 group by state order by sex_ratio desc

--avg literacy rate of the states

select state , avg(literacy) avg_literacy from data1 group by state order by avg_literacy desc
select state, round(avg (literacy),0) avg_literacy from data1 group by state order by avg_literacy desc
select state, round(avg(literacy),0) avg_literacy from data1 group by state having round(avg(literacy),0)>0 order by avg_literacy desc 

--top 3 states showing highest growth 

select top 3 state, avg(growth)*100 avg_growth from Data1 group by state order by avg_growth desc 

--bottom 3 states showing lowest growth

select top 3 state, avg(growth)*100 avg_growth from Data1 group by state order by avg_growth asc

--top 3 states showing highest sex ratio

select top 3 state, round(avg(sex_ratio),0)*100 avg_growth from Data1 group by state order by avg_growth desc

--bottom 3 states showing lowest sex ratio 
select top 3 state,round(avg(sex_ratio),0)*100 avg_growth from Data1 group by state order by avg_growth asc

-- Comparing top and bottom 3 states with lowest& highest sex ratio by creating temporary tables

create table #topstate (state varchar(255),topstates float)
Insert into #topstate select state, round(avg (literacy),0) avg_literacy from data1 group by state order by avg_literacy desc

select* from #topstate order by topstates desc

create table #bottomstate (state varchar(255),bottomstates float)
Insert into #bottomstate select state, round(avg (literacy),0) avg_literacy from data1 group by state order by avg_literacy asc

select* from #bottomstate order by bottomstates asc

---Union Operator
select* from (
select top 3 * from #topstate order by topstates desc) a

Union

select* from (
select top 3 * from #bottomstate order by bottomstates asc) b

--joining tables to find number of males & females in a state

select d.state,sum(d.males) total_males,sum(d.females) total_females from
(select c.district,c.state,round(c.population/(c.sex_ratio+1),0) males,round(c.population*(c.sex_ratio) / (c.sex_ratio +1),0) females from 

(select a.district, a.state,a.sex_ratio/1000 sex_ratio,b.population from Data1 a inner join data2 b on a.District = b.district) c) d
group by d.State

--total literacy rate

Select c.state, sum(literate_people) Literate_people,sum(illiterate_people) Illiterate_people from
(select d.district, d.state, round(d.literacy_ratio*d.population,0) literate_people, round ((1-d.literacy_ratio)*d.population,0) illiterate_people from
(select a.district, a.state,a.literacy/100 literacy_ratio,b.population from Data1 a inner join data2 b on a.District = b.district) d) c
 Group by c.State

 --population in the previous Census

 
 select sum(m.previous_census_population) previous_census_population,sum(m.current_census_population) current_census_population from(
select e.state,sum(e.previous_census_population) previous_census_population,sum(e.current_census_population) current_census_population from
(select d.district,d.state,round(d.population/(1+d.growth),0) previous_census_population,d.population current_census_population from
(select a.district,a.state,a.growth growth,b.population from data1 a inner join data2 b on a.district=b.district) d) e
group by e.state)m


-- population vs area

select (g.total_area/g.previous_census_population)  as previous_census_population_vs_area, (g.total_area/g.current_census_population) as 
current_census_population_vs_area from
(select q.*,r.total_area from (

select '1' as keyy,n.* from
(select sum(m.previous_census_population) previous_census_population,sum(m.current_census_population) current_census_population from(
select e.state,sum(e.previous_census_population) previous_census_population,sum(e.current_census_population) current_census_population from
(select d.district,d.state,round(d.population/(1+d.growth),0) previous_census_population,d.population current_census_population from
(select a.district,a.state,a.growth growth,b.population from data1 a inner join data2 b on a.district=b.district) d) e
group by e.state)m) n) q inner join (

select '1' as keyy,z.* from (
select sum(area_km2) total_area from data2)z) r on q.keyy=r.keyy)g

--window 

output top 3 districts from each state with highest literacy rate


select a.* from
(select district,state,literacy,rank() over(partition by state order by literacy desc) rnk from data1) a

where a.rnk in (1,2,3) order by state
