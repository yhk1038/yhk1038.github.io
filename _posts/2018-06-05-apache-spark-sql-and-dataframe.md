---
layout: post
title:  "Apache SparkSQL과 Dataframe"
subtitle: "Apache SparkSQL과 Dataframe"
categories: data
tags: engineering
comments: true
---

SparkSQL과 Dataframe에 대한 포스팅입니다!


## SparkSQL
- SQL을 원하는 이유
	- 익숙한 언어인 SQL로 데이터를 분석하고 싶어서
	- 맵리듀스보다 빠르고 편함
	- 정형화된 데이터의 경우 높은 최적화를 구현 가능 
- Hive의 경쟁자
- Spark 2.0은 Spark SQL을 위한 업데이트
	- 더 많은 쿼리와 파일포맷 지원 강화
- 현재 Low-level API인 RDD와 공존, 앞으로 Dataset API쪽으로도 무게가 실릴 수도!
- 머신러닝은 정형화된 데이터셋을 주로 다루기 때문에 Dataframe API로 다시 쓰여짐 
- [SparkSQL Programming Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html)

## Spark Dataset, Dataframe API
- SparkSQL은 구조화된 데이터가 필요
- ```Dataset[Row] = Dataframe```
- Json 파일에서 스키마를 읽어 Dataframe을 만듬
- RDD -> DS, DF로 변환 가능
- MLlib에서도 구조화된 데이터를 다루고 언어별 통일성 등의 장점을 취하기 위해 Dataframe 사용
- [Dataset(Dataframe) Document](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Dataset)
- [Functions](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.functions$) : 들어가서 함수 Check! agg 안에 넣을 수 있는 함수들입니다

## Dataframe 실습

```
val df = spark.read.option("header", "true").csv("character-deaths.csv")

df.show(10, true) // true : 내용이 길면 ...화 되는 것! false로 하면 풀네임이 나옴
// API 문서찾을 때 Dataset을 찾아보면 됩니다!!


df.count

// TempView 생성
df.createOrReplaceTempView("game_of_throne")


%sql
select *
from game_of_throne

%sql
select Allegiances, count(1)
from game_of_throne
group by Allegiances


// sql로 표현한 것을 df에서 진행하면 아래와 같음
df.groupBy("Allegiances").count.show

df.groupBy("Allegiances").agg(count("Name")).show



df.groupBy("Allegiances").agg(count("Name"), collect_list("Name")).show(false)


df.groupBy("Allegiances").agg(count($"Name"), sum("Gender"), sum("Nobility")).show(false)



%sql
select *
from game_of_throne

// 바차트를 선택한 후, settings를 클릭해서 key와 그룹을 조절할 수 있음

// error 발생
df.filter("Allegiances" == "Baratheon").show 

// ===를 사용, 앞의 $"Allegiances" : column name이 됨
df.filter($"Allegiances" === "Baratheon").show

// Death Year 역순 정렬
df.filter($"Allegiances" === "Baratheon").orderBy($"Death Year".desc).show


// 특정 가문의 성별 count
df.filter($"Allegiances" === "Baratheon").groupBy("Gender").count.show


// select
df.filter($"Allegiances" === "Baratheon").select($"Name", $"Death Year").show

// 파일 쓰기
val df2 = df.filter($"Allegiances" === "Baratheon").select($"Name", $"Death Year")

df2.write.csv("df2")


// 하둡에 저장하는 것을 기본으로 해서, 읽기 쓰기는 쉽지만 삭제 및 수정이 어려움
// mode("overwrite")로 가능
df2.write.mode("overwrite").option("header", "true").csv("df2")

df2.write.mode("overwrite").option("header", "true").json("df2")


// colum 2개를 삭제하고 Gender2를 복사, Gender가 1이냐 아니냐로 나눔
df.drop("GoT", "CoK").withColumn("Gender2", $"Gender" ===1).show

// 성별을 남녀로 하고싶다면 UDF를 사용해야 합니다
// UDF 정의
// SQL용
sqlContext.udf.register("genderString", (gender: int => if (gender ==1) "Male" else "Female")

// Dataframe용 : 2개가 나뉨.. 과도기라서 둘 다 알아야 합니다
val genderString = udf((gender: Int) => if (gender ==1) "Male" else "Female")

// UDF를 사용해서 성별을 Male, Female로 나옴
df.drop("GoT", "CoK").withColumn("Gender2", genderString($"Gender")).show

```


