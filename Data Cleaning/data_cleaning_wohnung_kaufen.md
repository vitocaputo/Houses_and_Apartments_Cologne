# DATA CLEANING

## WOHNUNG_KAUFEN table 
---
# 1. Explore the table structure

---
```sql
SELECT *
FROM wohnung_kaufen
```
| link                                   | anzeige                                                                                     | wohnfläche | zimmer | ort                        | kaufpreis | hausgeld | wohnungslage | baujahr | effizienzklasse | makler          |
|----------------------------------------|---------------------------------------------------------------------------------------------|------------|--------|----------------------------|-----------|----------|--------------|---------|-----------------|-----------------|
| https://www.immowelt.de/expose/2uwpa4n | Die perfekte Größe! 2-Zimmer-Wohnung als Kapitalanlage in der Liebigstr. 161-163 - WE 11    | 48         | 2      | 50825 Köln  (Neuehrenfeld) | 279000    | 138      | 3. Geschoss  | 1957    | F               | GLOBAL-ACT GmbH |
| https://www.immowelt.de/expose/2u4yc4n | Mein Veedel !!! 2-Zimmer-Wohnung als Kapitalanlage zu verkaufen - Karolingerring 19, WE 14  | 43         | 2      | 50678 Köln  (Neustadt-Süd) | 265000    |          | 5. Geschoss  | 1959    | E               | GLOBAL-ACT GmbH |
| https://www.immowelt.de/expose/2v4gx4x | Wie für mich gemacht! vermietete 1-Zimmer Wohnung zu verkaufen ( WE 1 )                     | 26         | 1      | 50676 Köln  (Altstadt-Süd) | 169000    | 142      | Erdgeschoss  | 2018    | C               | GLOBAL-ACT GmbH |
| https://www.immowelt.de/expose/2w8ck4q | Die perfekte Größe! vermiete 2-Zimmer-Wohnung in der Liebigstr. 161-163 zu verkaufen! WE 15 | 41         | 2      | 50823 Köln  (Neuehrenfeld) | 249000    | 124      | 3. Geschoss  | 2020    | E               | GLOBAL-ACT GmbH |
| https://www.immowelt.de/expose/2yr7j4f | FRESH, HIP UND BUNT - Kapitalanlage- Hansemannstr. 16, Köln-Ehrenfeld WE 1                  | 26         | 2      | 50823 Köln  (Ehrenfeld)    | 169000    | 157      | 1. Geschoss  | 1950    | C               | GLOBAL-ACT GmbH |

```sql
-- Number of ads for apartments in sale 

SELECT count(anzeige) as num_ads
FROM wohnung_kaufen
```
| num_ads |
|---------|
| 752     |

---

# 2. Manage duplicates

---

```sql
SELECT 
    total_ads,
    unique_ads,
    total_ads - unique_ads as potential_duplicates
FROM (
    SELECT 
        COUNT(anzeige) as total_ads,
        COUNT(DISTINCT anzeige) as unique_ads
    FROM
        wohnung_kaufen) 
    dupl;
```
| total_ads | unique_ads | potential_duplicates |
|-----------|------------|----------------------|
| 752       | 705        | 47                   |

