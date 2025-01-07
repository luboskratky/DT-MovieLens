# **ETL proces datasetu MovieLens**

Tento repozitár obsahuje ETL proces implementovaný v Snowflake na spracovanie a analýzu dát z datasetu MovieLens. Analýza pokrýva filmy, žánre, hodnotenia a údaje o používateľoch, čím umožňuje multidimenzionálnu analýzu a vizualizáciu kľúčových metrik. Výsledný dátový model podporuje efektívne reportovanie a odhaľovanie trendov v preferenciách používateľov.

---
## **1. Úvod a popis zdrojových dát**

Cieľom projektu je preskúmať hodnotenia filmov a demografické údaje používateľov, aby sa získali poznatky o ich preferenciách a sledovali trendy vo filmovej obľúbenosti.

Zdrojové dáta pochádzajú z datasetu dostupného [tu](https://grouplens.org/datasets/movielens/). Dataset obsahuje osem hlavných tabuliek:
- `age_groups`
- `genres`
- `genres_movies`
- `movies`
- `occupations`
- `ratings`
- `tags`
- `users`

Účelom ETL procesu bolo tieto dáta pripraviť, transformovať a sprístupniť pre viacdimenzionálnu analýzu.

---
### **1.1 Dátová architektúra**

### **ERD diagram**
Surové dáta sú usporiadané v relačnom modeli, ktorý je znázornený na **entitno-relačnom diagrame (ERD)**:

<p align="center">
  <img src="MovieLens_ERD.png">
  <br>
  <em>Obrázok 1 Entitno-relačná schéma MovieLens</em>
</p>

---
## **2. Dimenzionálny model**

Navrhnutý bol **hviezdicový model (star schema)**, pre efektívnu analýzu kde centrálny bod predstavuje faktová tabuľka **`fact_ratings`** a dimenzie:
- **`dim_movies`**: Obsahuje informácie o filmoch (názov, rok vydania).
    - **`subdim_genres`**: Obsahuje žánre filmov filmoch (názov).
- **`dim_users`**: Obsahuje demografické údaje o používateľoch(vek, vekové kategórie, pohlavie, povolanie).
- **`dim_date`**: Zahrňuje informácie o dátumoch hodnotení (deň, mesiac, rok,).
- **`dim_time`**: Obsahuje podrobné časové údaje (hodina, minuta, sekunda).

Štruktúra hviezdicového modelu je znázornená na diagrame nižšie. Diagram ukazuje prepojenia medzi faktovou tabuľkou a dimenziami, čo zjednodušuje pochopenie a implementáciu modelu.

<p align="center">
  <img src="starschema.png">
  <br>
  <em>Obrázok 2 Schéma hviezdy pre MovieLens</em>
</p>
---
## **3. ETL proces v Snowflake**
ETL proces pozostával z troch hlavných fáz: `extrahovanie` (Extract), `transformácia` (Transform) a `načítanie` (Load). Tento proces bol implementovaný v Snowflake s cieľom pripraviť zdrojové dáta zo staging vrstvy do viacdimenzionálneho modelu vhodného na analýzu a vizualizáciu.

---
### **3.1 Extrahovanie dát (Extract)**
Dáta zo zdrojového datasetu (formát `.csv`) boli najprv nahraté do Snowflake prostredníctvom interného stage úložiska s názvom `boa_stage`. Stage v Snowflake slúži ako dočasné úložisko na import alebo export dát. Vytvorenie stage bolo zabezpečené príkazom:

#### Príklad kódu:
```sql
CREATE OR REPLACE STAGE boa_stage;
```
Vytvorili sa stage tabulky ako dočasné úložisko pre údaje počas ETL procesu. Pre každú tabuľku sa použil podobný príkaz:
#### Príklad kódu:
```sql
CREATE OR REPLACE TABLE movies_staging (
    id INT PRIMARY KEY,
    title VARCHAR(255),
    release_year CHAR(4)
);
```
Do stage boli nahrané súbory s informáciami o filmoch, používateľoch, hodnoteniach, profesiách a žánroch. Dátové súbory boli načítané do staging tabuliek prostredníctvom príkazu COPY INTO. Pre každú tabuľku sa použil obdobný príkaz:

```sql
COPY INTO movies_staging
FROM @boa_stage/movies.csv
FILE_FORMAT = (TYPE = 'CSV'  SKIP_HEADER = 1);
```

V prípade nekonzistentných záznamov bol použitý parameter `ON_ERROR = 'CONTINUE'`, ktorý zabezpečil pokračovanie procesu bez prerušenia pri chybách.

---
### **3.2 Transformácia dát (Transform)**

V tejto fáze boli údaje zo staging tabuliek spracované, transformované a doplnené o ďalšie informácie. Hlavným cieľom bolo pripraviť dimenzionálne a faktové tabuľky na uľahčenie a zefektívnenie analýzy.


#### Príklad kódu dim_movie:
```sql
CREATE OR REPLACE TABLE dim_movie AS
SELECT
    movies_staging.id AS dim_movieId,
    movies_staging.title,
    movies_staging.release_year
FROM movies_staging;
```

Faktová tabuľka `fact_ratings` zaznamenáva hodnotenia a prepojenia na všetky príslušné dimenzie.
```sql
CREATE OR REPLACE TABLE fact_rating AS
SELECT
    rating_staging.id AS fact_ratingId,
    rating_staging.rating AS rating,
    dim_users.dim_usersId,
    dim_movie.dim_movieId,
    dim_time.dim_timeId,
    dim_date.dim_dateId,
    dim_tags.dim_tagsId  AS dim_tagsId
FROM rating_staging
JOIN dim_users ON rating_staging.user_id = dim_users.dim_usersId
JOIN dim_movie ON rating_staging.movie_id = dim_movie.dim_movieId
JOIN dim_time ON CAST(rating_staging.rated_at AS TIME) = dim_time.timestamp
JOIN dim_date ON CAST(rating_staging.rated_at AS DATE) = dim_date.timestamp
LEFT JOIN tags_staging ON rating_staging.id = tags_staging.id
LEFT JOIN dim_tags ON tags_staging.id = dim_tags.dim_tagsId;
```

---
### **3.3 Načítanie dát (Load)**

Po úspešnom vytvorení dimenzií a faktovej tabuľky boli dáta nahraté do finálnej štruktúry. Na záver boli staging tabuľky odstránené, aby sa optimalizovalo využitie úložiska:
```sql
DROP TABLE IF EXISTS users_staging;
DROP TABLE IF EXISTS genres_movies_staging;
DROP TABLE IF EXISTS age_group_staging;
DROP TABLE IF EXISTS rating_staging;
DROP TABLE IF EXISTS tags_staging;
DROP TABLE IF EXISTS occupations_staging;
DROP TABLE IF EXISTS genres_staging;
DROP TABLE IF EXISTS movies_staging;
```

ETL proces v Snowflake umožnil transformáciu pôvodných údajov z formátu `.csv` do viacdimenzionálneho hviezdicového modelu. Tento proces zahŕňal čistenie, obohacovanie a reorganizáciu dát. Výsledný model poskytuje základ pre analýzu preferencií a správania používateľov, čím umožňuje vytváranie vizualizácií a reportov.

