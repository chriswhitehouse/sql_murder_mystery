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

### Step 5: Identifying Suspects
Licence Plate "H42W":
``` SQL
select *
From drivers_license
Where plate_number like '%H42W%'
```

Clue:
|id	|age	|height	|eye_color	|hair_color	|gender	|plate_number	|car_make	|car_model|
|--|------|-------|-----------|-----------|-------|-------------|---------|---------|
|183779	|21	|65	|blue	|blonde	|female	|H42W0X	|Toyota	|Prius|
|423327|	30|	70|	brown|	brown|	male|	0H42W2|	Chevrolet|	Spark| LS|
|664760|	21|	71|	black|	black|	male|	4H42WR|	Nissan|	Altima|

Get Fit Now Membership '48Z' with Gold membership:
```SQL
select *
From get_fit_now_member
Where id like '48Z%' and membership_status = 'gold'
```

Clue:
|id	|person_id|	name|	membership_start_date|	membership_status|
|---|---------|-----|----------------------|-------------------|
|48Z7A|	28819|	Joe Germuska|	20160305|	gold|
|48Z55|	67318|	Jeremy Bowers|	20160101|	gold|


Get Fit Now January the 9th checkin:
```SQL
select *
From get_fit_now_check_in
Where membership_id like '48Z%' and check_in_date = 20180109
```

Clue:
|membership_id|	check_in_date|	check_in_time|	check_out_time|
|-------------|--------------|---------------|----------------|
|48Z7A	|20180109	|1600	|1730|
|48Z55|	20180109|	1530|	1700|

### Step 6: Confirming Suspect
Clue: person_id 28819 or 67318 in person table
```SQL
select *
From person
Where id in (28819, 67318)
```

Clue:
|id	|name	|license_id|	address_number|	address_street_name|	ssn|
|---|-----|----------|----------------|--------------------|-----|
|28819|	Joe Germuska|	173289|	111|	Fisk Rd|	138909730|
|67318|	Jeremy Bowers|	423327|	530|	Washington Pl, Apt 3A|	871539279|

Clue: person_id 28819 or 67318 in interview table:
```SQL
select *
From interview
Where person_id in (28819, 67318)
```

Clue:
|person_id	|transcript|
|67318|	I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.|

### Step 7: Identify Woman with Red Hair
```SQL
SELECT *
From drivers_license
Where hair_color = 'red' and car_make = 'Tesla' and car_model = 'Model S' and gender = 'female'
 ```

Clue:
|id|	age|	height|	eye_color|	hair_color|	gender|	plate_number|	car_make|	car_model|
|---|----|--------|---------|-------------|-------|-------------|---------|----------|
|202298| 68|	66|	green|	red|	female|	500123|	Tesla|	Model S|
|291182|	65|	66|	blue|	red|	female|	08CM64|	Tesla|	Model S|
|918773|	48|	65|	black|	red|	female|	917UU3|	Tesla|	Model S|

```SQL
Select *
FROM person
Where license_id in (202298, 291182, 918773)
```

Clue:
|id	|name	|license_id|	address_number|	address_street_name|	ssn|
|----|-----|---------|----------------|-------------------|------|
|78881	|Red Korb|	918773|	107|	Camerata Dr|	961388910|
|90700	|Regina George|	291182|	332|	Maple Ave|	337169072|
|99716	|Miranda Priestly|	202298|	1883|	Golden Ave|	987756388|


```SQL
Select *
FROM income
Where ssn in (961388910, 337169072, 987756388)
```
Clue
|ssn	|annual_income|
|----|------------|
|961388910|	278000|
|987756388|	310000|

```SQL
Select *
FROM facebook_event_checkin
Where person_id in (78881, 99716)
```

Clue:
|person_id	|event_id|	event_name|	date|
|---|-----|----------|------------|------|
|99716|	1143|	SQL Symphony Concert|	20171206|
|99716|	1143	|SQL Symphony Concert|	20171212|
|99716|	1143|	SQL Symphony Concert|	20171229|

Miranda Priestly checked into the SQL Symphony Concert 3 times in December 2017!

```SQL
INSERT INTO solution VALUES (1, 'Miranda Priestly');

        SELECT value FROM solution;
```
|value|
|-----|
|Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne!|
