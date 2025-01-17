# SQL-data-cleaning
This is an educational project on data cleaning and preparation using SQL. The original database in CSV format is located in the file club_member_info.csv. Here, we will explore the steps that need to be applied to obtain a cleansed version of the dataset.

Let's inspect the initial rows to analyze data in its original format

`select * from club_member_info cmi limit 10;`

|full_name|age|martial_status|email|phone|full_address|job_title|membership_date|
|---------|---|--------------|-----|-----|------------|---------|---------------|
|addie lush|40|married|alush0@shutterfly.com|254-389-8708|3226 Eastlawn Pass,Temple,Texas|Assistant Professor|7/31/2013|
|      ROCK CRADICK|46|married|rcradick1@newsvine.com|910-566-2007|4 Harbort Avenue,Fayetteville,North Carolina|Programmer III|5/27/2018|
|Sydel Sharvell|46|divorced|ssharvell2@amazon.co.jp|702-187-8715|4 School Place,Las Vegas,Nevada|Budget/Accounting Analyst I|10/6/2017|
|Constantin de la cruz|35||co3@bloglines.com|402-688-7162|6 Monument Crossing,Omaha,Nebraska|Desktop Support Technician|10/20/2015|
|  Gaylor Redhole|38|married|gredhole4@japanpost.jp|917-394-6001|88 Cherokee Pass,New York City,New York|Legal Assistant|5/29/2019|
|Wanda del mar       |44|single|wkunzel5@slideshare.net|937-467-6942|10864 Buhler Plaza,Hamilton,Ohio|Human Resources Assistant IV|3/24/2015|
|Joann Kenealy|41|married|jkenealy6@bloomberg.com|513-726-9885|733 Hagan Parkway,Cincinnati,Ohio|Accountant IV|4/17/2013|
|   Joete Cudiff|51|divorced|jcudiff7@ycombinator.com|616-617-0965|975 Dwight Plaza,Grand Rapids,Michigan|Research Nurse|11/16/2014|
|mendie alexandrescu|46|single|malexandrescu8@state.gov|504-918-4753|34 Delladonna Terrace,New Orleans,Louisiana|Systems Administrator III|3/12/1921|
| fey kloss|52|married|fkloss9@godaddy.com|808-177-0318|8976 Jackson Park,Honolulu,Hawaii|Chemical Engineer|11/5/2014|

## Copy the table
### Create a new table for cleaning
Generate a new table for cleaning

    CREATE TABLE club_member_info_cleaned (
	full_name VARCHAR(50),
	age INTEGER,
	martial_status VARCHAR(50),
	email VARCHAR(50),
	phone VARCHAR(50),
	full_address VARCHAR(50),
	job_title VARCHAR(50),
	membership_date VARCHAR(50)
    );
Copy all values from original table to destinated table

    Insert into club_member_info_cleaned
    select * from club_member_info;

## Cleaning data
### Cleaning the name

	Update club_member_info_cleaned 
	set full_name = trim(full_name)

 	Update club_member_info_cleaned 
	set full_name = TRIM(UPPER(SUBSTR(full_name, 1, 1)) ||LOWER(SUBSTR(full_name, 2, INSTR(full_name || ' ', ' ') - 1)) || UPPER(SUBSTR(full_name, INSTR(full_name || ' ', ' ') + 1, 1)) || LOWER(SUBSTR(full_name, INSTR(full_name || ' ', ' ') + 2)))

Result
|full_name|full_name_clean|
|---------|---------------|
|addie lush|Addie Lush|
|      ROCK CRADICK|Rock Cradick|
|Sydel Sharvell|Sydel Sharvell|
|Constantin de la cruz|Constantin De la cruz|
|  Gaylor Redhole|Gaylor Redhole|
|Wanda del mar       |Wanda Del mar|
|Joann Kenealy|Joann Kenealy|
|   Joete Cudiff|Joete Cudiff|
|mendie alexandrescu|Mendie Alexandrescu|
| fey kloss|Fey Kloss|

### Cleaning the age
Update ages to NULL if they are outside a reasonable range (e.g., ages > 120)

    UPDATE club_member_info_cleaned 
    SET age = NULL
    WHERE age IS NOT NULL AND (age > 120 OR age < 0);
Result
|age|age_clean|
|---|---------|
|40|40|
|555|NULL|
|46|46|
|499|NULL|
|522|NULL|
||NULL|
|277|NULL|
||NULL|
|288|NULL|
|588|NULL|

### Cleaning the marital status
Change the column name to marital_status from martial_status

    ALTER TABLE club_member_info_cleaned 
    RENAME COLUMN martial_status TO marital_status
Correct the spelling of divorced

    UPDATE club_member_info_cleaned 
    set marital_status = 'divorced'
    where marital_status = 'divored'
Empty value change to Null

	UPDATE club_member_info_cleaned
 	set marital_status = NULL
  	where marital_status = ''
Result
|martial_status|marital_status|
|--------------|--------------|
|divored|divorced|
|married|married|
|married|married|
|divorced|divorced|
||NULL|
|married|married|
|single|single|

### Cleaning the job title
Change the empty job title to NULL value

    UPDATE club_member_info_cleaned
	set job_title = NULL
	where job_title = ''
### Cleaning Duplicate
Check duplicate rows

 	SELECT full_name, age, marital_status, email, phone, full_address, job_title, membership_date, count(*)
	FROM club_member_info_cleaned cmic 
	GROUP BY full_name, age, marital_status, email, phone, full_address, job_title, membership_date
	HAVING count(*) >1

Result
|full_name|age|marital_status|email|phone|full_address|job_title|membership_date|count(*)|
|---------|---|--------------|-----|-----|------------|---------|---------------|--------|
|Erwin Huxter|25|married|ehuxterm0@marketwatch.com|704-295-3261|0 Homewood Road,Charlotte,North Carolina|Software Test Engineer III|9/29/2017|2|
|Nicki Filliskirk|66|married|nfilliskirkd5@newsvine.com|410-848-2272|7657 Alpine Plaza,Baltimore,Maryland|Geologist IV|6/18/2021|2|
|Tamqrah Dunkersley|36|single|tdunkersley8u@dedecms.com|651-939-2423|0 Colorado Terrace,Saint Paul,Minnesota|VP Sales|6/27/2016|2|

Remove duplicate

	-- Step 1: Create a temporary table
	CREATE TEMPORARY TABLE temp_table AS
	SELECT full_name, age, marital_status, email, phone, full_address, job_title, membership_date
	FROM club_member_info_cleaned
	WHERE 1=0;
 
	/*The 1=0 condition is used to create an empty table
	with the same structure as the club_member_info_cleaned table.
	This ensures that the temporary table (temp_table) has the same columns but no rows initially.
	*/
 
	-- Step 2: Insert unique rows into the temporary table
	INSERT INTO temp_table
	SELECT full_name, age, marital_status, email, phone, full_address, job_title, membership_date
	FROM club_member_info_cleaned
	GROUP BY full_name, age, marital_status, email, phone, full_address, job_title, membership_date;

	-- Step 3: Delete all rows from the original table
	DELETE FROM club_member_info_cleaned;

	-- Step 4: Insert the unique rows back into the original table
	INSERT INTO club_member_info_cleaned
	SELECT * FROM temp_table;

	-- Step 5: Drop temp_table
	DROP TABLE temp_table;


## Final result
Final result is in the table *club_member_info_cleaned* in `club_member.db` file