```sql
-- Insered the column with ref_num

ALTER TABLE wohnung_kaufen
ADD COLUMN ref_num VARCHAR(255);
```
```sql
UPDATE wohnung_kaufen
SET ref_num = SUBSTRING(link, LENGTH('https://www.immowelt.de/expose/') + 1);
```
```sql
ALTER TABLE "immo_köln"."wohnung_kaufen" 
CHANGE COLUMN "ref_num" "ref_num" VARCHAR(255) NULL DEFAULT NULL AFTER "link";
```
```sql
-- it seems to be that 47 ads are duplicates, tried to understand if these are all to remove

SELECT *
FROM wohnung_kaufen
WHERE anzeige IN (
    SELECT anzeige
    FROM wohnung_kaufen
    GROUP BY anzeige
    HAVING COUNT(*) > 1
)
ORDER BY anzeige ASC;
```
| link                                   | ref_num | anzeige                                                                                                                                               | wohnfläche | zimmer   | ort                         | kaufpreis | hausgeld | wohnungslage | baujahr | effizienzklasse | makler                                                |
|----------------------------------------|---------|-------------------------------------------------------------------------------------------------------------------------------------------------------|------------|----------|-----------------------------|-----------|----------|--------------|---------|-----------------|-------------------------------------------------------|
| https://www.immowelt.de/expose/2c6j75d | 2c6j75d | **Wohnkomfort wie nie zuvor**                                                                                                                         | 70         | 3        | 50735 Köln  (Niehl)         | 329000    |          |              |         |                 |                                                       |
| https://www.immowelt.de/expose/2ccw659 | 2ccw659 | **Wohnkomfort wie nie zuvor**                                                                                                                         | 96         | 3        | 50858 Köln  (Junkersdorf)   | 735000    |          |              |         | B               |                                                       |
| https://www.immowelt.de/expose/2bv7l5p | 2bv7l5p | *Offene Beratung am So., den 12.11. von 12:00-12:30 Uhr, Georg-Zapf-Straße 2a in 51061 Köln*                                                          | 59         | 2        | 51061 Köln  (Flittard)      | 339900    | 5742     | Erdgeschoss  | 2024    | A+              | CLOUDBERRY Real Estate GmbH                           |
| https://www.immowelt.de/expose/2by7l5p | 2by7l5p | *Offene Beratung am So., den 12.11. von 12:00-12:30 Uhr, Georg-Zapf-Straße 2a in 51061 Köln*                                                          | 58         | 2        | 51061 Köln  (Flittard)      | 336900    | 5849     | Erdgeschoss  | 2024    | A+              | CLOUDBERRY Real Estate GmbH                           |
| https://www.immowelt.de/expose/2bu8l5p | 2bu8l5p | *Offene Beratung am So., den 12.11. von 12:00-12:30 Uhr, Georg-Zapf-Straße 2a in 51061 Köln*                                                          | 56         | 2        | 51061 Köln  (Flittard)      | 336900    | 6027     | 1. Geschoss  | 2024    | A+              | CLOUDBERRY Real Estate GmbH                           |
| https://www.immowelt.de/expose/2bx9l5p | 2bx9l5p | *Offene Beratung am So., den 12.11. von 12:00-12:30 Uhr, Georg-Zapf-Straße 2a in 51061 Köln*                                                          | 78         | 4        | 51061 Köln  (Flittard)      | 469900    | 6017     | 1. Geschoss  | 2024    | A+              | CLOUDBERRY Real Estate GmbH                           |
| https://www.immowelt.de/expose/2cmqt5c | 2cmqt5c | + 2-Zimmer-Wohnung mit Terrasse +                                                                                                                     | 65         | 2        | 51149 Köln  (Westhoven)     | 180000    |          |              |         |                 |                                                       |
| https://www.immowelt.de/expose/2cmhx5c | 2cmhx5c | + 2-Zimmer-Wohnung mit Terrasse +                                                                                                                     | 65         | 2        | 51149 Köln  (Westhoven)     | 180000    |          |              |         |                 | AZ Agentur für Zwangsversteigerungsinformationen GmbH |
| https://www.immowelt.de/expose/2c6wt5c | 2c6wt5c | + 3-Zimmer-Wohnung mit TG-Stellplatz +                                                                                                                | 95         | 3        | 50999 Köln  (Sürth)         | 404000    |          |              |         |                 |                                                       |
| https://www.immowelt.de/expose/2cbhx5c | 2cbhx5c | + 3-Zimmer-Wohnung mit TG-Stellplatz +                                                                                                                | 95         | 3        | 50999 Köln  (Sürth)         | 404000    |          |              |         |                 | AZ Agentur für Zwangsversteigerungsinformationen GmbH |
| https://www.immowelt.de/expose/2cbmt5c | 2cbmt5c | 2-Zimmer-Wohnung + provisionsfrei +                                                                                                                   | 42         | 2        | 50823 Köln  (Neuehrenfeld)  | 140000    |          |              |         |                 |                                                       |
| https://www.immowelt.de/expose/2c5hx5c | 2c5hx5c | 2-Zimmer-Wohnung + provisionsfrei +                                                                                                                   | 42         | 2        | 50823 Köln  (Neuehrenfeld)  | 140000    |          |              |         |                 | AZ Agentur für Zwangsversteigerungsinformationen GmbH |
| https://www.immowelt.de/expose/2bwsn56 | 2bwsn56 | 3 Zimmer Wohnung mit Loggia in Köln-Weidenpesch OHNE KÄUFERPROVISION                                                                                  | 79         | 3        | 50737 Köln  (Weidenpesch)   | 232000    | 2937     | 5. Geschoss  | 1974    | E               | Vonovia SE-Selbstständiger Vertriebspartner           |
| https://www.immowelt.de/expose/2bxsn56 | 2bxsn56 | 3 Zimmer Wohnung mit Loggia in Köln-Weidenpesch OHNE KÄUFERPROVISION                                                                                  | 79         | 3        | 50737 Köln  (Weidenpesch)   | 232000    | 2937     | 8. Geschoss  | 1974    | E               | Vonovia SE-Selbstständiger Vertriebspartner           |
| https://www.immowelt.de/expose/2cxut5c | 2cxut5c | 3-Zimmer-Wohnung - provisionsfrei                                                                                                                     | 72         | 3        | 51063 Köln  (Mülheim)       | 205000    |          | Dachgeschoss |         |                 |                                                       |
| https://www.immowelt.de/expose/2cchx5c | 2cchx5c | 3-Zimmer-Wohnung - provisionsfrei                                                                                                                     | 72         | 3        | 51063 Köln  (Mülheim)       | 205000    |          | Dachgeschoss |         |                 | AZ Agentur für Zwangsversteigerungsinformationen GmbH |
| https://www.immowelt.de/expose/2crqt5c | 2crqt5c | 3-Zimmer-Wohnung mit Balkon + provisionsfrei +                                                                                                        | 62         | 3        | 51147 Köln  (Grengel)       | 190000    |          |              |         |                 |                                                       |
| https://www.immowelt.de/expose/2ckhx5c | 2ckhx5c | 3-Zimmer-Wohnung mit Balkon + provisionsfrei +                                                                                                        | 62         | 3        | 51147 Köln  (Grengel)       | 190000    |          |              |         |                 | AZ Agentur für Zwangsversteigerungsinformationen GmbH |
| https://www.immowelt.de/expose/2cdyt5c | 2cdyt5c | 4-Zimmer-Wohnung mit Garten und Garage + provisionsfrei +                                                                                             | 156        | 4        | 50935 Köln  (Lindenthal)    | 1285000   |          | Erdgeschoss  |         |                 |                                                       |
| https://www.immowelt.de/expose/2c8hx5c | 2c8hx5c | 4-Zimmer-Wohnung mit Garten und Garage + provisionsfrei +                                                                                             | 156        | 4        | 50935 Köln  (Lindenthal)    | 1285000   |          | Erdgeschoss  |         |                 | AZ Agentur für Zwangsversteigerungsinformationen GmbH |
| https://www.immowelt.de/expose/2c8tt5c | 2c8tt5c | 5-Zimmer-Wohnung mit Loggia + provisionsfrei +                                                                                                        | 90         | 5        | 51149 Köln  (Porz)          | 225000    |          |              |         |                 |                                                       |
| https://www.immowelt.de/expose/2cnhx5c | 2cnhx5c | 5-Zimmer-Wohnung mit Loggia + provisionsfrei +                                                                                                        | 90         | 5        | 51149 Köln  (Porz)          | 225000    |          |              |         |                 | AZ Agentur für Zwangsversteigerungsinformationen GmbH |
| https://www.immowelt.de/expose/2bt6c56 | 2bt6c56 | Barrierefreie Eigentumswohnung für die ganze Familie mit großem Bad und Loggia                                                                        | 102        | 4        | 50737 Köln  (Weidenpesch)   | 569000    |          | 3. Geschoss  | 2023    |                 | Bonava Deutschland GmbH                               |
| https://www.immowelt.de/expose/2b9fb5r | 2b9fb5r | Barrierefreie Eigentumswohnung für die ganze Familie mit großem Bad und Loggia                                                                        | 90         | 4        | 50737 Köln  (Weidenpesch)   | 529000    |          | 2. Geschoss  | 2023    |                 | Bonava Deutschland GmbH                               |
| https://www.immowelt.de/expose/2bbq956 | 2bbq956 | Barrierefreie Eigentumswohnung mit offenem Wohn- und Küchenbereich sowie Loggia                                                                       | 75         | 3        | 50737 Köln  (Weidenpesch)   | 429000    |          | 1. Geschoss  | 2023    |                 | Bonava Deutschland GmbH                               |
| https://www.immowelt.de/expose/2bss956 | 2bss956 | Barrierefreie Eigentumswohnung mit offenem Wohn- und Küchenbereich sowie Loggia                                                                       | 79         | 3        | 50737 Köln  (Weidenpesch)   | 459000    |          | 2. Geschoss  | 2023    |                 | Bonava Deutschland GmbH                               |
| https://www.immowelt.de/expose/2b25a56 | 2b25a56 | Barrierefreie Eigentumswohnung mit offenem Wohn- und Küchenbereich sowie Loggia                                                                       | 79         | 3        | 50737 Köln  (Weidenpesch)   | 489000    |          | 3. Geschoss  | 2023    |                 | Bonava Deutschland GmbH                               |
| https://www.immowelt.de/expose/2c3gy5d | 2c3gy5d | Dachgeschosswohnung in 51063 Köln, Berliner Str.                                                                                                      | 72         | 3        | 51063 Köln  (Mülheim)       | 205000    |          | Erdgeschoss  | 1920    |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2cacs5d | 2cacs5d | Dachgeschosswohnung in 51063 Köln, Berliner Str.                                                                                                      | 72         | 3        | 51063 Köln  (Mülheim)       | 205000    |          | Erdgeschoss  | 1920    |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2bvqz5t | 2bvqz5t | Doppeltes Glück: 2 Bäder & 4 Zimmer, Gartenhaus und Riesenkeller, STP                                                                                 | 82         | 4        | 51143 Köln  (Zündorf)       | 254000    | 390      | 1. Geschoss  | 1968    | F               | Immobiehler e.K.                                      |
| https://www.immowelt.de/expose/2bsl35u | 2bsl35u | Doppeltes Glück: 2 Bäder & 4 Zimmer, Gartenhaus und Riesenkeller, STP                                                                                 | 82         | 4        | 51143 Köln  (Zündorf)       | 254000    | 390      | 1. Geschoss  | 1968    | F               | Immobiehler e.K.                                      |
| https://www.immowelt.de/expose/2bvgu5t | 2bvgu5t | Eigentumswohnung in 51149 Köln                                                                                                                        | 90         |   k.A.   | 51149 Köln  (Porz)          | 225000    |          |              |         |                 | Dein-ImmoCenter                                       |
| https://www.immowelt.de/expose/2b6sa5p | 2b6sa5p | Eigentumswohnung in 51149 Köln                                                                                                                        | 65         |   k.A.   | 51149 Köln  (Westhoven)     | 180000    |          |              |         |                 | Dein-ImmoCenter                                       |
| https://www.immowelt.de/expose/2ctqt5c | 2ctqt5c | Einfamilien-Doppelhaushälfte mit Terrasse                                                                                                             | 122        | 4        | 51069 Köln  (Dellbrück)     | 560000    |          |              |         |                 |                                                       |
| https://www.immowelt.de/expose/2cehx5c | 2cehx5c | Einfamilien-Doppelhaushälfte mit Terrasse                                                                                                             | 122        | 4        | 51069 Köln  (Dellbrück)     | 560000    |          |              |         |                 | AZ Agentur für Zwangsversteigerungsinformationen GmbH |
| https://www.immowelt.de/expose/2c37q5b | 2c37q5b | Erdgeschosswohnung in 50935 Köln, Viktor-Schnitzler-Str.                                                                                              | 156        | 4        | 50935 Köln  (Lindenthal)    | 1285000   |          | Erdgeschoss  | 1953    |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2cnsr5b | 2cnsr5b | Erdgeschosswohnung in 50935 Köln, Viktor-Schnitzler-Str.                                                                                              | 156        | 4        | 50935 Köln  (Lindenthal)    | 1285000   |          | Erdgeschoss  | 1953    |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2cv775c | 2cv775c | Etagenwohnung in 50735 Köln, An der Schanz                                                                                                            | 82         | 3        | 50735 Köln  (Riehl)         | 145000    |          | 10. Geschoss |         |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2cdx95c | 2cdx95c | Etagenwohnung in 50735 Köln, An der Schanz                                                                                                            | 82         | 3        | 50735 Köln  (Riehl)         | 145000    |          | 10. Geschoss |         |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2c8xj5a | 2c8xj5a | Etagenwohnung in 50823 Köln, Graeffstr.                                                                                                               | 42         | 2        | 50823 Köln  (Neuehrenfeld)  | 140000    |          | 30. Geschoss | 1973    |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2cb6c5a | 2cb6c5a | Etagenwohnung in 50823 Köln, Graeffstr.                                                                                                               | 42         | 2        | 50823 Köln  (Neuehrenfeld)  | 140000    |          | 30. Geschoss | 1973    |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2c8k25d | 2c8k25d | Etagenwohnung in 50933 Köln, Voigtelstr.                                                                                                              | 143        | 3        | 50933 Köln  (Braunsfeld)    | 949000    |          | 3. Geschoss  |         |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2c3f85d | 2c3f85d | Etagenwohnung in 50933 Köln, Voigtelstr.                                                                                                              | 143        | 3        | 50933 Köln  (Braunsfeld)    | 949000    |          | 3. Geschoss  |         |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2cbnt59 | 2cbnt59 | Etagenwohnung in 50999 Köln, Sürther Hauptstr.                                                                                                        | 95         | 3        | 50999 Köln  (Sürth)         | 404000    |          | 1. Geschoss  | 1997    |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2cy6z59 | 2cy6z59 | Etagenwohnung in 50999 Köln, Sürther Hauptstr.                                                                                                        | 95         | 3        | 50999 Köln  (Sürth)         | 404000    |          | 1. Geschoss  | 1997    |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2cr6g5c | 2cr6g5c | Etagenwohnung in 51147 Köln, Akazienweg                                                                                                               | 62         | 3        | 51147 Köln  (Grengel)       | 190000    |          | 2. Geschoss  | 1972    |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2cyum5c | 2cyum5c | Etagenwohnung in 51147 Köln, Akazienweg                                                                                                               | 62         | 3        | 51147 Köln  (Grengel)       | 190000    |          | 2. Geschoss  | 1972    |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2crf75c | 2crf75c | Etagenwohnung in 51149 Köln, Konrad-Adenauer-Str.                                                                                                     | 90         | 5        | 51149 Köln  (Porz)          | 225000    |          | 3. Geschoss  | 1969    |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2cp8a5c | 2cp8a5c | Etagenwohnung in 51149 Köln, Konrad-Adenauer-Str.                                                                                                     | 90         | 5        | 51149 Köln  (Porz)          | 225000    |          | 3. Geschoss  | 1969    |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2cjma5c | 2cjma5c | Etagenwohnung in 51149 Köln, Nikolausstr.                                                                                                             | 65         | 3        | 51149 Köln  (Westhoven)     | 180000    |          | 1. Geschoss  | 1974    |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2chkg5c | 2chkg5c | Etagenwohnung in 51149 Köln, Nikolausstr.                                                                                                             | 65         | 3        | 51149 Köln  (Westhoven)     | 180000    |          | 1. Geschoss  | 1974    |                 | Argetra GmbH                                          |
| https://www.immowelt.de/expose/2csqt5c | 2csqt5c | Große 3-Zimmer-Wohnung mit Balkon und Garage + provisionsfrei +                                                                                       | 143        | 3        | 50933 Köln  (Braunsfeld)    | 949000    |          |              |         |                 |                                                       |
| https://www.immowelt.de/expose/2c7hx5c | 2c7hx5c | Große 3-Zimmer-Wohnung mit Balkon und Garage + provisionsfrei +                                                                                       | 143        | 3        | 50933 Köln  (Braunsfeld)    | 949000    |          |              |         |                 | AZ Agentur für Zwangsversteigerungsinformationen GmbH |
| https://www.immowelt.de/expose/2z7k64d | 2z7k64d | KÖLN-LIVE- Kapitalanlage im Mauritiuswall 33! ( WE 8 )                                                                                                | 62         | 3        | 50676 Köln  (Altstadt-Süd)  | 403260    | 225      | 3. Geschoss  | 2020    | D               | GLOBAL-ACT GmbH                                       |
| https://www.immowelt.de/expose/272pz5f | 272pz5f | KÖLN-LIVE- Kapitalanlage im Mauritiuswall 33! ( WE 8 )                                                                                                | 62         | 3        | 50676 Köln  (Altstadt-Süd)  | 403260    |          | 3. Geschoss  | 1966    |                 | GLOBAL-ACT GmbH                                       |
| https://www.immowelt.de/expose/29ck65m | 29ck65m | meine.liebe - von Herzen Wohnen in Porz/Zündorf                                                                                                       | 107        | 3        | 51143 Köln  (Zündorf)       | 369000    | 3440     |              |         | D               | Volksbank Rhein - Erft - Köln eG                      |
| https://www.immowelt.de/expose/2afl353 | 2afl353 | meine.liebe - von Herzen Wohnen in Porz/Zündorf                                                                                                       | 32         | 1        | 51143 Köln  (Zündorf)       | 119000    | 3665     |              |         | E               | Volksbank Rhein - Erft - Köln eG                      |
| https://www.immowelt.de/expose/2abl353 | 2abl353 | meine.liebe - von Herzen Wohnen in Porz/Zündorf                                                                                                       | 87         | 3        | 51143 Köln  (Zündorf)       | 339000    | 3892     |              |         | E               | Volksbank Rhein - Erft - Köln eG                      |
| https://www.immowelt.de/expose/2ael353 | 2ael353 | meine.liebe - von Herzen Wohnen in Porz/Zündorf                                                                                                       | 85         | 3        | 51143 Köln  (Zündorf)       | 299000    | 3533     |              |         | E               | Volksbank Rhein - Erft - Köln eG                      |
| https://www.immowelt.de/expose/2acl353 | 2acl353 | meine.liebe - von Herzen Wohnen in Porz/Zündorf                                                                                                       | 49         | 2        | 51143 Köln  (Zündorf)       | 179000    | 3678     |              |         | E               | Volksbank Rhein - Erft - Köln eG                      |
| https://www.immowelt.de/expose/29mk65m | 29mk65m | meine.liebe - von Herzen Wohnen in Porz/Zündorf                                                                                                       | 93         | 2        | 51143 Köln  (Zündorf)       | 359000    | 3874     |              |         | D               | Volksbank Rhein - Erft - Köln eG                      |
| https://www.immowelt.de/expose/29kk65m | 29kk65m | meine.liebe - von Herzen Wohnen in Porz/Zündorf                                                                                                       | 127        | 3        | 51143 Köln  (Zündorf)       | 469000    | 3702     |              |         | D               | Volksbank Rhein - Erft - Köln eG                      |
| https://www.immowelt.de/expose/2ahl353 | 2ahl353 | meine.liebe - von Herzen Wohnen in Porz/Zündorf                                                                                                       | 94         | 4        | 51143 Köln  (Zündorf)       | 359000    | 3815     |              |         | E               | Volksbank Rhein - Erft - Köln eG                      |
| https://www.immowelt.de/expose/293k65m | 293k65m | meine.liebe - von Herzen Wohnen in Porz/Zündorf                                                                                                       | 171        | 4        | 51143 Köln  (Zündorf)       | 599000    | 3493     |              |         | D               | Volksbank Rhein - Erft - Köln eG                      |
| https://www.immowelt.de/expose/2bzwb56 | 2bzwb56 | Moderne, barrierefreie Eigentumswohnung mit großer Dachterrasse im Simonsveedel                                                                       | 45         | 2        | 50737 Köln  (Weidenpesch)   | 299000    |          | 2. Geschoss  | 2023    |                 | Bonava Deutschland GmbH                               |
| https://www.immowelt.de/expose/2badc56 | 2badc56 | Moderne, barrierefreie Eigentumswohnung mit großer Dachterrasse im Simonsveedel                                                                       | 77         | 3        | 50737 Köln  (Weidenpesch)   | 479000    |          | 3. Geschoss  | 2023    |                 | Bonava Deutschland GmbH                               |
| https://www.immowelt.de/expose/27rwz5f | 27rwz5f | Natürlich Köln- 3-Zimmer-Wohnung als KAPITALANLAGE zu verkaufen! WE 1                                                                                 | 67         | 3        | 50827 Köln  (Bickendorf)    | 366410    | 281      | 1. Geschoss  | 1966    |                 | GLOBAL-ACT GmbH                                       |
| https://www.immowelt.de/expose/29q6856 | 29q6856 | Natürlich Köln- 3-Zimmer-Wohnung als KAPITALANLAGE zu verkaufen! WE 1                                                                                 | 67         | 3        | 50827 Köln  (Bickendorf)    | 366410    | 281      |              | 1966    | E               | GLOBAL-ACT GmbH                                       |
| https://www.immowelt.de/expose/2c8t45c | 2c8t45c | Privatverkauf: Renditeobjekt aus Eigenbestand, Provision frei; vermietete 4-Zimmer-Eigentumswohnung mit gr. Loggia, Garage, Pkw-Stplz in Köln-Ostheim | 83         | 4        | 51107 Köln  (Ostheim)       | 284000    | 478      | 5. Geschoss  | 1974    | F               | Pars Immobilien Cologne                               |
| https://www.immowelt.de/expose/2cylx5d | 2cylx5d | Privatverkauf: Renditeobjekt aus Eigenbestand, Provision frei; vermietete 4-Zimmer-Eigentumswohnung mit gr. Loggia, Garage, Pkw-Stplz in Köln-Ostheim | 83         | 4        | 51107 Köln  (Ostheim)       | 284000    | 478      | 5. Geschoss  | 1974    | F               | Pars Immobilien Cologne                               |
| https://www.immowelt.de/expose/2bxct5z | 2bxct5z | RATHENAUPLATZ / KWARTIER LATÄNG -- MITTEN IM VEEDEL: 3 - Zimmerwohnung mit Balkon                                                                     | 92         | 3        | 50672 Köln  (Neustadt-Nord) | 430000    | 249      | 2. Geschoss  | 1900    | B               | MERZENICH Immobilien GmbH                             |
| https://www.immowelt.de/expose/2bvbt5z | 2bvbt5z | RATHENAUPLATZ / KWARTIER LATÄNG -- MITTEN IM VEEDEL: 3 - Zimmerwohnung mit Balkon                                                                     | 92         | 3        | 50672 Köln  (Neustadt-Nord) | 430000    | 249      | 2. Geschoss  | 1900    | B               | MERZENICH Immobilien GmbH                             |
| https://www.immowelt.de/expose/27j255h | 27j255h | Riehler Ruheterrasse am Zoo!                                                                                                                          | 88         | 3        | 50735 Köln / Riehl          | 409000    | 140      |              |         | C               | Immobiehler e.K.                                      |
| https://www.immowelt.de/expose/27kgl5k | 27kgl5k | Riehler Ruheterrasse am Zoo!                                                                                                                          | 88         | 3        | 50735 Köln / Riehl          | 409000    | 140      |              |         | C               | Immobiehler e.K.                                      |
| https://www.immowelt.de/expose/2cvnf57 | 2cvnf57 | Strahlende Oase: Begehrt in der Altstadt-Süd mit Stellplatz                                                                                           | 26         | 1        | 50678 Köln  (Altstadt-Süd)  | 169000    | 152      |              |         |                 | Immobiehler e.K.                                      |
| https://www.immowelt.de/expose/2crmf57 | 2crmf57 | Strahlende Oase: Begehrt in der Altstadt-Süd mit Stellplatz                                                                                           | 26         | 1        | 50678 Köln  (Altstadt-Süd)  | 169000    | 152      |              |         |                 | Immobiehler e.K.                                      |
| https://www.immowelt.de/expose/295eq5h | 295eq5h | vermietete Wohnung mit Balkon - provisionsfrei                                                                                                        | 57         | 2        | 51149 Köln  (Gremberghoven) | 167100    | 394      | 2. Geschoss  | 1957    | E               | Vonovia SE-Selbstständiger Vertriebspartner           |
| https://www.immowelt.de/expose/2bem756 | 2bem756 | vermietete Wohnung mit Balkon - provisionsfrei                                                                                                        | 60         | 2        | 51103 Köln  (Höhenberg)     | 150000    | 380      | 3. Geschoss  | 1966    | F               | Vonovia SE-Selbstständiger Vertriebspartner           |
| https://www.immowelt.de/expose/2blmz5j | 2blmz5j | vermietete Wohnung mit Balkon - provisionsfrei                                                                                                        | 66         | 2        | 51065 Köln  (Mülheim)       | 179000    | 330      | Erdgeschoss  | 1963    | C               | Vonovia SE-Selbstständiger Vertriebspartner           |
| https://www.immowelt.de/expose/2c6qs5c | 2c6qs5c | vermietete Wohnung mit Balkon - provisionsfrei                                                                                                        | 40         | 2        | 50823 Köln  (Neuehrenfeld)  | 150000    | 2019     | Erdgeschoss  | 1953    | A+              | Vonovia SE-Selbstständiger Vertriebspartner           |
| https://www.immowelt.de/expose/2b6xl5d | 2b6xl5d | Vermietetes Apartement mit Stellplatz!                                                                                                                | 33         | 1        | 50737 Köln  (Longerich)     | 114800    | 80       | 1. Geschoss  | 1967    | D               | S Immobilienpartner GmbH                              |
| https://www.immowelt.de/expose/2b5xl5d | 2b5xl5d | Vermietetes Apartement mit Stellplatz!                                                                                                                | 33         | 1        | 50737 Köln  (Longerich)     | 118800    | 75       | Erdgeschoss  | 1967    | D               | S Immobilienpartner GmbH                              |

