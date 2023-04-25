/*
==========================================================================================
Author: Derek Zhu
Create date: 21-APR-2023
Description: Create sample data for PIT creation of Data Vault 2.0 Demo
==========================================================================================
*/

-- Data Initialization
/*


-- person bio info
DROP TABLE IF EXISTS ods_finance.person;

CREATE TABLE person (
"person_id" varchar(20) NOT NULL,
"person_name" varchar(20),
"birth_date" date,
"system" varchar(20)
);

INSERT INTO ods_finance.person
("person_id", "person_name", "birth_date", "system")
VALUES('SA001', '小明', '2017-04-12', 'maternity');

-- person healthy info
DROP TABLE IF EXISTS ods_finance.person_health;

CREATE TABLE person_health (
"person_id" varchar(20) NOT NULL,
"healthy_info" varchar(255),
"event_date" date,
"system" varchar(20)
);

INSERT INTO ods_finance.person_health
("person_id", "healthy_info", "event_date", "system")
VALUES
('SA001', '门诊看病', '2023-04-03', 'hospital')
,('SA001', '雾化治疗', '2023-04-05', 'hospital')
,('SA001', '病愈复诊', '2023-04-07', 'hospital');

-- person financial info
DROP TABLE IF EXISTS ods_finance.person_finance;

CREATE TABLE person_finance (
"person_id" varchar(20) NOT NULL,
"financial_info" varchar(255),
"salary" decimal(10,2),
"event_date" date,
"system" varchar(20)
);

INSERT INTO ods_finance.person_finance
("person_id", "financial_info", salary, "event_date", "system")
VALUES
('SA001', '第一周工资', 100, '2023-04-04', 'payroll')
, ('SA001', '第二周工资', 80, '2023-04-08', 'payroll')
;
 
 */

-- Data Vault Core Model
/*
-- HUB_PERSON
DROP TABLE IF EXISTS ods_finance.hub_person;

SELECT 
upper(md5(upper(btrim(COALESCE(a.person_id, ''::character varying)::text))))::character(32) AS hashkey
, a.person_id 
, a.birth_date AS load_dts
, a."system" AS rec_src
INTO ods_finance.hub_person
FROM ods_finance.person AS a;

SELECT * FROM ods_finance.hub_person;

|hashkey                         |person_id|load_dts  |rec_src  |
|--------------------------------|---------|----------|---------|
|9DF830AB0A1A66390C002B619B188D10|SA001    |2017-04-12|maternity|



-- SAT_PERSON_BIO

SELECT 
upper(md5(upper(btrim(COALESCE(a.person_id, ''::character varying)::text))))::character(32) AS hashkey
, upper(md5(upper(concat(btrim(COALESCE(a.person_id, ''::character varying)::text)
, ';', btrim(COALESCE(a.person_name , ''::character varying)::text)
, ';', btrim(COALESCE(a.birth_date::text, ''::character varying)::text)
))))::character(32) AS hashdiff
, a.person_name
, a.birth_date
, a.birth_date AS load_dts
, a."system" AS rec_src
INTO ods_finance.sat_person_bio
FROM ods_finance.person AS a;

SELECT * FROM ods_finance.sat_person_bio;

|hashkey                         |hashdiff                        |person_name|birth_date|load_dts  |rec_src  |
|--------------------------------|--------------------------------|-----------|----------|----------|---------|
|9DF830AB0A1A66390C002B619B188D10|F894BCBC4749CCDE8B512E059ACF4A53|小明         |2017-04-12|2017-04-12|maternity|


-- SAT_PERSON_HEALTH

SELECT
upper(md5(upper(btrim(COALESCE(a.person_id, ''::character varying)::text))))::character(32) AS hashkey
, upper(md5(upper(concat(btrim(COALESCE(a.person_id, ''::character varying)::text)
, ';', btrim(COALESCE(a.healthy_info , ''::character varying)::text)
, ';', btrim(COALESCE(a.event_date::text, ''::character varying)::text)
))))::character(32) AS hashdiff
, a.healthy_info
, a.event_date
, a.event_date AS load_dts
, "system" AS rec_src
INTO ods_finance.sat_person_health
FROM ods_finance.person_health AS a;

SELECT * FROM ods_finance.sat_person_health;

|hashkey                         |hashdiff                        |healthy_info|event_date|load_dts  |rec_src |
|--------------------------------|--------------------------------|------------|----------|----------|--------|
|9DF830AB0A1A66390C002B619B188D10|018BC84D334E1579F27F1820293928DA|门诊看病        |2023-04-03|2023-04-03|hospital|
|9DF830AB0A1A66390C002B619B188D10|7A3827F1E141A01F12094B7798C77C93|雾化治疗        |2023-04-05|2023-04-05|hospital|
|9DF830AB0A1A66390C002B619B188D10|6F8F5F7D712C20505674912FB29EFC23|病愈复诊        |2023-04-07|2023-04-07|hospital|

-- SAT_PERSON_FINANCE

SELECT
upper(md5(upper(btrim(COALESCE(a.person_id, ''::character varying)::text))))::character(32) AS hashkey
, upper(md5(upper(concat(btrim(COALESCE(a.person_id, ''::character varying)::text)
, ';', btrim(COALESCE(a.financial_info , ''::character varying)::text)
, ';', btrim(COALESCE(a.salary::text, ''::character varying)::text)
, ';', btrim(COALESCE(a.event_date::text, ''::character varying)::text)
))))::character(32) AS hashdiff
, a.financial_info
, a.salary
, a.event_date
, a.event_date AS load_dts
, "system" AS rec_src
INTO ods_finance.sat_person_finance
FROM ods_finance.person_finance AS a;

SELECT * FROM ods_finance.sat_person_finance;

|hashkey                         |hashdiff                        |financial_info|salary|event_date|load_dts  |rec_src|
|--------------------------------|--------------------------------|--------------|------|----------|----------|-------|
|9DF830AB0A1A66390C002B619B188D10|09B52CC9347FDF2DB0924A02D5564205|第一周工资         |100   |2023-04-04|2023-04-04|payroll|
|9DF830AB0A1A66390C002B619B188D10|BB7FA008C724E7B24C3EE16047057458|第二周工资         |80    |2023-04-08|2023-04-08|payroll|

*/

