---
layout: post
title:  "Cálculo de letras del DNI con T-SQL"
date:   2020-05-29 00:00:00 +0200
tipue_search_active: true
excerpt_separator: <!--end_excerpt-->
tags: TSQL
---

>NOTA: Este post ha sido migrado de mi blog oficial tal cual. Fue escrito en 2007

Si necesitas calcular la letra DNI de un conjunto grande de números DNI, aqui tienes una posible solución con T-SQL. 

Para optimizar, puedes almacenar el NIF en un tipo de datos INT ( sin letra ), que puedes poner como PK. Con la siguiente CTE podrás calcular la letra de esos nº de NIF:

<!--end_excerpt-->

```sql
use tempdb
go
DECLARE @NIFS_table AS TABLE(numnif int primary key);
insert into @NIFS_table(numnif) values(12569875),(52345612);
select * from @NIFS_table;

DECLARE @letras_nif as CHAR(23) = 'TRWAGMYFPDXBNJZSQVHLCKE';

WITH SubSelect AS(
SELECT numnif,
CONVERT(INT,
  FLOOR(
     FLOOR(
           (
            (
             CONVERT(FLOAT,numnif)/23)-FLOOR(CONVERT(FLOAT,numnif)/23)
          )*100
    )*0.23+0.5
 )+1
)AS pos_letra
FROM @NIFS_table
)
SELECT SUBSTRING(@letras_nif, pos_letra ,1) AS letra,
       str(numnif)+'-'+SUBSTRING(@letras_nif, pos_letra ,1) dni        
FROM subselect
```