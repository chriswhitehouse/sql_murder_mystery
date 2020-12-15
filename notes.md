#SQL Murder Mystery - Notes

### Clue 1 Revisiting Crime Scene Report
Query:
```SQL
Select *
From crime_scene_report

Where type = 'murder' and date = 20180115 and city = 'SQL City'
```
Clue: Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".

### Step 2 interrogating the person table for witnesses
Witness 1:
```SQL
Select *

From person

Where address_number = (

Select max(address_number) as 'Max Address Number'
From person

Where address_street_name = 'Northwestern Dr')
```

Clue;
|id|	name|	license_id|	address_number|	address_street_name|	ssn|
|--|-----|------------|---------------|---------------------|----|
|14887|	Morty Schapiro|	118009|	4919|	Northwestern Dr|	111564949|

Witness 2:
``` SQL
select *

From person

Where address_street_name = 'Franklin Ave' and name like 'Annabel%'
```

Clue:
|id	|name	|license_id	|address_number	|address_street_name	|ssn|
|--|-----|------------|---------------|---------------------|----|
|16371|	Annabel Miller|	490173|	103|	Franklin Ave|	318771143|

### Step 4: Looking up witness interviews
Witness 1:
``` SQL
select *

From interview

Where person_id = 14887
```

Clue:
|person_id|	transcript|
|--|--|
|14887|	I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".|

Witness 2:
``` SQL
select *

From interview

Where person_id = 16371
```

Clue:
|person_id|	transcript|
|--|--|
|16371|	I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.|
