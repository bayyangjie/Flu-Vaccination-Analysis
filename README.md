# Objectives
The dashboard is designed to present comprehensive information regarding flu vaccinations administered in Massachusetts, United States, throughout the year 2022.

1. The overall percentage of patients receiving flu vaccinations, categorized by:
a. Age group
b. Racial demographics
c. County of residence
d. Aggregate data reflecting both the percentage and total count of individuals who have received the flu vaccine

2. Cumulative total of flu vaccinations administered during the year 2022
3. Total count of flu vaccinations provided in 2022
4. A detailed list of patients indicating their vaccination status regarding flu shots.

The above objectives apply only to active patients at the hospital. Some patients might stay far away from the hospital thus making it inconvenient for them to continue further regular visits to the hospital, resulting in their patient status as inactive in the hospital records. Inactive patients are excluded from the analysis. 

# Approach
Using **CREATE TABLE** , four tables are first created in the database while defining the data types of the column variables from the datasets. After which, the tables are populated by importing the datasets. A total of 4 datasets were used for this analysis - Patients, Immunizations, Encounters and Conditions. Verifications are then done to ensure the correct data is being imported into the right table.

PostgreSQL's GUI tool pgadmin4 was used as the tool for managing and interacting with the imported csv datasets. And Tableau was used as the tool for developing the visualisations.

Once the tables are created successfully with the correct data, this is followed by the creation of two CTE tables that are subsequently incorporated together as a subquery filter and in a JOIN statement.

With the finalised joined table containing the new fields created from PostgreSQL, the visualisations created in Tableau then provde a summary of flu shots administered to patients to provide an overview of the distribution of vaccinations. The final dashboard pulls together the visualisations and dynamic filters are also added to allow users to interact with the data.

# About the dataset
The data used in this analysis is synthetically generated from an open source educational tool, Synthea which consists of mock patient data that simulates those from an actual healthcare database.

Patients dataset: Information about the patients such as their birthdate, deathdate, first and last names, race, city etc.

Conditions dataset: Contains information about the patient's conditions that he or she is suffering from. This include information such as the period of time the patient has had the condition for, details of the illness/conditions, patien IDs.

Encounters dataset: Information about the patient's visits such as description of visit, start and end dates/times of each hospital visit, amount paid for each visit, claimed amount, the patient IDs etc.

Immunizations dataset: Contains details about the patients' vaccinations such as patient IDs, vaccination date, description of vaccination received.

# SQL
The query below creates the CTE table 'active_patients' which only pulls patient IDs that satisfy conditions that are stated in the WHERE clause. In this case, the conditions are to only retrieve patients who are still alive, visited the hospital between the start of 2020 and end of 2022 and are 6 months or older.
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

As there could be patients that received more than one flu vaccinations in the year, the CTE query below creates a table that groups the patients by their unique patient IDs and returns records of the earliest vaccination date individually. This query also specifies the vaccine code '5302' for flu vaccines as well as the timeline period between the first and last day of 2022. 
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

A **LEFT JOIN** is used to merge the earliest flu shot dates of the patients with their demograpic information from the 'patients' table, such as their race, age, county location, full names. The **CASE WHEN** clause is used to denote a '1' for patients with a flu vaccination and '0' for patients without a flu vaccination. This helps when performing data manipulation in Tableau to get the number and percentages of patients who got the flu shots vice versa. 

The CTE table 'active_patients' that was created earlier is used a subquery filter with the WHERE clause to return only active, alive and patients who are 6 months or older with the flu vaccinations. 

An 'age' column is also created to list the ages of patients as of the end of 2022.

The finalised table from this JOIN clause is then loaded into Tableau for the creating the visualisations/dashboard.
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

# Tableau Visualisations

## Flu Shots by Age
Young patients below the ages of 17 and elderly patients above the age of 50 make up the bulk of patients who had gotten flu vaccinations. Majority of patients in all age groups have also gotten flu vaccinations.

<img src="images/flu shots by age.png" width="100%">

## Flu Shots by Race
All patients who are of Native race and Others are all vaccinated with the flu vaccine. Asians are the next group of patients who form the highest. Overall, majority of patients in each race are vaccinated with the flu vaccine.

<img src="images/flu shots by race.png" width="100%">

## Flu Shots by County
Out of all the cities in Masachusetts, Nantucket County holds the highest proportion in terms of amount of residents who have gotten flu vaccinations at 87.5% and Hampshire County has the lowest proportion of residents with flu vaccinations at 75.6%.

<img src="images/flu shots by county.png" wdith="100%">

## Running Sum of Flu Shots
Throughout the year, the number of flu vaccinations exhibit a growing upward trend. The steepest increase in flu vaccinations occurreed between the months of February and August. After which, the flu vaccination rate tapered down abit while still showing an increasing trend.

<img src="images/running sum of flu shots.png" width="100%">

# Overall proportion of administered flu shots and vaccinated patients
Out of the entire population of patients in the dataset, approximately 82% of patients have the flu vaccinations. The total number of flu shots administered is also equal to 493 in the year 2022.

<img src="images/overall.png" width="100%">

# Conclusion
The development of this dashboard provides a quick overview of patients' flu vaccination status and this can provide many useful benefits for hospital staff such as doctors,clinicians, nurses. 
* It provides clinicians with a record of the patients vaccination history. That can help aid in medical diagnosis of illnesses or if there are possible side effects from the vaccines.
* This can support resource allocation as well by distributing vaccines where needed
* Vaccination trends can also be tracked over time such as observing seasonal patterns
* Hospitals can also use this dashboard to schedule follow-up reminders to patients for annual flu shots 

# Improvements
Based on the insights derived from this analysis, forecasting can serve as an additional tool to anticipate flu seasons by examining flu vaccination rates. Predictive forecasting may also be employed to project the number of hospitalizations, medical consultations, and fatalities associated with influenza, contingent upon the levels of flu coverage and activity in specific regions or states. Furthermore, this analysis could be enhanced by integrating real-time data into the dashboard, providing hospital personnel with up-to-date information regarding patient vaccinations as well as their hospital visits. This integration facilitates a more effective distribution of resources to departments that experience a higher volume of patient visits.