```sql
-- Not all the duplicates are to remove. Some have the same title but are a part of a build with different apartments.
-- They have different living area, price or floor

DELETE FROM wohnung_kaufen
WHERE ref_num IN("2cmqt5c", "2c6wt5c", "2cbmt5c", "2cxut5c",
                "2crqt5c", "2cdyt5c", "2c8tt5c", "2c3gy5d",
                "2bsl35u", "2ctqt5c", "2cnsr5b", "2cv775c",
                "2c8xj5a", "2c8k25d", "2cbnt59", "2cr6g5c",
                "2crf75c", "2cjma5c", "2csqt5c", "272pz5f",
                "29q6856", "2c8t45c", "2bxct5z", "27j255h", "2cvnf57")
```
---

# 3. Manage missing values

---
```sql
SELECT count(anzeige) as null_wohnfläche
FROM wohnung_kaufen
WHERE wohnfläche = 0
```

| null_wohnfläche |
|-----------------|
| 1               |

```sql
-- Identified the number of null values in the column "wohnfläche"
SELECT *
FROM wohnung_kaufen
WHERE wohnfläche = 0
```

| link                                   | ref_num | anzeige                     | wohnfläche | zimmer | ort                   | kaufpreis | hausgeld | wohnungslage | baujahr | effizienzklasse | makler |
|----------------------------------------|---------|-----------------------------|------------|--------|-----------------------|-----------|----------|--------------|---------|-----------------|--------|
| https://www.immowelt.de/expose/2czsn57 | 2czsn57 | Ihre neue 2-Zimmer-Wohnung! | k.A.       | 2      | 51065 Köln  (Mülheim) | 219000    | 180      | 3. Geschoss  | 1954    | C               |        |

