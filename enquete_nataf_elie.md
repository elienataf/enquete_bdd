### Enquête sur la base de donnée

# Installation

J'ai commencé par télécharger Sqlite, puis je l'ai installé.

J'ai ouvert la base de donnée fournie pour obtenir les informations
liées à l'enquête. Commande utilisée :

``` sql
sqlite3 "C:\Users\Elie\Téléchargements\sql-murder-mystery.db"
```

# Début de l'enquête

J'ai fais un `.table` pour comprendre la structure de la base

Voici le résultat renvoyé :

-----------------------------------------------------------------------

| Tables                  |
| :-----------------------|
| crime_scene_report      |
| get_fit_now_check_in    |
| interview               |
| drivers_license         |
| get_fit_now_member      |
| person                  |
| facebook_event_checkin  |
| income                  |
| solution                |

-----------------------------------------------------------------------

  

J'ai fais un `SELECT * FROM` pour toutes les tables afin de comprendre
vers ou j'allais m'orienter pour réaliser mon enquête.

On sait que le meutre à eu lieu le 15 janvier 2018 dans la ville de SQL
City. La table qui me semble la plus pertinente est la première car elle
devrait contenir le rapport du crime

J'ai donc réalisé la commande suivante qui me permet d'obtenir les
détails du meurtre :

``` sql
SELECT * FROM crime_scene_report WHERE type = 'murder' AND city = 'SQL City' AND date = 20180115;
```

Voici le résultat renvoyé :

  -----------------------------------------------------------------------
  |date        |  type       |   description               |    city    |
  | :----------|-------------|-----------------------------|-----------:|
  |20180115     | murder       | Security footage shows that there were 2 witnesses. The first witness lives at the  last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".| SQL City|

  -----------------------------------------------------------------------
Le rapport indique qu'un meutre à eu lieu le 15 janvier 2018 à SQL City,
2 coupables sont mentionnés. Il existe 2 témoins : Annabel qui habite
vers "Franklin Ave" et un autre dont on ne connait pas le prénom mais
qui vit dans la dernière maison de "Northwestern Dr".

# Comprendre la table person :

Je commence par explorer la table person afin de comprendre sa structure
et ainsi faire des recherches sur mes témoins.

Voici la commande que je tape dans mon terminal :

``` sql
SELECT * FROM person LIMIT 10;
```

Voici une ligne de ce que cette commande me renvoie :

-----------------------------------------------------------------------

| id      |  name                 |  license_id  |  address_number  |  address_street_name  |  ssn        |
| :------ |---------------------- |-------------:|-----------------:|-----------------------|------------:|
| 10000   |  Christoper Peteuil   |  993845      |  624             |  Bankhall Ave         |  747714076  |

-----------------------------------------------------------------------


Je réalise ensuite cette commande qui va me permettre d'avoir le nom des
colonnes ppour que je puisse les utiliser :

``` sql
.schema person
```

Rien ne m'a été renvoyé donc j'ai refais la commande en enlevant le `;`

Voici le résultat obtenue :

``` sql
CREATE TABLE person (id integer PRIMARY KEY, name text, license_id integer, address_number integer, address_street_name text, ssn CHAR REFERENCES income (ssn), FOREIGN KEY (license_id) REFERENCES drivers_license (id));
```

Cette table comprend un ID, un numéro de rue , un nom de rue...

# Enquête sur le premier témoin :

Le premier témoin vit sur "Northwestern Dr"

Je vais donc écrire la ligne de code suivante qui va me permettre de
trouver les personnes vivants dans cette rue :

``` sql
SELECT * FROM person WHERE address_street_name = 'Northwestern Dr';
```

J'obtiens un grand nombre de résultats donc je vais filtrer mes
résultats selon l'ordre décroissant car on sait que notre témoin vit
dans la dernière maison.

Voici la commande que j'utilise :

``` sql
SELECT * FROM person WHERE address_street_name = 'Northwestern Dr' ORDER BY address_number DESC;
```

La personne vivant dans la dernière maison est la suivante : Morty
Schapiro

# Enquête sur le deuxième témoin :

