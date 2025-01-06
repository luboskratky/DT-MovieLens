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
  <img ******>
  <br>
  <em>Obrázok 1 Entitno-relačná schéma MovieLens</em>
</p>

---
## **2. Dimenzionálny model**