```sql
-- Searched in the ad the information about living area
-- Found that the apartment has a living area of 62 sm

UPDATE wohnung_kaufen
SET wohnfläche = 62
WHERE ref_num = "2czsn57"
```
```sql
-- Identified the number of null values in the column "zimmer"

SELECT count(anzeige) as null_zimmer
FROM wohnung_kaufen
WHERE zimmer = 0
```

| null_zimmer |
|-------------|
| 13          |


```sql
SELECT *
FROM wohnung_kaufen
WHERE zimmer = 0
```

| link                                   | ref_num | anzeige                                                                                  | wohnfläche | zimmer | ort                        | kaufpreis | hausgeld | wohnungslage | baujahr | effizienzklasse | makler                                      |
|----------------------------------------|---------|------------------------------------------------------------------------------------------|------------|--------|----------------------------|-----------|----------|--------------|---------|-----------------|---------------------------------------------|
| https://www.immowelt.de/expose/27z3n5f | 27z3n5f | Stilvoll! Vermietete 3-Zimmer-Wohnung als Kapitalanlage zu verkaufen! ( WE 5 )           | 65         | k.A.   | 50674 Köln  (Neustadt-Süd) | 391560    | 5041     | 6. Geschoss  |         |                 | GLOBAL-ACT GmbH                             |
| https://www.immowelt.de/expose/27dts5f | 27dts5f | Sinfonie der Großstadt! Vermietete 3-Zimmer-Wohnung als Kapitalanlage zu verkaufen! WE 8 | 68         | k.A.   | 50674 Köln  (Neustadt-Süd) | 399000    |          | 3. Geschoss  |         |                 | GLOBAL-ACT GmbH                             |
| https://www.immowelt.de/expose/2ap3n5v | 2ap3n5v | Interessante Kapitalanlage im 4 Sterne Hotel Mercure in Köln!                            | 60         | k.A.   | 50676 Köln  (Altstadt-Süd) | 195000    |          |              | 1989    | D               | Immo Projekte P2 GmbH                       |
| https://www.immowelt.de/expose/2ban958 | 2ban958 | Handwerker aufgepasst, gut aufgeteilte 3 Zimmer Wohnung                                  | 66         | k.A.   | 51147 Köln  (Wahnheide)    | 169500    | 316      | Erdgeschoss  | 1963    |                 | Klaus-Peter Heine Immobilienverwaltung GmbH |
| https://www.immowelt.de/expose/2bgxe58 | 2bgxe58 | Eigentumswohnung in 50823 Köln                                                           | 42         | k.A.   | 50823 Köln  (Neuehrenfeld) | 140000    |          |              |         |                 | Dein-ImmoCenter                             |
| https://www.immowelt.de/expose/2bpjw5m | 2bpjw5m | Eigentumswohnung in 50735 Köln                                                           | 82         | k.A.   | 50735 Köln  (Riehl)        | 145000    |          |              |         |                 | Dein-ImmoCenter                             |
| https://www.immowelt.de/expose/2b6sa5p | 2b6sa5p | Eigentumswohnung in 51149 Köln                                                           | 65         | k.A.   | 51149 Köln  (Westhoven)    | 180000    |          |              |         |                 | Dein-ImmoCenter                             |
| https://www.immowelt.de/expose/2b4fw5p | 2b4fw5p | Eigentumswohnung in 50933 Köln                                                           | 143        | k.A.   | 50933 Köln  (Braunsfeld)   | 949000    |          |              |         |                 | Dein-ImmoCenter                             |
| https://www.immowelt.de/expose/2b2gw5p | 2b2gw5p | Eigentumswohnung in 51147 Köln                                                           | 62         | k.A.   | 51147 Köln  (Grengel)      | 190000    |          |              |         |                 | Dein-ImmoCenter                             |
| https://www.immowelt.de/expose/2bvgu5t | 2bvgu5t | Eigentumswohnung in 51149 Köln                                                           | 90         | k.A.   | 51149 Köln  (Porz)         | 225000    |          |              |         |                 | Dein-ImmoCenter                             |
| https://www.immowelt.de/expose/2bw5t5v | 2bw5t5v | Eigentumswohnung in 51063 Köln                                                           | 72         | k.A.   | 51063 Köln  (Mülheim)      | 205000    |          |              |         |                 | Dein-ImmoCenter                             |
| https://www.immowelt.de/expose/2b6zx5x | 2b6zx5x | Eigentumswohnung in 50999 Köln                                                           | 95         | k.A.   | 50999 Köln  (Sürth)        | 404000    |          |              |         |                 | Dein-ImmoCenter                             |
| https://www.immowelt.de/expose/2cy6h53 | 2cy6h53 | Eigentumswohnung in 50935 Köln                                                           | 156        | k.A.   | 50935 Köln  (Lindenthal)   | 1285000   |          |              |         |                 | Dein-ImmoCenter                             |


