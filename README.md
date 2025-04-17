# campaign-case-study

solution 


  select	a.*,
			case 
			when Campaigns is null then (
			select top 1 Campaigns 
			from Sheet1$
			where a.[S No ]>[S No ] and Campaigns is not null
			order by [S No ] desc )
			else Campaigns end as Campaign
			,
			case 
			when Duration is null then (
			select top 1 Duration 
			from Sheet1$
			where a.[S No ]>[S No ] and Duration is not null
			order by [S No ] desc )
			else Duration end as Duration1
			,
			case 
			when [product ] is null then (
			select top 1 [product ] 
			from Sheet1$
			where a.[S No ]>[S No ] and [product ] is not null
			order by [S No ] desc )
			else [product ] end as [Products]

  ---into		Campaign_Data__CS 
  from		[Pratice].[dbo].[Sheet1$] a

 
alter table Campaign_Data__CS  drop column Campaigns,[product ],Duration


  Select * from Campaign_Data__CS

  Select *,
SUBSTRING(Duration1,0,CHARINDEX('-',Duration1)) Start_Date,
SUBSTRING(Duration1,CHARINDEX('-',Duration1)+1,CHARINDEX('-',Duration1)) End_Date
----into Campaign_Data1_CS
From Campaign_Data__CS

Select * from Campaign_Data1_CS

Select *,
CONVERT(date,Start_Date,3) as [Start_DateNr],
CONVERT(date,[End_Date],3) as [End_DateNr]
----INto Campaign_Data2_CS
From Campaign_Data1_CS


select * from Campaign_Data3_CS

select *,
DATENAME(WEEKDAY,Start_DateNr) as Start_Week,
DATENAME(WEEKDAY,End_DateNr) as End_Week
----Into Campaign_Data3_CS
From Campaign_Data2_CS

Select *
,case when Start_Week='Saturday' then DATEADD(DAY,6,[Start_DateNr])
when Start_Week='Sunday' then DATEADD(DAY,5,[Start_DateNr])
when Start_Week='Monday' then DATEADD(DAY,4,[Start_DateNr])
when Start_Week='Tuesday' then DATEADD(DAY,3,[Start_DateNr])
when Start_Week='Wednesday' then DATEADD(DAY,2,[Start_DateNr])
when Start_Week='Thursday' then DATEADD(DAY,1,[Start_DateNr])
when Start_Week='Friday' then DATEADD(DAY,0,[Start_DateNr])
End as Friday_Start_Date,
case when End_Week='Saturday' then DATEADD(DAY,6,[End_DateNr])
when End_Week='Sunday' then DATEADD(DAY,5,[End_DateNr])
when End_Week='Monday' then DATEADD(DAY,4,[End_DateNr])
when End_Week='Tuesday' then DATEADD(DAY,3,[End_DateNr])
when End_Week='Wednesday' then DATEADD(DAY,2,[End_DateNr])
when End_Week='Thursday' then DATEADD(DAY,1,[End_DateNr])
when End_Week='Friday' then DATEADD(DAY,0,[End_DateNr])
End as Friday_End_Date
----Into Campaign_Data4_CS
From Campaign_Data3_CS

Select *, 
(LEN([Products])-LEN(REPLACE([Products],'-',''))+1) as [No. of Products]
------into Campaign_Data5_CS
from Campaign_Data4_CS

select * from Campaign_Data5_CS

Select *,
		Replace(SUBSTRING(Spend,2,6),',','') as Total_Spend
---Into Campaign_Data7_CS
from Campaign_Data5_CS

select *,
		Total_Spend/[No. of Products] as [Per Product Spend],
		DATEDIFF(Day,Start_DateNr,End_DateNr)+1 as Total_Days,
		SUBSTRING(Products,1,CHARINDEX('-',Products)-1) as Product1,
		SUBSTRING(Products,CHARINDEX('-',Products)+1,5) as Product2,
		SUBSTRING(Products,CHARINDEX('-',Products)+7,5) as Product3
------into 	Campaign_Data9_CS
from Campaign_Data7_CS

Select a.*,
b.[All Dates]
----Into 		Campaign_Data10_CS
from Campaign_Data9_CS a
cross join [dbo].['All Dates_CS$'] b


Select *,COnvert(DAte,[All Dates],3) as [All_Date]
-------Into   Campaign_Data11_CS 
from   Campaign_Data10_CS

Select * from [dbo].['All Dates_CS$']

select *,
		case 
		when All_Date >= Friday_Start_Date and  All_Date <= Friday_End_Date   then 1
		else 0 end as [Validation]
------into Campaign_Data14_CS
from Campaign_Data11_CS

Alter table Campaign_Data11_CS
Drop Column [All Dates],[Start_Date],[End_Date],[Start_Week],[End_Week],[Spend]


Delete From Campaign_Data12_CS
where Validation = 0

Select * From Campaign_Data12_CS

Select *,
		case 
		when DATEDIFF(Day,Start_DateNr,All_Date)<=6 then DATEDIFF(Day,Start_DateNr,All_Date)+1
		when DATEDIFF(Day,All_Date,End_DateNr)<=6 and  DATEDIFF(Day,All_Date,End_DateNr)<=0  then 7+DATEDIFF(Day,All_Date,End_DateNr)
		else 7
		end as [No. Of Days In Week]
----Into Campaign_Data13_CS
From Campaign_Data12_CS


select a.*
----into Campaign_Final_Output
from
(Select a.*,
       case 
	   when len(a.Product)<3 Then 'Remove'
	   else 'Keep'
	   end as Indicatore
From 
(select a.[channel ],a.Category,a.Product1 as Product,a.Campaign,a.Start_DateNr,a.End_DateNr,a.[Per Product Spend],a.All_Date,a.[No. Of Days In Week]
from Campaign_Data13_CS a
union all
select a.[channel ],a.Category,a.Product2 as Product,a.Campaign,a.Start_DateNr,a.End_DateNr,a.[Per Product Spend],a.All_Date,a.[No. Of Days In Week]
from Campaign_Data13_CS a
union all
select a.[channel ],a.Category,a.Product3 as Product,a.Campaign,a.Start_DateNr,a.End_DateNr,a.[Per Product Spend],a.All_Date,a.[No. Of Days In Week]
from Campaign_Data13_CS a) a) a
where a.Indicatore = 'Keep'

select *
from Campaign_Final_Output
order by Campaign