Pour le second témoin on a le nom et la rue de la personnne donc on a
juste a selectionner les personnes qui s'appellent Annabel et qui vivent
dans l'avenue Franklin.

Pour cela j'utilise la commande suivante :

``` sql
SELECT * FROM person WHERE name LIKE '%Annabel%' AND address_street_name = 'Franklin Ave';
```

On obtient le nom du deuxième témoin : Annabel Miller.

# Obtenir les commentaires des 2 témoins

J'ai d'abord pas curiosité fais un

``` sql
SELECT * FROM solution;
```

mais j'ai obtenu aucun résultat.

Ensuite, j'ai fais un

``` sql
SELECT * FROM interview;
```

il y en a un très grand nombre et il faut l'ID des personnes pour
trouver leurs interview donc je suis allé rechercher les ID de nos 2
témoins.

ID Morty Schapiro : 14887
ID Annabel Miller : 16371

Maintenant, je peux faire la commande suivante pour obtenir les
interviews de nos témoins :

``` sql
SELECT * FROM interview WHERE person_id = 14887;
```

Le résultat obtenue pour Morty :

-----------------------------------------------------------------------

| person_id | transcript |
| :---------|-----------:|
| 14887     | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W". |

-----------------------------------------------------------------------


La traduction en français donne ca : "J'ai entendu un coup de feu, puis
j'ai vu un homme sortir en courant. Il portait un sac de la salle de
sport « Get Fit Now ». Le numéro d'adhérent sur le sac commençait par «
48Z ». Seuls les membres Gold possèdent ce type de sac. L'homme est
monté dans une voiture dont la plaque d'immatriculation comportait les
lettres « H42W »."

Je fais la même commande pour Annabel et voici le résultat que j'obtiens
:

``` sql
SELECT * FROM interview WHERE person_id = 16371;
```

-----------------------------------------------------------------------

| person_id | transcript |
| :---------|-----------:|
| 16371     | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th. |

-----------------------------------------------------------------------


Voici la traduction en francais : "J'ai vu le meurtre se produire et
j'ai reconnu le tueur de ma salle de sport lorsque je m'entraînais la
semaine dernière, le 9 janvier."

# Retrouver le meutrier en accédant aux données de la salle de sport

Dans un premier temps, j'ai fais un

``` sql
.schema get_fit_now_member
```

et un

``` sql
.schema get_fit_now_check_in
```

pour que je sache ou est ce que je vais chercher ses informations.

Voici le résultat pour la première commande :

``` sql
CREATE TABLE get_fit_now_member (
        id text PRIMARY KEY,
        person_id integer,
        name text,
        membership_start_date integer,
        membership_status text,
        FOREIGN KEY (person_id) REFERENCES person(id)
    );
```

Voici le résultat pour la seconde commande :

``` sql
CREATE TABLE get_fit_now_check_in (
        membership_id text,
        check_in_date integer,
        check_in_time integer,
        check_out_time integer,
        FOREIGN KEY (membership_id) REFERENCES get_fit_now_member(id)
    );
```

Je comprends donc que pour avoir le nom de la personne, il faut que je
fasse des recherches sur la table get_fit_now_member.

On a grâce au premier témoin les 3 premiers caractères de son numéro
d'adhérent qui sont les suivants : 48Z.

Voici la commmande que j'utilise :

``` sql
SELECT * FROM get_fit_now_member WHERE id LIKE '48Z%';
```

Voici le résultat renvoyé :

-----------------------------------------------------------------------

| id     |  person_id  |  name            |  membership_start_date  |  membership_status  |
| :----- |------------:|----------------- |------------------------:|---------------------|
| 48Z38  |  49550      |  Tomas Baisley   |  20170203               |  silver             |
| 48Z7A  |  28819      |  Joe Germuska    |  20160305               |  gold               |
| 48Z55  |  67318      |  Jeremy Bowers   |  20160101               |  gold               |

-----------------------------------------------------------------------


On sait aussi grâce au premier témoin qu'il possède un sac que seul les
membres gold peuvent avoir donc on sait que le suspect n'est pas Tomas
Baisley.