Information in the title of the ad:
27z3n5f -> 3
27dts5f -> 3
2ban958 -> 3

Information following the link:
2bgxe58	-> 2
2bpjw5m -> 3
2b6sa5p -> 2
2b4fw5p -> 3
2b2gw5p -> 2
2bvgu5t -> 3
2bw5t5v -> 3
2b6zx5x -> 3
2cy6h53 -> 3

```sql
UPDATE wohnung_kaufen
SET zimmer = 2
WHERE ref_num IN("2bgxe58", "2b6sa5p", "2b2gw5p")
```
```sql
UPDATE wohnung_kaufen
SET zimmer = 3
WHERE ref_num IN("27z3n5f", "27dts5f", "2ban958", "2bpjw5m", "2b4fw5p", 
                "2bvgu5t", "2bw5t5v", "2b6zx5x", "2cy6h53")

```
```sql
SELECT *
FROM wohnung_kaufen
WHERE zimmer = 0
```

| link                                   | ref_num | anzeige                                                       | wohnfläche | zimmer | ort                        | kaufpreis | hausgeld | wohnungslage | baujahr | effizienzklasse | makler                |
|----------------------------------------|---------|---------------------------------------------------------------|------------|--------|----------------------------|-----------|----------|--------------|---------|-----------------|-----------------------|
| https://www.immowelt.de/expose/2ap3n5v | 2ap3n5v | Interessante Kapitalanlage im 4 Sterne Hotel Mercure in Köln! | 60         | k.A.   | 50676 Köln  (Altstadt-Süd) | 195000    |          |              | 1989    | D               | Immo Projekte P2 GmbH |

