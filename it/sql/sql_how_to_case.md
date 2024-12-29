# CASE式の使い方

## アドホックな集約キー
```
次の人口(Populations)テーブルについて、  
四国、九州、その他で集約した総人口を求める。
```

|PREFECTURE|POP|
|:--|:--|
|愛媛|50|
|高知|40|
|佐賀|30|
|福岡|20|
|愛知|10|

地域を示すカラムを追加することなく、対応することができる。

```sql
SELECT CASE WHEN PREFECTURE = '愛媛' THEN '四国'
            WHEN PREFECTURE = '高知' THEN '四国'
            WHEN PREFECTURE = '佐賀' THEN '九州'
            WHEN PREFECTURE = '福岡' THEN '九州'
            ELSE 'その他' END AS DISTRICT,
       SUM(POP) AS POP
  FROM Populations
 GROUP BY DISTRICT;
```

> ⚡  
> DBによっては `GROUP BY` に同一の `CASE` 式を記述する必要がある。

## アドホックなソートキー
```
次の人口(Populations)テーブルについて、  
愛媛、愛知、佐賀、高知、福岡の順で求める。
```

|PREFECTURE|POP|
|:--|:--|
|愛媛|50|
|高知|40|
|佐賀|30|
|福岡|20|
|愛知|10|

同様に、ソート用のカラムを追加することなく対応できる。

```sql
SELECT PREFECTURE,
       POP
  FROM Populations
 ORDER BY CASE WHEN PREFECTURE = '愛媛' THEN 1
               WHEN PREFECTURE = '愛知' THEN 2
               WHEN PREFECTURE = '佐賀' THEN 3
               WHEN PREFECTURE = '高知' THEN 4
               WHEN PREFECTURE = '高知' THEN 5
               ELSE NULL END;
```

## 集約関数に使う
```
次の人口(Populations)テーブルについて、男女を集計した結果を求める。
```

|PREFECTURE|SEX|POP|
|:--|:--|
|愛媛|1|50|
|愛媛|2|40|
|高知|1|40|
|高知|2|30|
|佐賀|1|30|
|佐賀|2|20|
|福岡|1|30

```sql
SELECT PREFECTURE,
       SUM(CASE WHEN SEX = 1 THEN POP ELSE 0 END) AS POP_1,
       SUM(CASE WHEN SEX = 2 THEN POP ELSE 0 END) AS POP_2
  FROM Populations
 GROUP BY PREFECTURE;
```

## 全称量子化の実現
```
次の成績(TestScore)テーブルについて、各教科がすべて50点以上である生徒を求める。
```

|ID|SUBJECT|SCORE|
|:--|:--|:--|
|1|MATH|50|
|1|SCIENCE|60
|2|MATH|50
|2|HISTORY|40
|3|MATH|50|
|3|HISTORY|60

```sql
SELECT ID
  FROM TestScore
 GROUP BY ID
HAVING COUNT(*) = SUM(CASE WHEN SCORE >= 50 THEN 1
                           ELSE 0 END);
```