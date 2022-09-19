<h2 align="center"> SQL SERVER BASICS</h2>

>This project is about how to create a database, add tables that exist in Excel and make some queries that related with this tables.

|| Headlines|
| :------: | ------ |
|`1` |[Create a database](#1)|
|`2` |[Add existing tables from Excel](#2)|
|`3` |[Queries abaout related table](#3)|

#### 1- Create a database <a name="#1"></a>

- On MS SQL Server first right click on databases folder and click on Create database.
- Write the database name and press OK button.
<img src="/images/createdb.png" alt="drawing" width="400"/> | <img src="/images/createdb2.png" alt="drawing" width="400"/>

#### 2- Add existing tables from Excel <a name="#2"></a>

- First lets look at the table that exist on Excel
<img src="/images/Exceltable.png" alt="drawing" height=500 width=800/>
<br>
- As we see there are three tables on excel so lets add them to the database.
- First right click on database and from task find İmport Data and click on it.
<img src="/images/importdata.png" alt="drawing" height=350 width=450/>
<br>
- Choose data source Microsoft Excel
- After that attach the file path of excel file. Simply click on Browse button and attach it.
- Press Next button
<img src="/images/datasource.png" alt="drawing" height=400 width=450/>
<br>
- Choose SQL Server Native Client as a Destination.
- Press Next button
<img src="/images/destination.png" alt="drawing" height=400 width=450/>
<br>
- Choose tables that you want to implement
- Press Next button
<img src="/images/tables.png" alt="drawing" height=400 width=450/>
<br>
- Click on finish and the tables will be implemented succesfully
<img src="/images/finish.png" alt="drawing" height=400 width=450/>
<br>
- The tables are implemented succesfully
<img src="/images/showtables.png" alt="drawing" height=450/>
<br>
- Press database diagram and set arrangment between primary and foreign keys.
<img src="/images/diagram.png" alt="drawing" height=350 width=450/>
<br>
- Tables relations is below. Now we can save and move to the T-Sql queies
<img src="/images/arrangetables.png" alt="drawing" height=450 width=600/>
<br>

#### 3- Queries abaout related table <a name="#3"></a>

######3.1 Write the query that brings the list of employees still working in our company. <br> Note: Those who have an null  indate are the employees who continue to work.

```sql
SELECT * FROM PERSON WHERE OUTDATE IS NULL
```
<br>

######3.2 Please write the query that brings the numbers of WOMEN and MEN who are still working on a departmental basis in our company.

```sql
--Using join
Select d.DEPARTMENT,
case 
	when p.GENDER='E' then 'ERKEK'
	when p.GENDER='K' then 'KADIN'
	end AS GENDER,
count(p.ID) PERSONCOUNT
from DEPARTMENT d inner join PERSON p on 
d.ID=p.DEPARTMENTID
WHERE OUTDATE IS NULL
group by p.GENDER,d.DEPARTMENT
```
```sql
--Using subquery
Select 
(Select DEPARTMENT from DEPARTMENT d where d.ID=p.DEPARTMENTID) as DEPARTMENT,
case 
	when p.GENDER='E' then 'ERKEK'
	when p.GENDER='K' then 'KADIN'
end GENDER,
count(p.DEPARTMENTID) PERSONCOUNT
from person p
WHERE OUTDATE IS NULL
group by p.GENDER,p.DEPARTMENTID
order by department
```
- Output
<img src="/images/q2.png" alt="drawing" height="300"/>

######3.3 Please write the query that brings the numbers of WOMEN and MEN who are still working on a departmental basis in our company. <br> Note: Women and men must be in different columns
```sql
--Using subquery
Select DEPARTMENT,
(Select count(*) from person where d.ID=DEPARTMENTID and GENDER='E' and OUTDATE is null) MALE_PERSONCOUNT,
(Select count(*) from person where d.ID=DEPARTMENTID and GENDER='K' and OUTDATE is null) FEMALE_PERSONCOUNT
from DEPARTMENT d
order by DEPARTMENT
```
- Output
<img src="/images/q3.png" alt="drawing" height="300"/>

######3.4 A new chief has been appointed to our company's Planning department and we would like to set his salary. Write the query that returns the minimum, maximum and average chief salary for the planning department. <br> Note: Dismissal salaries are also included.
```sql
--Using subquery
Select
(SELECT POSITION FROM POSITION WHERE P.POSITIONID=ID) POSITION,
MIN(SALARY) MINSALARY,
MAX(SALARY) MAXSALARY,
CEILING(AVG(SALARY)) AVGSALARY
from person P WHERE POSITIONID=(SELECT ID FROM POSITION WHERE POSITION='PLANLAMA ŞEFİ')
GROUP BY P.POSITIONID
```
```sql
--Using join
SELECT 
PST.POSITION,
MIN(SALARY) MINSALARY,
MAX(SALARY) MAXSALARY,
CEILING(AVG(SALARY)) AVGSALARY
FROM POSITION PST INNER JOIN PERSON P ON PST.ID=P.POSITIONID
WHERE PST.POSITION='PLANLAMA ŞEFİ'
GROUP BY PST.POSITION
```
- Output
<img src="/images/q4.png" alt="drawing"/>

######3.5 As current employees in each position, we want to list how many people and what their average salary is. Write the query that returns this result.

```sql
--Using subquery
SELECT 
(SELECT POSITION FROM POSITION WHERE P.POSITIONID=ID) AS POSITION,
COUNT(POSITIONID) PERSONCOUNT,
CEILING(AVG(SALARY)) AVG_SALARY
FROM PERSON P
WHERE OUTDATE IS NULL
GROUP BY POSITIONID
ORDER BY POSITION 
```
```sql
--Using Join
SELECT PST.POSITION,
COUNT(PST.POSITION) PERSONCOUNT,
CEILING(AVG(SALARY)) AVD_SALARY
FROM PERSON P INNER JOIN POSITION PST
ON P.POSITIONID=PST.ID
WHERE P.OUTDATE IS NULL
GROUP BY PST.POSITION
ORDER BY PST.POSITION
```

- Output
<img src="/images/q5.png" alt="drawing" height=250 />

######3.6 Write the query that lists the number of personnel recruited by year on the basis of men and women.

```sql
SELECT TMP.YEAR_,
(SELECT COUNT(GENDER) FROM PERSON WHERE GENDER='E' AND YEAR(INDATE)=YEAR_) MALE_PERSON, 
(SELECT COUNT(GENDER) FROM PERSON WHERE GENDER='K' AND YEAR(INDATE)=YEAR_) FEMALE_PERSON
FROM
(SELECT
CASE WHEN YEAR(INDATE)='2015' THEN 2015
WHEN YEAR(INDATE)='2016' THEN 2016
WHEN YEAR(INDATE)='2017' THEN 2017
WHEN YEAR(INDATE)='2018' THEN 2018
WHEN YEAR(INDATE)='2019' THEN 2019
END YEAR_,GENDER
FROM PERSON) TMP
GROUP BY YEAR_
ORDER BY YEAR_
```
- Output 
<img src="/images/q6.png" alt="drawing"/>

######3.7 Please write the query that brings the information of how long each of our personnel has been working in months.
```sql
SELECT
TMP.PERSON,CAST(TMP.INDATE AS DATE) INDATE,
CAST(TMP.OUTDATE AS DATE) OUTDATE,
DATEDIFF(MONTH,TMP.INDATE,TMP.OUTD) AS WORKINGTIME
FROM
(Select NAME_ +' ' + SURNAME as PERSON,
INDATE, OUTDATE,
CASE 
WHEN OUTDATE IS NULL THEN GETDATE()
WHEN OUTDATE IS NOT NULL THEN OUTDATE
END OUTD
from PERSON) AS TMP
```
- Output
<img src="/images/q7.png" alt="drawing" height=250/>

######3.8 In its 5th year, our company will print an agenda with the initials of everyone's names and surnames and present it to its employees. For this, write the query that answers the question of which letter combination will be printed at least how many times. <br> Note: The first letter of the first name will be used for those with two names.

```sql
SELECT 
SUBSTRING(NAME_,1,1)+'.'+SUBSTRING(SURNAME,1,1) AS SHORTNAME,
COUNT(SUBSTRING(NAME_,1,1)+'.'+SUBSTRING(SURNAME,1,1)) PERSONCOUNT
FROM PERSON
GROUP BY SUBSTRING(NAME_,1,1)+'.'+SUBSTRING(SURNAME,1,1)
ORDER BY COUNT(SUBSTRING(NAME_,1,1)+'.'+SUBSTRING(SURNAME,1,1)) DESC
```
- Output
<img src="/images/q8.png" alt="drawing" height=250/>

######3.9 Write the query that will list the departments with an average salary of more than 5,550 TL.

```sql
--Using Subquery
SELECT 
(SELECT DEPARTMENT FROM DEPARTMENT WHERE ID=P.DEPARTMENTID) DEPARTMENT,
ROUND(AVG(SALARY),0) AVGSALARY FROM PERSON P
GROUP BY DEPARTMENTID
HAVING AVG(SALARY)>5500
ORDER BY DEPARTMENT
```
```sql
--Using Join
SELECT 
DPT.DEPARTMENT,ROUND(AVG(SALARY),0) AS AVGSALARY
FROM DEPARTMENT DPT INNER JOIN PERSON P
ON P.DEPARTMENTID=DPT.ID
GROUP BY DPT.DEPARTMENT
HAVING AVG(SALARY)>5500
```
- Output
<img src="/images/q9.png" alt="drawing"/>

######3.10 Write the query in the drawer to calculate the average seniority of the departments in months and as in the figure.

```sql
SELECT 
TMP.DEPARTMENT, 
AVG(DATEDIFF(MONTH,TMP.INDATE,TMP.OUTDTH)) AVG_WORKINGTIME 
FROM
(Select 
(SELECT DEPARTMENT FROM DEPARTMENT WHERE P.DEPARTMENTID=ID) DEPARTMENT,
INDATE,OUTDATE,
CASE
	WHEN OUTDATE IS NULL THEN GETDATE()
	WHEN OUTDATE IS NOT NULL THEN OUTDATE
	END OUTDTH
from PERSON P) TMP
GROUP BY TMP.DEPARTMENT
```
- Output
<img src="/images/q10.png" alt="drawing" height=200/>

######3.11 Write down the query that brings the name of each personnel, the name of the unit manager to which they are affiliated, and their position.

```sql
SELECT
NAME_+' '+SURNAME AS PERSON,
(SELECT POSITION FROM POSITION WHERE ID=P.POSITIONID) AS POSITION,
(SELECT NAME_+' '+SURNAME FROM PERSON WHERE ID=P.MANAGERID) as MANAGER,
(Select POSITION from POSITION where ID=P.PARENTPOSITIONID) AS MANAGERPOSITION
FROM PERSON P
where MANAGERID is not null
```
- Output
<img src="/images/q11.png" alt="drawing" height=200/>