```sql
-- It remains only an ad that refers to an hotel room. 
-- Due to the nature of the offer, provided to delete it

DELETE 
FROM wohnung_kaufen
WHERE ref_num = "2ap3n5v"
```
```sql
-- Identified the number of null values in the column "ort"

SELECT count(anzeige) as null_ort
FROM wohnung_kaufen
WHERE ort = ""
```

| null_ort |
|----------|
| 0        |

```sql
-- Identified the number of null values in the column "kaufpreis"
SELECT count(anzeige) as null_kaufpreis
FROM wohnung_kaufen
WHERE kaufpreis = 0
```

| null_kaufpreis |
|----------------|
| 2              |


```sql
SELECT *
FROM wohnung_kaufen
WHERE kaufpreis = 0
```

| link                                   | ref_num | anzeige                                                                             | wohnfläche | zimmer | ort                        | kaufpreis   | hausgeld | wohnungslage               | baujahr | effizienzklasse | makler                        |
|----------------------------------------|---------|-------------------------------------------------------------------------------------|------------|--------|----------------------------|-------------|----------|----------------------------|---------|-----------------|-------------------------------|
| https://www.immowelt.de/expose/2btgf57 | 2btgf57 | Preis auf Anfrage: Apartment in Köln-Nippes, ca. 35 qm, vermietet.                  | 35         | 1      | 50733 Köln  (Nippes)       | auf Anfrage | 170      | 1. Geschoss                | 1993    | D               | Tre Orsetti Cologne e.K.      |
| https://www.immowelt.de/expose/2c5rj5c | 2c5rj5c | Top-Investment: 6,5 % Rendite p.a. mit Eigentumswohnung in Köln-Südstadt, Ubierring | 140        | 4      | 50678 Köln  (Neustadt-Süd) | auf Anfrage |          | 4. Geschoss (Dachgeschoss) | 1906    |                 | Stiftungsberatung Dr. Kade KG |

```sql
-- deleted the 2 rows due to the lack in price (only by request)

DELETE
FROM wohnung_kaufen
WHERE kaufpreis = 0
```
```sql
-- Identified the number of null values in the column "hausgeld"

SELECT count(ref_num) as null_hausgeld
FROM wohnung_kaufen
WHERE hausgeld = 0
```
| null_hausgeld |
|---------------|
| 160           |


```sql
-- Identified the number of null values in the column "makler"

SELECT count(ref_num) as null_makler
FROM wohnung_kaufen
WHERE makler = ""
```
| null_makler |
|-------------|
| 72          |