-- PIT table creation 
-- ods_finance.pit_person definition
-- Drop table
-- DROP TABLE ods_finance.pit_person;
CREATE TABLE ods_finance.pit_person (
	pit_key bpchar(32) NULL,
	hashkey bpchar(32) NULL,
	snapshot_dts date NULL,
	personbio_hk bpchar NULL,
	personbio_ldts date NULL,
	personhealth_hk bpchar NULL,
	personhealth_ldts date NULL,
	personfinance_hk bpchar NULL,
	personfinance_ldts date NULL
);
CREATE INDEX pit_person_pit_key_idx ON ods_finance.pit_person USING btree (pit_key);

-- PIT without Ghost Records
/*
 * The logarithmic PIT managed windows, the Pros are as below
 * 1. it is REPEATABLE
 * 2. takes away the complexity of combining these structures away from the business user
 * 
 * */
WITH as_of_dates AS (
SELECT hashkey, datum AS snapshot_dts, person_id 
FROM ods_finance.hub_person, dmt_vx_sales.fn_get_calendar('2023-04-01', 7)
)
, new_rows_as_of_dates AS (
SELECT
upper(md5(upper(concat(btrim(COALESCE(person_id, ''::character varying)::text), ';', btrim(COALESCE(snapshot_dts::text, ''::character varying)::text)))))::character(32) AS pit_key
, hashkey
, snapshot_dts
FROM as_of_dates
)
SELECT
a.pit_key, a.hashkey, a.snapshot_dts
, COALESCE(max(sat1.hashkey), CAST('00000000000000000000000000000000' AS character varying)) AS personbio_hk
, COALESCE(max(sat1.load_dts), '1900-01-01') AS personbio_ldts
, COALESCE(max(sat2.hashkey), CAST('00000000000000000000000000000000' AS character varying)) AS personhealth_hk
, COALESCE(max(sat2.load_dts), '1900-01-01') AS personhealth_ldts
, COALESCE(max(sat3.hashkey), CAST('00000000000000000000000000000000' AS character varying)) AS personfinance_hk
, COALESCE(max(sat3.load_dts), '1900-01-01') AS personfinance_ldts
--INTO pit_person
FROM new_rows_as_of_dates AS a
LEFT JOIN ods_finance.sat_person_bio AS sat1 ON a.hashkey = sat1.hashkey AND sat1.load_dts <= a.snapshot_dts
LEFT JOIN ods_finance.sat_person_health AS sat2 ON a.hashkey = sat2.hashkey AND sat2.load_dts <= a.snapshot_dts
LEFT JOIN ods_finance.sat_person_finance AS sat3 ON a.hashkey = sat3.hashkey AND sat3.load_dts <= a.snapshot_dts
GROUP BY a.pit_key, a.hashkey, a.snapshot_dts
ORDER BY a.snapshot_dts
--
--|pit_key                         |hashkey                         |snapshot_dts|personbio_hk                    |personbio_ldts|personhealth_hk                 |personhealth_ldts|personfinance_hk                |personfinance_ldts|
--|--------------------------------|--------------------------------|------------|--------------------------------|--------------|--------------------------------|-----------------|--------------------------------|------------------|
--|445A52CD8E37599F6FCF576DA0C8FF1E|9DF830AB0A1A66390C002B619B188D10|2023-04-01  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |                                |                 |                                |                  |
--|254BEA08C4C3AEDB616C86D7BF92C913|9DF830AB0A1A66390C002B619B188D10|2023-04-02  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |                                |                 |                                |                  |
--|95CB1B5DABB4E7A5131A40396E3888C1|9DF830AB0A1A66390C002B619B188D10|2023-04-03  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |9DF830AB0A1A66390C002B619B188D10|2023-04-03       |                                |                  |
--|5DE036BDCEF260198293B306F484433A|9DF830AB0A1A66390C002B619B188D10|2023-04-04  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |9DF830AB0A1A66390C002B619B188D10|2023-04-03       |9DF830AB0A1A66390C002B619B188D10|2023-04-04        |
--|CF4F6D85790E28718E532B048B9B91E3|9DF830AB0A1A66390C002B619B188D10|2023-04-05  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |9DF830AB0A1A66390C002B619B188D10|2023-04-05       |9DF830AB0A1A66390C002B619B188D10|2023-04-04        |
--|1D2CA3BF2F43B92BDD5C6B4FC37C6490|9DF830AB0A1A66390C002B619B188D10|2023-04-06  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |9DF830AB0A1A66390C002B619B188D10|2023-04-05       |9DF830AB0A1A66390C002B619B188D10|2023-04-04        |
--|3D5CEDD4B9399F66863A2EA732D264E3|9DF830AB0A1A66390C002B619B188D10|2023-04-07  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |9DF830AB0A1A66390C002B619B188D10|2023-04-07       |9DF830AB0A1A66390C002B619B188D10|2023-04-04        |
--|6743FC0C18076621F32759560A35BBEF|9DF830AB0A1A66390C002B619B188D10|2023-04-08  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |9DF830AB0A1A66390C002B619B188D10|2023-04-07       |9DF830AB0A1A66390C002B619B188D10|2023-04-08        |
--
--
|pit_key                         |hashkey                         |snapshot_dts|personbio_hk                    |personbio_ldts|personhealth_hk                 |personhealth_ldts|personfinance_hk                |personfinance_ldts|
|--------------------------------|--------------------------------|------------|--------------------------------|--------------|--------------------------------|-----------------|--------------------------------|------------------|
|445A52CD8E37599F6FCF576DA0C8FF1E|9DF830AB0A1A66390C002B619B188D10|2023-04-01  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |00000000000000000000000000000000|1900-01-01       |00000000000000000000000000000000|1900-01-01        |
|254BEA08C4C3AEDB616C86D7BF92C913|9DF830AB0A1A66390C002B619B188D10|2023-04-02  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |00000000000000000000000000000000|1900-01-01       |00000000000000000000000000000000|1900-01-01        |
|95CB1B5DABB4E7A5131A40396E3888C1|9DF830AB0A1A66390C002B619B188D10|2023-04-03  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |9DF830AB0A1A66390C002B619B188D10|2023-04-03       |00000000000000000000000000000000|1900-01-01        |
|5DE036BDCEF260198293B306F484433A|9DF830AB0A1A66390C002B619B188D10|2023-04-04  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |9DF830AB0A1A66390C002B619B188D10|2023-04-03       |9DF830AB0A1A66390C002B619B188D10|2023-04-04        |
|CF4F6D85790E28718E532B048B9B91E3|9DF830AB0A1A66390C002B619B188D10|2023-04-05  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |9DF830AB0A1A66390C002B619B188D10|2023-04-05       |9DF830AB0A1A66390C002B619B188D10|2023-04-04        |
|1D2CA3BF2F43B92BDD5C6B4FC37C6490|9DF830AB0A1A66390C002B619B188D10|2023-04-06  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |9DF830AB0A1A66390C002B619B188D10|2023-04-05       |9DF830AB0A1A66390C002B619B188D10|2023-04-04        |
|3D5CEDD4B9399F66863A2EA732D264E3|9DF830AB0A1A66390C002B619B188D10|2023-04-07  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |9DF830AB0A1A66390C002B619B188D10|2023-04-07       |9DF830AB0A1A66390C002B619B188D10|2023-04-04        |
|6743FC0C18076621F32759560A35BBEF|9DF830AB0A1A66390C002B619B188D10|2023-04-08  |9DF830AB0A1A66390C002B619B188D10|2017-04-12    |9DF830AB0A1A66390C002B619B188D10|2023-04-07       |9DF830AB0A1A66390C002B619B188D10|2023-04-08        |