# Vérifier si les deux suspects étaient présent le jour du crime

Notre deuxième témoin à assurer avoir vu le tueur a la salle de sport
une semaine avant soit le 9 janvier. Je vais pouvoir vérifier la date a
laquelle les deux suspects sont venus a la salle de sport grace a la
table get_fit_now_check_in.

Je vais donc utiliser la ligne de code suivant qui va vérifier la
présence des suspects :

``` sql
SELECT * FROM get_fit_now_check_in WHERE membership_id = '48Z7A';
```

Joe Germuska était bien présent le 9 janvier.

``` sql
SELECT * FROM get_fit_now_check_in WHERE membership_id = '48Z55';
```

Jeremy Bowers était également présent le 9 janvier.

# Vérifier la plaque d'immatriculation des 2 suspects

Avec l'aide des témoins, le seul moyen d'identification qu'il nous reste
est leur plaque d'immatriculation car on sait que l'homme est monté dans
un véhicule immatriculé H42W...

On va pour cela utiliser la table drivers_license, dans un premier temps
on fait un .schema pour vérifier si les plaques sont enregistrés.

commande :

``` sql
.schema drivers_license
```

résultat :

``` sql
CREATE TABLE drivers_license (
        id integer PRIMARY KEY,
        age integer,
        height integer,
        eye_color text,
        hair_color text,
        gender text,
        plate_number text,
        car_make text,
        car_model text
    );
```

plate_number va nous permettre de retrouver les plaques.

Voici la commande que j'utilise :

``` sql
SELECT * FROM drivers_license WHERE plate_number LIKE '%H42W%';
```

Voici le résultat obtenu :

-----------------------------------------------------------------------

| id      |  age  |  height  |  eye_color  |  hair_color  |  gender  |  plate_number  |  car_make   |  car_model  |
| :------ |------:|---------:|-------------|--------------|----------|----------------|-------------|-------------|
| 183779  |  21   |  65      |  blue       |  blonde      |  female  |  H42W0X        |  Toyota     |  Prius      |
| 423327  |  30   |  70      |  brown      |  brown       |  male    |  0H42W2        |  Chevrolet  |  Spark LS   |
| 664760  |  21   |  71      |  black      |  black       |  male    |  4H42WR        |  Nissan     |  Altima     |

-----------------------------------------------------------------------


Je vais donc chercher dans la table personne quels sont les noms
associés à ces numéros de licenses à l'aide de la commande suivante :

``` sql
SELECT * FROM person WHERE license_id IN (183779, 423327, 664760);
```

et le résultat que l'on obtient va nous permettre de conclure :

-----------------------------------------------------------------------

| id      |  name               |  license_id  |  address_number  |  address_street_name        |  ssn        |
| :------ |-------------------- |-------------:|-----------------:|-----------------------------|------------:|
| 51739   |  Tushar Chandra     |  664760      |  312             |  Phi St                     |  137882671  |
| 67318   |  Jeremy Bowers      |  423327      |  530             |  Washington Pl, Apt 3A      |  871539279  |
| 78193   |  Maxine Whitely     |  183779      |  110             |  Fisk Rd                    |  137882671  |

-----------------------------------------------------------------------


# Conlusion

En effet, le meurtrier est donc Jeremy Bowers car Joe Germuska n'est pas
mentionné.

Pour ajouter Jeremy a la table solution, j'ai fais un

``` sql
INSERT INTO solution VALUES (1, 'Jeremy Bowers');
```

J'ai obtenu ce message : "Congrats, you found the murderer! But wait,
there's more... If you think you're up for a challenge, try querying the
interview transcript of the murderer to find the real villain behind
this crime. If you feel especially confident in your SQL skills, try to
complete this final step with no more than 2 queries. Use this same
INSERT statement with your new suspect to check your answer."

Je décide donc d'aller interoger Jeremy, le tueur, avec cette commande :

``` sql
SELECT * FROM interview WHERE person_id = 67318;
```

