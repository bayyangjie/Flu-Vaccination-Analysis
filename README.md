# Content 
- [Project Structure](#project-structure)
- [Objectives](#objectives)
  - [Dashboard Specifications](#dashboard-specifications)
- [Tool used](#tools-used)
- [Dataset](#dataset)
- [Data Preparation](#data-preparation)
- [Data Transformation](#data-transformation)
- [Visualization](#visualization)
  - [Flu Shots by Age](#flu-shots-by-age)
  - [Flu Shots by Race](#flu-shots-by-race)
  - [Flu Shots by Country](#flu-shots-by-country)
  - [Running Sum of Flu Shots](#running-sum-of-flu-shots)
  - [Overall proportion of administered flu shots and vaccinated patients](#overall-proportion-of-administered-flu-shots-and-vaccinated-patients)
- [Conclusion](#conclusion)
- [Suggested Improvements](#suggested-improvements)


# Project Structure
<img src="https://github.com/bayyangjie/Flu-Vaccination-Analysis/blob/main/images/%20tree%20structure.png?raw=true" width="50%">

# Objectives
This analysis aims to present statistical information about patients' flu vaccinations administered in Massachusetts, United States in 2022. 

## Dashboard Specifications
1. The overall percentage of patients receiving flu vaccinations is categorized by:
* Age group
* Racial demographics
* County of residence
* Aggregated data reflecting both the percentage and total count of individuals who have received the flu vaccine

2. Cumulative total of flu vaccinations administered
   
3. Total count of flu vaccinations provided

4. A detailed list of patients and their flu vaccination status

The analysis considers only patients who are still active in the hospital records. Patients who discontinue visits to the hospital are not considered. Conditions are created in the analysis to also distinguish active and inactive patients.

Please click [here](https://public.tableau.com/views/AnalysisofFluVaccinations/Dashboard1?:language=en-GB&:sid=&:display_count=n&:origin=viz_share_link) to view the interactive dashboard on my Tableau Public profile.

# Tools used
* pgAdmin4 - management and interaction with the imported CSV datasets
* Tableau - generating visualizations

# Dataset
The data is generated from Synthea, an open source educational tool which consists of mock patient data simulating an actual healthcare database. The source data comprises a total of 4 datasets.

* Patients: Information about the patients such as their birthdate, deathdate, first and last names, race, city etc.

* Conditions: Information about the patient's conditions that he or she is suffering from. This include information such as the period of time the patient has had the condition for, details of the illness/conditions, patien IDs.

* Encounters: Information about the patient's visits such as description of visit, start and end dates/times of each hospital visit, amount paid for each visit, claimed amount, the patient IDs etc.

* Immunizations: Contains details about the patients' vaccinations such as patient IDs, vaccination date, description of vaccination received.

# Data Preparation
The tables are created in the database and the actual source data prior to loading the actual data. 

```SQL
CREATE TABLE conditions (
START DATE
,STOP DATE
,PATIENT VARCHAR(1000)
,ENCOUNTER VARCHAR(1000)
,CODE VARCHAR(1000)
,DESCRIPTION VARCHAR(200)
);

CREATE TABLE encounters (
 Id VARCHAR(100)
,START TIMESTAMP
,STOP TIMESTAMP
,PATIENT VARCHAR(100)
,ORGANIZATION VARCHAR(100)
,PROVIDER VARCHAR(100)
,PAYER VARCHAR(100)
,ENCOUNTERCLASS VARCHAR(100)
,CODE VARCHAR(100)
,DESCRIPTION VARCHAR(100)
,BASE_ENCOUNTER_COST FLOAT
,TOTAL_CLAIM_COST FLOAT
,PAYER_COVERAGE FLOAT
,REASONCODE VARCHAR(100)
--,REASONDESCRIPTION VARCHAR(100)
);

CREATE TABLE immunizations
(
 DATE TIMESTAMP
,PATIENT varchar(100)
,ENCOUNTER varchar(100)
,CODE int
,DESCRIPTION varchar(500)
--,BASE_COST float
);

CREATE TABLE patients
(
 Id VARCHAR(100)
,BIRTHDATE date
,DEATHDATE date
,SSN VARCHAR(100)
,DRIVERS VARCHAR(100)
,PASSPORT VARCHAR(100)
,PREFIX VARCHAR(100)
,FIRST VARCHAR(100)
,LAST VARCHAR(100)
,SUFFIX VARCHAR(100)
,MAIDEN VARCHAR(100)
,MARITAL VARCHAR(100)
,RACE VARCHAR(100)
,ETHNICITY VARCHAR(100)
,GENDER VARCHAR(100)
,BIRTHPLACE VARCHAR(100)
,ADDRESS VARCHAR(100)
,CITY VARCHAR(100)
,STATE VARCHAR(100)
,COUNTY VARCHAR(100)
,FIPS INT 
,ZIP INT
,LAT float
,LON float
,HEALTHCARE_EXPENSES float
,HEALTHCARE_COVERAGE float
,INCOME int
,Mrn int
);
```

Data is loaded into the tables using the Import Data feature in pgadmin4. Verification is also performed to ensure that the data is correctly imported into each table.

<img src="https://github.com/bayyangjie/Flu-Vaccination-Analysis/blob/main/images/insert_data.png?raw=true" width="100%">

# Data Transformation
CTEs are created while incorporating the use of Subquery and JOINs in the data transformation step. The aim is to encapsulate the complex JOINs & Subqueries into reference objects which are then used to create the final table for use in the Tableau analysis. 

The query belows return patients who are alive and aged 6 months or older.
```
with active_patients as 
(
	select distinct patient 
	from encounters as e
	join patients as pat
		on e.patient = pat.id
	where start between '2020-01-01 00:00' and '2022-12-31 23:59'
		and pat.deathdate is null
		and extract(month from age('2022-12-31',pat.birthdate)) >= 6
)
```

The query below creates another CTE that contains the earliest flu shot dates of each distinct patient.
```
flu_shot_2022 as
(
select patient, min(date) as earliest_flu_shot_2022 
from immunizations
where code = '5302'
	and date between '2022-01-01 00:00' and '2022-12-31 23:59'
group by patient 
)
```
The query below performs a JOIN to merge patient's earliest flu shot dates with their demographic information. CASE WHEN is used to provide numerical indications for differentiating between vaccinated and non-vaccinated patients. The resulting table information is then loaded into Tableau for visualisation analysis.
```
select  pat.birthdate
	   ,pat.race
	   ,pat.county
	   ,pat.id
	   ,pat.first
	   ,pat.last 
	   ,flu.earliest_flu_shot_2022
	   ,flu.patient
	   ,case when flu.patient is not null then 1 
	    else 0
		end as flu_shot_2022
	   ,extract(YEAR FROM age('12-31-2022', birthdate)) as age 
from patients as pat
left join flu_shot_2022 as flu
	on pat.id = flu.patient
where 1=1
	and pat.id in (select patient from active_patients)
```

# Visualization

### Flu Shots by Age
Young patients below the ages of 17 and elderly patients above the age of 50 make up the bulk of patients who had gotten flu vaccinations. Majority of patients in all age groups have also gotten flu vaccinations.

<img src="images/flu shots by age.png" width="100%">

### Flu Shots by Race
All patients who are of Native race and Others are all vaccinated with the flu vaccine. Asians are the next group of patients who form the highest. Overall, majority of patients in each race are vaccinated with the flu vaccine.

<img src="images/flu shots by race.png" width="100%">

### Flu Shots by Country
Out of all the cities in Masachusetts, Nantucket County holds the highest proportion in terms of amount of residents who have gotten flu vaccinations at 87.5% and Hampshire County has the lowest proportion of residents with flu vaccinations at 75.6%.

<img src="images/flu shots by county.png" wdith="100%">

### Running Sum of Flu Shots
Throughout the year, the number of flu vaccinations exhibit a growing upward trend. The steepest increase in flu vaccinations occurreed between the months of February and August. After which, the flu vaccination rate tapered down abit while still showing an increasing trend.

<img src="images/running sum of flu shots.png" width="100%">

### Overall proportion of administered flu shots and vaccinated patients
Out of the entire population of patients in the dataset, approximately 82% of patients have the flu vaccinations. The total number of flu shots administered is also equal to 493 in the year 2022.

<img src="images/overall.png" width="70%">

# Conclusion
The dashboard provides a quick overview of patients' flu vaccination status and this can provide many useful benefits for hospital staff such as doctors,clinicians, nurses. 
* It provides clinicians with a record of the patients vaccination history. That can help aid in medical diagnosis of illnesses or if there are possible side effects from the vaccines.
* This can support resource allocation as well by distributing vaccines where needed
* Vaccination trends can also be tracked over time such as observing seasonal patterns
* Hospitals can also use this dashboard to schedule follow-up reminders to patients for annual flu shots 

# Suggested Improvements
ased on the insights derived from this analysis, forecasting can serve as an additional tool to anticipate flu seasons by examining flu vaccination rates. Predictive forecasting may also be employed to project the number of hospitalizations, medical consultations, and fatalities associated with influenza, contingent upon the levels of flu coverage and activity in specific regions or states. Furthermore, this analysis could be enhanced by integrating real-time data into the dashboard, providing hospital personnel with up-to-date information regarding patient vaccinations as well as their hospital visits. This integration facilitates a more effective distribution of resources to departments that experience a higher volume of patient visits.