DROP TABLE ods_finance.sat_person_health;
-- query the PIT for best performace with Ghost Records

-- 想一想，为什么DV1.0里面，Ghost Records 要 by BK去创建，每一个BK一条Ghost Record

CREATE VIEW v_dim_person AS 
SELECT
pit.pit_key ,
pit.hashkey AS "pit_hashkey",
pit.snapshot_dts,
CASE WHEN a.person_id IS NOT NULL AND a.person_id <> '' THEN a.person_id ELSE '?' END AS "hub_business_key",
COALESCE(sat1.person_name , '?') AS "sat1_person_name",
COALESCE(sat1.birth_date, '1900-01-01') AS "sat1_person_birth_date",
COALESCE(sat2.healthy_info, '?') AS "sat2_person_healthinfo",
COALESCE(sat3.financial_info, '?') AS "sat3_person_financialinfo",
COALESCE(sat3.salary, 0) AS "sat3_person_salary"
FROM ods_finance.pit_person AS pit
INNER JOIN ods_finance.hub_person AS a ON pit.hashkey = a.hashkey 
INNER JOIN ods_finance.sat_person_bio AS sat1 ON (sat1.hashkey = pit.hashkey AND sat1.load_dts = pit.personbio_ldts)
INNER JOIN ods_finance.sat_person_health AS sat2 ON (sat2.hashkey = pit.hashkey AND sat2.load_dts = pit.personhealth_ldts)
INNER JOIN ods_finance.sat_person_finance AS sat3 ON (sat3.hashkey = pit.hashkey AND sat3.load_dts = pit.personfinance_ldts)