Résultat :

  -----------------------------------------------------------------------
  person_id                         transcript
  --------------------------------- -------------------------------------
  67318                             I was hired by a woman with a lot of
                                    money. I don't know her name but I
                                    know she's around 5'5" (65") or 5'7"
                                    (67"). She has red hair and she
                                    drives a Tesla Model S. I know that
                                    she attended the SQL Symphony Concert
                                    3 times in December 2017.

  -----------------------------------------------------------------------

Traduction : 67318\|J'ai été embauché par une femme très riche. J'ignore
son nom, mais je sais qu'elle mesure environ 1,65 m ou 1,70 m. Elle est
rousse et conduit une Tesla Model S. Je sais qu'elle a assisté trois
fois au concert symphonique de SQL en décembre 2017.

Je vais donc sélectionner tous les gens avec un permis possedant un
Tesla Model S :

``` sql
SELECT * FROM drivers_license WHERE car_make = 'Tesla' AND car_model = 'Model S';
```
La commande suivante va nous permettre de filtré selon le témoignage de
Jeremy :

``` sql
SELECT * FROM drivers_license WHERE car_make = 'Tesla' AND car_model = 'Model S' AND gender = 'female' AND hair_color = 'red' AND height BETWEEN 65 AND 70;
```


Voici ce qu'on obtient :

-----------------------------------------------------------------------

| id      |  age  |  height  |  eye_color  |  hair_color  |  gender  |  plate_number  |  car_make  |  car_model  |
| :------ |------:|---------:|-------------|--------------|----------|----------------|------------|-------------|
| 202298  |  68   |  66      |  green      |  red         |  female  |  500123        |  Tesla     |  Model S    |
| 291182  |  65   |  66      |  blue       |  red         |  female  |  08CM64        |  Tesla     |  Model S    |
| 918773  |  48   |  65      |  black      |  red         |  female  |  917UU3        |  Tesla     |  Model S    |

-----------------------------------------------------------------------



Grâce à la commande suivante, on va obtenir les noms des 3 nouveaux
suspects :

``` sql
SELECT * FROM person WHERE license_id IN (202298, 291182, 918773);
```

résultat :

-----------------------------------------------------------------------

| id      |  name               |  license_id  |  address_number  |  address_street_name  |  ssn        |
| :------ |-------------------- |-------------:|-----------------:|-----------------------|------------:|
| 78881   |  Red Korb           |  918773      |  107             |  Camerata Dr          |  961388910  |
| 90700   |  Regina George      |  291182      |  332             |  Maple Ave            |  337169072  |
| 99716   |  Miranda Priestly   |  202298      |  1883            |  Golden Ave           |  987756388  |

-----------------------------------------------------------------------


Je vais regarder ce qui compe la table facebook_event_checkin

j'utilise cette commande :

``` sql
.schema facebook_event_checkin
```

Je vais récupérer toutes les données associés aux 3 ID que j'ai trouvé.

``` sql
SELECT * FROM facebook_event_checkin WHERE person_id = '78881';
SELECT * FROM facebook_event_checkin WHERE person_id = '90700';
SELECT * FROM facebook_event_checkin WHERE person_id = '99716';
```

J'ai aucun résultats pour les 2 premiers mais pour le troisième, voici
ce que j'obtiens :

-----------------------------------------------------------------------

| person_id  |  event_id  |  event_name             |  date       |
| :----------|-----------:|------------------------ |------------:|
| 99716      |  1143      |  SQL Symphony Concert   |  20171206   |
| 99716      |  1143      |  SQL Symphony Concert   |  20171212   |
| 99716      |  1143      |  SQL Symphony Concert   |  20171229   |

-----------------------------------------------------------------------


Elle a assisté trois fois au concert symphonique de SQL en décembre
2017, comme Jeremy nous l'a indiqué. La commanditaire de ce meurtre est
donc Miranda Priestly.

Je vais vérifer ca avec cette commande :

``` sql
INSERT INTO solution VALUES (1, 'Miranda Priestly');
```

Le résultat que j'obtiens conclue cette enquête : "Congrats, you found
the brains behind the murder! Everyone in SQL City hails you as the
greatest SQL detective of all time. Time to break out the champagne!"