```sql
-- Set as private offer where the column "makler" is null

UPDATE wohnung_kaufen
SET makler = "Privatanbieter"
WHERE makler = ""
```
```sql
-- Convert the column "wohnfläche, kaufpreis, hausgeld, zimmer as float

ALTER TABLE "immo_köln"."wohnung_kaufen" 
CHANGE COLUMN "wohnfläche" "wohnfläche" FLOAT NULL DEFAULT NULL ;
CHANGE COLUMN "kaufpreis" "kaufpreis" FLOAT NULL DEFAULT NULL ;
CHANGE COLUMN "hausgeld" "hausgeld" FLOAT NULL DEFAULT NULL ;
CHANGE COLUMN "zimmer" "zimmer" FLOAT NULL DEFAULT NULL ;
```
---

# 4. Dealing with outliers

---

```sql
-- Checked the value range of the living area

SELECT 
    MIN(wohnfläche) as min_wf,
    MAX(wohnfläche) as max_wf
FROM wohnung_kaufen
```
| min_wf | max_wf |
|--------|--------|
| 18     | 314    |


```sql
-- Checked the value range of the rooms

SELECT zimmer, 
    count(zimmer) as n_ads
FROM wohnung_kaufen
GROUP BY zimmer
ORDER BY zimmer DESC
```

| zimmer | n_ads |
|--------|-------|
| 7      | 1     |
| 6      | 10    |
| 5      | 19    |
| 4      | 119   |
| 3      | 325   |
| 2      | 196   |
| 1      | 54    |

```sql
-- Checking if in the column ort there are only postal codes of Cologne (doesn´t start with 5)

SELECT *
FROM wohnung_kaufen
WHERE ort NOT LIKE '5%';
```

| link                                   | ref_num | anzeige                                                           | wohnfläche | zimmer | ort                           | kaufpreis | hausgeld | wohnungslage | baujahr | effizienzklasse | makler                  |
|----------------------------------------|---------|-------------------------------------------------------------------|------------|--------|-------------------------------|-----------|----------|--------------|---------|-----------------|-------------------------|
| https://www.immowelt.de/expose/2bzfs5x | 2bzfs5x | Geräumige Wohnung mit großem Balkon in ruhiger Lage ++AB SOFORT++ | 63         | 3      | 70378 Stuttgart  (Marienburg) | 265000    | 491      | 1. Geschoss  | 1988    |                 | Immo-Team GmbH & Co. KG |

```sql
-- The ad refers to an apartment in Stuttgart, for this reason provided to delete it

DELETE FROM wohnung_kaufen
WHERE ref_num = "2bzfs5x"
```
```sql
-- Checked the range of the house prices

SELECT 
    min(kaufpreis) as min_price,
    max(kaufpreis) as max_price
FROM wohnung_kaufen
```
| min_price | max_price |
|-----------|-----------|
| 4878      | 3900000   |

```sql
-- Selected all ads whose price is below 30.000 €

SELECT *
FROM wohnung_kaufen
WHERE kaufpreis < 30000
```
| link                                   | ref_num | anzeige                                                                                                               | wohnfläche | zimmer | ort                | kaufpreis | hausgeld | wohnungslage | baujahr | effizienzklasse | makler        |
|----------------------------------------|---------|-----------------------------------------------------------------------------------------------------------------------|------------|--------|--------------------|-----------|----------|--------------|---------|-----------------|---------------|
| https://www.immowelt.de/expose/2bwpy5d | 2bwpy5d | Rundbogenhalle mit PVC Plane in Weiß Quadratmeterfläche - 91,50m^2 - Tierstall, Industriehalle, Waren/Holz/Strohlager | 92         | 1      | 50937 Köln  (Sülz) | 4878      | 0        |              |         |                 | Covertop GmbH |

```sql
-- The ad refers to a container, for this reason deleted

DELETE FROM wohnung_kaufen
WHERE ref_num = "2bwpy5d"
```

| link                                   | ref_num | anzeige                                                                                | wohnfläche | zimmer | ort                           | kaufpreis | hausgeld | wohnungslage | baujahr | effizienzklasse | makler                                         |
|----------------------------------------|---------|----------------------------------------------------------------------------------------|------------|--------|-------------------------------|-----------|----------|--------------|---------|-----------------|------------------------------------------------|
| https://www.immowelt.de/expose/27kcb5h | 27kcb5h | Penthaus mit Rheinblick                                                                | 137        | 4      | 50668 Köln  (Neustadt-Nord)   | 1568800   | 11343    |              | Neubau  |                 | S Immobilienpartner GmbH                       |
| https://www.immowelt.de/expose/27cfd5h | 27cfd5h | Ideale Stadtwohnung mit Dachterrasse                                                   | 88         | 3      | 50668 Köln  (Neustadt-Nord)   | 989800    | 10813    | 4. Geschoss  | Neubau  |                 | S Immobilienpartner GmbH                       |
| https://www.immowelt.de/expose/2be625g | 2be625g | TOP GELEGENHEIT: modernisierte Maisonette in Rheinnähe mit Domblick in Köln-Bayenthal  | 79         | 3      | 50968 Köln / Bayenthal        | 395000    | 15000    |              | 1895    |                 | AmRhein ImmobilienManagement GmbH              |
| https://www.immowelt.de/expose/2bejj52 | 2bejj52 | EIGENNUTZUNG ODER KAPITALANLAGE IN KÖLN-BILDERSTÖCKCHEN MIT TRAUMHAFTER RAUMAUFTEILUNG | 84         | 3      | 50739 Köln  (Bilderstöckchen) | 325000    | 20000    |              | 1968    | E               | Com-Plex Immobilien & Facility Management GmbH |
| https://www.immowelt.de/expose/2b8e659 | 2b8e659 | Wunderschönes Penthouse in exklusiver Lage am Kanal im Stadtwaldviertel                | 222        | 6      | 50858 Köln  (Junkersdorf)     | 1850000   | 30000    |              | 2005    |                 | INPREX-IMMO GmbH                               |
| https://www.immowelt.de/expose/2bexs5e | 2bexs5e | Neubau Maisonette-Wohnung für die Familie nähe Rheinwiesen                             | 167        | 4      | 51149 Köln  (Ensen)           | 799900    | 27500    |              | 2023    |                 | Uckelmann Immobilien, Boarding Concept         |
| https://www.immowelt.de/expose/2bpys5e | 2bpys5e | Haus im Haus - Neubau-Maisonette-Wohnung in Rheinnhähe                                 | 167        | 4      | 51149 Köln  (Ensen)           | 799900    | 27500    |              | 2023    |                 | Uckelmann Immobilien, Boarding Concept         |
| https://www.immowelt.de/expose/2baku5k | 2baku5k | Stadthaus in Marienburg                                                                | 124        | 4      | 50968 Köln  (Marienburg)      | 1179700   | 39900    |              |         |                 | S Immobilienpartner GmbH                       |
| https://www.immowelt.de/expose/2bcku5k | 2bcku5k | Modernes Stadthaus das begeistert                                                      | 123        | 4      | 50968 Köln  (Marienburg)      | 1179700   | 39900    |              |         |                 | S Immobilienpartner GmbH                       |
| https://www.immowelt.de/expose/2chtf56 | 2chtf56 | Super zentral gelegene 3-Zimmer-Wohnung mit Terrasse, Garten, offener EBK und 2 Bädern | 104        | 3      | 50676 Köln  (Altstadt-Süd)    | 1100000   | 10577    | Erdgeschoss  | 2016    | B               | Homeday GmbH                                   |
| https://www.immowelt.de/expose/2cwts5d | 2cwts5d | Exklusive Neubau EG Wohnung in Köln Rodenkirchen                                       | 150        | 4      | 50999 Köln  (Rodenkirchen)    | 1165000   | 15000    | Erdgeschoss  | 2023    |                 | VESER Real Estate GmbH & Co. KG                |
| https://www.immowelt.de/expose/2cwdj5b | 2cwdj5b | Die ideale Zweizimmer-Wohnung mit Sonnen-Loggia!!!                                     | 58         | 2      | 51145 Köln  (Urbach)          | 169000    | 20000    | 1. Geschoss  |         |                 | Privatanbieter                                 |
| https://www.immowelt.de/expose/2bg6t5b | 2bg6t5b | Verkauf ETW / 3 Zimmer / Balkon / Garage                                               | 89         | 3      | 50769 Köln  (Worringen)       | 253000    | 12000    | 1. Geschoss  | 1968    |                 | Privatanbieter                                 |
| https://www.immowelt.de/expose/227d85z | 227d85z | TOP-PREIS für möblierte 2 Zimmer-Whg. in Porz-Ensen mit sehr guter Verkehrsanbindung   | 53         | 2      | 51149 Köln  (Ensen)           | 295000    | 30000    |              | 1965    |                 | Emlak AG                                       |

