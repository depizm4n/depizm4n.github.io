# EXISTS句の使い方

サブクエリを常に利用し「サブクエリの結果1行以上を戻す」かの真理値を戻す。  
このため、 `IN` や `LIKE` のような1行の性質だけを調べる述語と比較して、  
高次元な状況を扱うことができる。

## 全称量子化への適用
述語論理には2つの特別な述語があり、  
全称量子化： すべての～を満たす    
存在量子化： ～という条件を満たすものがある  
という名前がつけられている。

存在量子化は `EXISTS` に相当するが、全称量子化は現在のSQLに存在しない。  
ただし、全称量子化の二重否定  
> ～でないという条件をみたすものはない

を存在量子化で表現することで、全称量子化をSQLで実現することができる。

## 基本形
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

「各教科がすべて50点以上である」を二重否定で表現する。

> 💡  
> 「50点未満である教科は存在しない」

```SQL
SELECT DISTINCT T1.ID
  FROM TestScore T1
 WHERE NOT EXISTS (SELECT 1
                     FROM TestScore T2
                    WHERE T1.ID = T2.ID
                      AND T2.SCORE < 50);
```

> 💡  
> `EXISTS` の引数がサブクエリであるため、  
> サブクエリ内で適切なカラムで結合することに注意する。

### 別解
このケースは全称量子化で表現できる。

```SQL
SELECT ID
  FROM TestScore
 GROUP BY ID
HAVING COUNT(*) = SUM(CASE WHEN SCORE >= 50
                           THEN 1
                           ELSE 0 END);
```

> 💡  
> `ID` で集約し、 `SCORE` が50以上である行で尽くされるか評価している。

## 量化に条件分岐を盛り込む
```
次の成績(TestScore)テーブルについて、
教科が数学の場合は50点以上、歴史の場合は60点以上である生徒を求める。  
（数学・歴史を受けていない生徒は結果に含まれるようにする）
```

前問の別解で利用した `CASE` 式を活用し、  
条件をもとに0,1に変換する `CASE` 式を考える。

```SQL
CASE WHEN SUBJECT = 'MATH'    AND SCORE >= 50 THEN 1
     WHEN SUBJECT = 'HISTORY' AND SCORE >= 60 THEN 1
     ELSE 0 END
```

> ⚡  
> ここでは数学、歴史以外の場合は0に捨てている。後述

これを二重否定し、 `NOT EXISTS` に組み込む。

```SQL
SELECT DISTINCT T1.ID
  FROM TestScore T1
 WHERE T1.SUBJECT IN ('MATH', 'HISTORY')
   AND NOT EXISTS (SELECT 1
                     FROM TestScore T2
                    WHERE T1.ID = T2.ID
                      AND 1 = (CASE WHEN SUBJECT = 'MATH'    AND SCORE < 50 THEN 1
                                    WHEN SUBJECT = 'HISTORY' AND SCORE < 60 THEN 1
                                    ELSE 0 END));
```

> ⚡  
> `IN` 句で参照しない教科を事前に捨てている。  
> これがない場合は `CASE` 句で数学・歴史以外の科目を拾いあげる必要がある。