-- Alternative SAT Tables with Ghost Records Creation
--WITH sat1 AS (
--SELECT hashkey, hashdiff, person_name, birth_date, 1 AS sequence_id, load_dts, rec_src
--FROM ods_finance.sat_person_bio
--UNION ALL
--SELECT 
--'00000000000000000000000000000000','00000000000000000000000000000000','00000000000000000000000000000000','1900-01-01',0,'1900-01-01','ghost'
--)
----SELECT *
----INTO ods_finance.sat_person_bio_alt
----FROM sat1 ORDER BY sequence_id
--, sat2 AS (
--SELECT hashkey, hashdiff, healthy_info ,event_date , row_number() OVER (PARTITION BY hashkey ORDER BY event_date) AS sequence_id, load_dts, rec_src
--FROM ods_finance.sat_person_health
--UNION ALL
--SELECT 
--'00000000000000000000000000000000','00000000000000000000000000000000','00000000000000000000000000000000','1900-01-01',0,'1900-01-01','ghost'
--)
----SELECT * 
----INTO ods_finance.sat_person_health_alt
----FROM sat2 ORDER BY sequence_id
--, sat3 AS (
--SELECT hashkey, hashdiff, financial_info, salary, event_date, row_number() OVER (PARTITION BY hashkey ORDER BY event_date) AS sequence_id, load_dts, rec_src
--FROM ods_finance.sat_person_finance
--UNION ALL
--SELECT 
--'00000000000000000000000000000000','00000000000000000000000000000000','00000000000000000000000000000000',0,'1900-01-01',0,'1900-01-01','ghost'
--)
----SELECT * 
----INTO ods_finance.sat_person_finance_alt
----FROM sat3 ORDER BY sequence_id

