# The-SQL-Murder-Mystery
A walkthrough on how i found whodunnit

![schema](https://github.com/user-attachments/assets/651621c3-3d78-44f7-88e8-b3a5948af0d1)

🔍 Solving the SQL Murder Mystery: A Data Analyst’s Adventure 🕵️♂️
I recently completed the SQL Murder Mystery, and I must say—it was even more 
enjoyable than SQL Island! This experience allowed me to step into the role of a detective, 
using my SQL skills to crack a murder case. Talk about combining data analysis with 
investigative work!
For a follow along, table schema can be found at the top of this document.
Excerpts: ““You vaguely remember that the crime was a murder that occurred sometime on Jan.15, 
2018 and that it took place in SQL City. Start by retrieving the corresponding crime scene report from 
the police department’s database."
My first task? Retrieve the crime scene report from the database.
Step 1: Uncovering the Crime Scene Details
Using the query below, I retrieved the crime scene report:

SELECT *
FROM crime_scene_report
WHERE date = 20180115
AND city = 'SQL City'
AND type = 'murder'; 

The report revealed crucial details: two witnesses—one living on "Northwestern Dr" and the 
other, Annabel, residing somewhere on "Franklin Ave."
date type description city
20180115murder Security footage shows that there were 2 witnesses. The first witness lives at the last house on 
"Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".
SQL 
City
Step 2: Finding and Questioning the Witnesses
To locate and interview these witnesses, I wrote the following queries:
SELECT *
FROM person
WHERE name LIKE '%Annabel%'
AND address_street_name LIKE 'Franklin Ave';
SELECT *
FROM person
WHERE address_street_name LIKE 'Northwestern Dr'
ORDER BY address_number DESC
LIMIT 1;
id name license_id address_number address_street_name ssn
16371 Annabel Miller 490173 103 Franklin Ave 318771143
id name license_id address_number address_street_name ssn
14887 Morty Schapiro 118009 4919 Northwestern Dr 111564949
Here’s what I discovered:
 Witness 1: Annabel Miller (Address: 103 Franklin Ave)
 Witness 2: Morty Schapiro (Address: 4919 Northwestern Dr)
Next, I interviewed them to piece together the events leading up to the murder:
SELECT *
FROM interview
WHERE person_id IN (14887, 16371);
person_id transcript
14887 I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number 
on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that 
included "H42W".
16371 I saw the murder happen, and I recognized the killer from my gym when I was working out last week on 
January the 9th.
Step 3: Following the Leads
The witnesses provided valuable clues:
1. The killer carried a "Get Fit Now Gym" bag with a membership ID starting with 
"48Z".
2. They fled in a car with a plate number containing "H42W".
3. The crime occurred on January 9, 2018.
My next move was to identify gym members and vehicle owners matching this description:
SELECT *
FROM get_fit_now_check_in
WHERE check_in_date = 20180109
AND membership_id LIKE '48Z%';
SELECT *
FROM drivers_license
WHERE plate_number LIKE '%H42W%';
These are the people that checked in on the day of the murder into the gym
membership_id check_in_date check_in_time check_out_time
48Z7A 20180109 1600 1730
48Z55 20180109 1530 1700
These are the people with plate number that included "H42W".
id age height eye_color hair_color gender plate_number car_make car_model
183779 21 65 blue blonde female H42W0X Toyota Prius
423327 30 70 brown brown male 0H42W2 Chevrolet Spark LS
664760 21 71 black black male 4H42WR Nissan Altima
By cross-referencing these two datasets, we know our killer has to match these two tables. To 
confirm their identity, since we don’t have their names, just their IDs and few other information, we 
go to our person table to get their names.
SELECT *
FROM person
WHERE license_id IN (183779, 423327, 664760);
Now we have the names of the people with plate number that included "H42W" and were most 
likely at the gym on the day of the murder.
id name license_id address_number address_street_name ssn
51739 Tushar Chandra 664760 312 Phi St 137882671
67318 Jeremy Bowers 423327 530 Washington Pl, Apt 3A 871539279
78193 Maxine Whitely 183779 110 Fisk Rd 137882671
Now we have the names, lets screen them against the gym membership to see which one is a 
member.
SELECT *
FROM get_fit_now_member
WHERE person_id IN (51739, 67318, 78193);
id person_id name membership_start_date membership_status
48Z55 67318 Jeremy Bowers 20160101 gold
The killer was Jeremy Bowers (Person ID: 67318), a gym member and the driver of the car identified 
at the scene.
Step 4: The Plot Twist
Just when I thought the case was closed, SQL murder decides to put a plot twist and makes us find 
who hired the killer, so next we interview the killer to see what we can find.
SELECT *
FROM interview
WHERE person_id = 67318;
person_id transcript
67318 I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 
5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony 
Concert 3 times in December 2017.
So up until now, I tried so hard to avoid using joins (lol) but SQL murder decides to challenge me to 
find the murder with two queries, so I have no choice but to use joins.
So here we go, we know it’s a woman, rich woman but we weren’t given a figure, so I guess rich is 
subjective, so I wont bother integrating that into my search (for now). We also know her height 
range, her hair color, the car she drives and the event she attended, all useful information to catch 
who hired our killer.
SELECT p.name, d.hair_color, f.event_name, d.gender, d.car_make, d.car_model, d.height, 
i.annual_income
FROM person p
JOIN drivers_license d
 ON p.license_id = d.id
JOIN facebook_event_checkin f
 ON p.id = f.person_id
JOIN income i
ON p.ssn= i.ssn
WHERE d.hair_color = 'red'
 AND f.event_name = 'SQL Symphony Concert'
 AND d.gender = 'female'
 AND d.car_make = 'Tesla';FROM person p
JOIN drivers_license d
 ON p.license_id = d.id
JOIN facebook_event_checkin f
 ON p.id = f.person_id
WHERE d.hair_color = 'red'
 AND f.event_name = 'SQL Symphony Concert'
 AND d.gender = 'female'
 AND d.car_make = 'Tesla';
name hair_color event_name gender car_make car_model height annual_income
Miranda 
Priestly
red SQL Symphony 
Concert
female Tesla Model S 66 310000
name hair_color event_name gender car_make car_model height annual_income
Miranda 
Priestly
red SQL Symphony 
Concert
female Tesla Model S 66 310000
Miranda 
Priestly
red SQL Symphony 
Concert
female Tesla Model S 66 310000
The mastermind behind the murder was Miranda Priestly—a red-haired, Tesla-driving woman of 
wealth and mystery. To confirm her financial status, I peeked at the income table, satisfying my 
curiosity:
SELECT *
FROM income
ORDER BY annual_income DESC
LIMIT 5;
ssn annual_income
361660921 498500
121635236 489800
118015315 486600
541217354 476300
313890530 475700
Case Closed
After analyzing the data, I identified both the killer and the person who hired them. This 
experience was a fantastic reminder of what it means to be a data analyst: gathering, 
analyzing, and synthesizing data to solve complex problems.
💡 Key Takeaway: Data analysts are detectives in their own right, transforming raw data into 
actionable insights. This game was a perfect combination of storytelling and SQL—a fun and 
educational experience I’d recommend to anyone!
Have you tried the SQL Murder Mystery? Let’s connect and discuss your favorite queries or 
challenges!
#SQL #DataAnalysis #StorytellingWithData #SQLMurderMystery #LearningJourney