#### The scraping of the condominium fee column did not return reliable results, as in each ad the row returned the price for the parking place or of the price per square meter.
#### Since this is not data that can be relied on, decided not to use it for the analysis

```sql
-- Separate the postal code from the neighborhood, creating two new column 

ALTER TABLE wohnung_kaufen
ADD COLUMN plz VARCHAR(255),
ADD COLUMN stadtteil VARCHAR(255);

UPDATE wohnung_kaufen
SET
plz = TRIM(SUBSTRING_INDEX(ort, ' ', 1)),
stadtteil = TRIM(SUBSTRING_INDEX(ort, ' ', -1))
```
```sql
-- Checked if there are postal code values with lenght below 5

SELECT plz 
FROM wohnung_kaufen 
WHERE LENGTH(ort) < 5
```
||||

```sql
-- Saw the "stadtteil" column 
SELECT stadtteil
FROM wohnung_kaufen
```
| stadtteil      |
|----------------|
| (Neuehrenfeld) |
| (Neustadt-Süd) |
| (Altstadt-Süd) |
| (Neuehrenfeld) |
| (Ehrenfeld)    |
| ...            |

```sql
-- Eliminated the parenthesis and the suffix "köln-"

UPDATE wohnung_kaufen
SET stadtteil = REPLACE(stadtteil, "(", "")

UPDATE wohnung_kaufen
SET stadtteil = REPLACE(stadtteil, ")", "")

UPDATE wohnung_kaufen
SET stadtteil = REPLACE(stadtteil, "Köln-", "")
```
```sql
-- Checked the results

SELECT stadtteil
FROM wohnung_kaufen
GROUP BY stadtteil
```
| stadtteil            |
|----------------------|
| Neuehrenfeld         |
| Neustadt-Süd         |
| Altstadt-Süd         |
| Ehrenfeld            |
| Buchheim             |
| Porz                 |
| Neustadt-Nord        |
| Niehl                |
| Humboldt-Gremberg    |
| Höhenberg            |
| Urbach               |
| Weiden               |
| Holweide             |
| Weidenpesch          |
| Altstadt-Nord        |
| Nippes               |
| Bickendorf           |
| Riehl                |
| Rondorf              |
| Mülheim              |
| Mauenheim            |
| Esch/Auweiler        |
| Widdersdorf          |
| Zollstock            |
| Braunsfeld           |
| Vingst               |
| Sürth                |
| Wahnheide            |
| Grengel              |
| Westhoven            |
| Dellbrück            |
| Raderberg            |
| Kalk                 |
| Zündorf              |
| Deutz                |
| Poll                 |
| Gremberghoven        |
| Neubrück             |
| Lindenthal           |
| Bocklemünd/Mengenich |
| Junkersdorf          |
| Merheim              |
| Eil                  |
| Ossendorf            |
| Seeberg              |
| Rodenkirchen         |
| Ostheim              |
| Bayenthal            |
| Höhenhaus            |
| Bilderstöckchen      |
| Rath/Heumar          |
| Klettenberg          |
| Merkenich            |
| Meschenich           |
| Longerich            |
| Heimersdorf          |
| Ensen                |
| Raderthal            |
| Sülz                 |
| Volkhoven/Weiler     |
| Langel               |
| Alt-Longerich        |
| Lövenich             |
| Pesch                |
| Müngersdorf          |
| Elsdorf              |
| Worringen            |
| Marienburg           |
| Dünnwald             |
| Flittard             |
| Brück                |
| Buchforst            |
| Weiß                 |
| Neu-Weiß             |
| Lind                 |
| Köln                 |
| Lindweiler           |
| Stammheim            |
| Vogelsang            |

```sql
-- Investigated the ad with neighborhood "Köln"

SELECT *
FROM wohnung_kaufen
WHERE stadtteil = "Köln"
```
| link                                   | ref_num | anzeige                                              | wohnfläche | zimmer | ort                    | kaufpreis | hausgeld | wohnungslage | baujahr | effizienzklasse | makler                         | plz   | stadtteil |
|----------------------------------------|---------|------------------------------------------------------|------------|--------|------------------------|-----------|----------|--------------|---------|-----------------|--------------------------------|-------|-----------|
| https://www.immowelt.de/expose/2bqbh5z | 2bqbh5z | Wohntraum auf 2 Ebenen: Doppelgarage und Süd-Balkon! | 108        | 4      | 50968 Raderberg / Köln | 529000    | 491      |              | 1993    | E               | FALC Immobilien Köln & Pulheim | 50968 | Köln      |

```sql
-- Updated the neighborhood value with the correct information 

UPDATE wohnung_kaufen
SET stadtteil = "Raderberg"
WHERE ref_num = "2bqbh5z"
```
Decided to join it with the table of postal codes to obtain the same resuts for all table
```sql
SELECT *
FROM wohnung_kaufen as wk
LEFT JOIN plz_koeln as pl
ON wk.plz = pl.plz
GROUP BY pl.stadtteil
ORDER BY count(*) DESC
```


























