/*
 * using a query to combine these SATs without a PIT
 * Work around way instead of PIT
 * 
 * */
WITH PIT AS (
SELECT hashkey, CAST('1900-01-01' AS DATE) AS PIT_EFFECTIVE_DATETIME FROM ods_finance.hub_person
UNION
SELECT hashkey, load_dts AS PIT_EFFECTIVE_DATETIME FROM ods_finance.hub_person
UNION
SELECT hashkey , load_dts AS PIT_EFFECTIVE_DATETIME FROM ods_finance.sat_person_bio_alt
UNION
SELECT hashkey , load_dts AS PIT_EFFECTIVE_DATETIME FROM ods_finance.sat_person_health_alt
UNION
SELECT hashkey , load_dts AS PIT_EFFECTIVE_DATETIME FROM ods_finance.sat_person_finance_alt
)
, TimeRanges AS (
SELECT 
	hashkey
	, PIT_EFFECTIVE_DATETIME
	, LEAD(PIT_EFFECTIVE_DATETIME,1,'9999-12-31') OVER (PARTITION BY hashkey ORDER BY PIT_EFFECTIVE_DATETIME ASC) AS PIT_EXPIRY_DATETIME
FROM PIT
)
SELECT
t1.hashkey AS "hub_hashkey",
a.PIT_EFFECTIVE_DATETIME,
a.PIT_EXPIRY_DATETIME,
COALESCE(max(sat1.hashkey),CAST('00000000000000000000000000000000' AS character varying)) AS "sat_person_bio.hashkey",
COALESCE(max(sat1.load_dts), '1900-01-01') AS "sat_person_bio.load_dts",
COALESCE(max(sat2.hashkey),CAST('00000000000000000000000000000000' AS character varying)) AS "sat_person_health.hashkey",
COALESCE(max(sat2.load_dts), '1900-01-01') AS "sat_person_health.load_dts",
COALESCE(max(sat3.hashkey),CAST('00000000000000000000000000000000' AS character varying)) AS "sat_person_finance.hashkey",
COALESCE(max(sat3.load_dts), '1900-01-01') AS "sat_person_finance.load_dts"
FROM TimeRanges AS a
INNER JOIN ods_finance.hub_person AS t1 ON a.hashkey = t1.hashkey
LEFT JOIN ods_finance.sat_person_bio_alt AS sat1 ON a.hashkey = sat1.hashkey AND sat1.load_dts <= a.PIT_EFFECTIVE_DATETIME
LEFT JOIN ods_finance.sat_person_health_alt AS sat2 ON a.hashkey = sat2.hashkey AND sat2.load_dts <= a.PIT_EFFECTIVE_DATETIME
LEFT JOIN ods_finance.sat_person_finance_alt AS sat3 ON a.hashkey = sat3.hashkey AND sat3.load_dts <= a.PIT_EFFECTIVE_DATETIME
GROUP BY t1.hashkey,
a.PIT_EFFECTIVE_DATETIME,
a.PIT_EXPIRY_DATETIME
--
--|hub_hashkey                     |pit_effective_datetime|pit_expiry_datetime|sat_person_bio.hashkey          |sat_person_bio.load_dts|sat_person_health.hashkey       |sat_person_health.load_dts|sat_person_finance.hashkey      |sat_person_finance.load_dts|
--|--------------------------------|----------------------|-------------------|--------------------------------|-----------------------|--------------------------------|--------------------------|--------------------------------|---------------------------|
--|9DF830AB0A1A66390C002B619B188D10|1900-01-01            |2017-04-12         |                                |                       |                                |                          |                                |                           |
--|9DF830AB0A1A66390C002B619B188D10|2017-04-12            |2023-04-03         |9DF830AB0A1A66390C002B619B188D10|2017-04-12             |                                |                          |                                |                           |
--|9DF830AB0A1A66390C002B619B188D10|2023-04-03            |2023-04-04         |9DF830AB0A1A66390C002B619B188D10|2017-04-12             |9DF830AB0A1A66390C002B619B188D10|2023-04-03                |                                |                           |
--|9DF830AB0A1A66390C002B619B188D10|2023-04-04            |2023-04-05         |9DF830AB0A1A66390C002B619B188D10|2017-04-12             |9DF830AB0A1A66390C002B619B188D10|2023-04-03                |9DF830AB0A1A66390C002B619B188D10|2023-04-04                 |
--|9DF830AB0A1A66390C002B619B188D10|2023-04-05            |2023-04-07         |9DF830AB0A1A66390C002B619B188D10|2017-04-12             |9DF830AB0A1A66390C002B619B188D10|2023-04-05                |9DF830AB0A1A66390C002B619B188D10|2023-04-04                 |
--|9DF830AB0A1A66390C002B619B188D10|2023-04-07            |2023-04-08         |9DF830AB0A1A66390C002B619B188D10|2017-04-12             |9DF830AB0A1A66390C002B619B188D10|2023-04-07                |9DF830AB0A1A66390C002B619B188D10|2023-04-04                 |
--|9DF830AB0A1A66390C002B619B188D10|2023-04-08            |9999-12-31         |9DF830AB0A1A66390C002B619B188D10|2017-04-12             |9DF830AB0A1A66390C002B619B188D10|2023-04-07                |9DF830AB0A1A66390C002B619B188D10|2023-04-08                 |
--
-- With Ghost Records
--
WITH PIT AS (
SELECT hashkey, CAST('1900-01-01' AS DATE) AS PIT_EFFECTIVE_DATETIME FROM ods_finance.hub_person
UNION
SELECT hashkey, load_dts AS PIT_EFFECTIVE_DATETIME FROM ods_finance.hub_person
UNION
SELECT hashkey , load_dts AS PIT_EFFECTIVE_DATETIME FROM ods_finance.sat_person_bio
UNION
SELECT hashkey , load_dts AS PIT_EFFECTIVE_DATETIME FROM ods_finance.sat_person_health
UNION
SELECT hashkey , load_dts AS PIT_EFFECTIVE_DATETIME FROM ods_finance.sat_person_finance
)
, TimeRanges AS (
SELECT 
	hashkey
	, PIT_EFFECTIVE_DATETIME
	, LEAD(PIT_EFFECTIVE_DATETIME,1,'9999-12-31') OVER (PARTITION BY hashkey ORDER BY PIT_EFFECTIVE_DATETIME ASC) AS PIT_EXPIRY_DATETIME
FROM PIT
)
SELECT
t1.hashkey AS "hub_hashkey",
a.PIT_EFFECTIVE_DATETIME,
a.PIT_EXPIRY_DATETIME,
COALESCE(max(sat1.hashkey),CAST('00000000000000000000000000000000' AS character varying)) AS "sat_person_bio.hashkey",
COALESCE(max(sat1.load_dts), '1900-01-01') AS "sat_person_bio.load_dts",
COALESCE(max(sat2.hashkey),CAST('00000000000000000000000000000000' AS character varying)) AS "sat_person_health.hashkey",
COALESCE(max(sat2.load_dts), '1900-01-01') AS "sat_person_health.load_dts",
COALESCE(max(sat3.hashkey),CAST('00000000000000000000000000000000' AS character varying)) AS "sat_person_finance.hashkey",
COALESCE(max(sat3.load_dts), '1900-01-01') AS "sat_person_finance.load_dts"
FROM TimeRanges AS a
INNER JOIN ods_finance.hub_person AS t1 ON a.hashkey = t1.hashkey
LEFT JOIN ods_finance.sat_person_bio AS sat1 ON a.hashkey = sat1.hashkey AND sat1.load_dts <= a.PIT_EFFECTIVE_DATETIME
LEFT JOIN ods_finance.sat_person_health AS sat2 ON a.hashkey = sat2.hashkey AND sat2.load_dts <= a.PIT_EFFECTIVE_DATETIME
LEFT JOIN ods_finance.sat_person_finance AS sat3 ON a.hashkey = sat3.hashkey AND sat3.load_dts <= a.PIT_EFFECTIVE_DATETIME
GROUP BY t1.hashkey,
a.PIT_EFFECTIVE_DATETIME,
a.PIT_EXPIRY_DATETIME
--
--|hub_hashkey                     |pit_effective_datetime|pit_expiry_datetime|sat_person_bio.hashkey          |sat_person_bio.load_dts|sat_person_health.hashkey       |sat_person_health.load_dts|sat_person_finance.hashkey      |sat_person_finance.load_dts|
--|--------------------------------|----------------------|-------------------|--------------------------------|-----------------------|--------------------------------|--------------------------|--------------------------------|---------------------------|
--|9DF830AB0A1A66390C002B619B188D10|1900-01-01            |2017-04-12         |00000000000000000000000000000000|1900-01-01             |00000000000000000000000000000000|1900-01-01                |00000000000000000000000000000000|1900-01-01                 |
--|9DF830AB0A1A66390C002B619B188D10|2017-04-12            |2023-04-03         |9DF830AB0A1A66390C002B619B188D10|2017-04-12             |00000000000000000000000000000000|1900-01-01                |00000000000000000000000000000000|1900-01-01                 |
--|9DF830AB0A1A66390C002B619B188D10|2023-04-03            |2023-04-04         |9DF830AB0A1A66390C002B619B188D10|2017-04-12             |9DF830AB0A1A66390C002B619B188D10|2023-04-03                |00000000000000000000000000000000|1900-01-01                 |
--|9DF830AB0A1A66390C002B619B188D10|2023-04-04            |2023-04-05         |9DF830AB0A1A66390C002B619B188D10|2017-04-12             |9DF830AB0A1A66390C002B619B188D10|2023-04-03                |9DF830AB0A1A66390C002B619B188D10|2023-04-04                 |
--|9DF830AB0A1A66390C002B619B188D10|2023-04-05            |2023-04-07         |9DF830AB0A1A66390C002B619B188D10|2017-04-12             |9DF830AB0A1A66390C002B619B188D10|2023-04-05                |9DF830AB0A1A66390C002B619B188D10|2023-04-04                 |
--|9DF830AB0A1A66390C002B619B188D10|2023-04-07            |2023-04-08         |9DF830AB0A1A66390C002B619B188D10|2017-04-12             |9DF830AB0A1A66390C002B619B188D10|2023-04-07                |9DF830AB0A1A66390C002B619B188D10|2023-04-04                 |
--|9DF830AB0A1A66390C002B619B188D10|2023-04-08            |9999-12-31         |9DF830AB0A1A66390C002B619B188D10|2017-04-12             |9DF830AB0A1A66390C002B619B188D10|2023-04-07                |9DF830AB0A1A66390C002B619B188D10|2023-04-08                 |



