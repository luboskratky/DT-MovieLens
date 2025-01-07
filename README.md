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