-- Building a Scalable Data Vault 2.0 Page 599
SELECT
a."key" ,
a.hashkey ,
a.loaddate AS snapshotdate,
--sat1.hashkey AS sat1_hashkey,
--sat1.loaddate AS sat1_loaddate,
--sat2.hashkey AS sat2_hashkey,
--sat2.loaddate AS sat2_loaddate,
--sat3.hashkey AS sat3_hashkey,
--sat3.loaddate AS sat3_loaddate
COALESCE(sat1.hashkey , CAST('00000' AS character varying)) AS sat1_hashkey,
COALESCE(sat1.loaddate , '1900-01-01') AS sat1_loaddate,
COALESCE(sat2.hashkey , CAST('00000' AS character varying)) AS sat2_hashkey,
COALESCE(sat2.loaddate , '1900-01-01') AS sat2_loaddate,
COALESCE(sat3.hashkey , CAST('00000' AS character varying)) AS sat3_hashkey,
COALESCE(sat3.loaddate , '1900-01-01') AS sat3_loaddate
--INTO patrickcuba_pit
FROM ods_finance.patrickcuba_hub AS a
LEFT OUTER JOIN ods_finance.patrickcuba_sat1 AS sat1 ON (a.hashkey = sat1.hashkey AND a.loaddate BETWEEN sat1.loaddate AND COALESCE(sat1.loaddate, '9999-12-31'))
LEFT OUTER JOIN ods_finance.patrickcuba_sat2 AS sat2 ON (a.hashkey = sat2.hashkey AND a.loaddate BETWEEN sat2.loaddate AND COALESCE(sat2.loaddate, '9999-12-31'))
LEFT OUTER JOIN ods_finance.patrickcuba_sat3 AS sat3 ON (a.hashkey = sat3.hashkey AND a.loaddate BETWEEN sat3.loaddate AND COALESCE(sat3.loaddate, '9999-12-31'))
-- PIT construct with NULL records
--|key|hashkey|snapshotdate|sat1_hashkey|sat1_loaddate|sat2_hashkey|sat2_loaddate|sat3_hashkey|sat3_loaddate|
--|---|-------|------------|------------|-------------|------------|-------------|------------|-------------|
--|444|a5114  |2023-04-23  |a5114       |2023-04-23   |a5114       |2023-04-23   |            |             |
--|555|b5385  |2023-04-23  |b5385       |2023-04-23   |b5385       |2023-04-23   |b5385       |2023-04-23   |
-- PIT construct with Ghost records
--|key|hashkey|snapshotdate|sat1_hashkey|sat1_loaddate|sat2_hashkey|sat2_loaddate|sat3_hashkey|sat3_loaddate|
--|---|-------|------------|------------|-------------|------------|-------------|------------|-------------|
--|444|a5114  |2023-04-23  |a5114       |2023-04-23   |a5114       |2023-04-23   |00000       |1900-01-01   |
--|555|b5385  |2023-04-23  |b5385       |2023-04-23   |b5385       |2023-04-23   |b5385       |2023-04-23   |


-- 
SELECT
a."key" ,
a.hashkey ,
a.loaddate AS snapshotdate,
COALESCE(sat1.hashkey , CAST('00000' AS character varying)) AS sat1_hashkey,
COALESCE(sat1.loaddate , '1900-01-01') AS sat1_loaddate,
COALESCE(sat2.hashkey , CAST('00000' AS character varying)) AS sat2_hashkey,
COALESCE(sat2.loaddate , '1900-01-01') AS sat2_loaddate,
COALESCE(sat3.hashkey , CAST('00000' AS character varying)) AS sat3_hashkey,
COALESCE(sat3.loaddate , '1900-01-01') AS sat3_loaddate
FROM ods_finance.patrickcuba_hub AS a
LEFT JOIN ods_finance.patrickcuba_sat1 AS sat1 ON a.hashkey = sat1.hashkey AND sat1.loaddate <= a.loaddate
LEFT JOIN ods_finance.patrickcuba_sat2 AS sat2 ON a.hashkey = sat2.hashkey AND sat2.loaddate <= a.loaddate
LEFT JOIN ods_finance.patrickcuba_sat3 AS sat3 ON a.hashkey = sat3.hashkey AND sat3.loaddate <= a.loaddate
-- Result
|key|hashkey|snapshotdate|sat1_hashkey|sat1_loaddate|sat2_hashkey|sat2_loaddate|sat3_hashkey|sat3_loaddate|
|---|-------|------------|------------|-------------|------------|-------------|------------|-------------|
|444|a5114  |2023-04-23  |a5114       |2023-04-23   |a5114       |2023-04-23   |00000       |1900-01-01   |
|555|b5385  |2023-04-23  |b5385       |2023-04-23   |b5385       |2023-04-23   |b5385       |2023-04-23   |
