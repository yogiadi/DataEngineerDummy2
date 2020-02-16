# Data Engineer Interview Questions

[Sequence of numbers](#sequence-of-numbers)

## Distance between two cities

```sql
drop table dist ;
create table dist (src varchar2(50), dest varchar2(50), distance number);

insert into dist values ('DEL','KOL',700);
insert into dist values ('KOL','DEL',700);
insert into dist values ('MUM','DEL',800);
insert into dist values ('DEL','MUM',800);
insert into dist values ('BAN','DEL',780);
```
### Remove duplicate distance two cities

<details>
<summary>Answer</summary>

```sql
Select
case when src < dest then src else dest end ,
case when src > dest then src else dest end
from 
dist 
group by 
case when src < dest then src else dest end ,
case when src > dest then src else dest end
having count(*) >1;
```
</details>

## Sequence of numbers

<details>
<summary>Answer</summary>
  
```sql
with t(n) as 
(select 1 as n from dual
union all
select n+1 as n from t where n < 25)
select * from t ;
```
</details>

## Employee timesheets

```sql
DROP TABLE TIMESHEETS;
CREATE TABLE TIMESHEETS(empid NUMBER , timestamps NUMBER , in_out VARCHAR2(50));

insert into timesheets values(1,830,'IN');
insert into timesheets values(2,900,'IN');
insert into timesheets values(3,930,'IN');
insert into timesheets values(1,945,'OUT');
insert into timesheets values(2,946,'OUT');
insert into timesheets values(4,1000,'IN');
```

<details>
<summary>Answer 1</summary>
  
```sql
select count(*) from 
(select dense_rank() over(partition by empid order by timestamps desc) rn , in_out from timesheets t)
where in_out='IN' and rn=1 ;
```
</details>

<details>
<summary>Answer 2</summary>
  
```sql
select sum(time_diff), empid 
from
(select 
lead(timestamps) over(partition by empid order by timestamps) - timestamps time_diff , empid , in_out from timesheets)
where in_out='IN'
group by empid ;
```
</details>

## Batsman Outscoring Himself

```sql
drop table ipl_batsman_score ;
create table ipl_batsman_score
(batsman_id number , match_id number, score number);

insert all
into ipl_batsman_score values (1,1,10)
into ipl_batsman_score values (1,2,15)
into ipl_batsman_score values (1,3,100)
into ipl_batsman_score values (2,1,101)
into ipl_batsman_score values (2,2,115)
into ipl_batsman_score values (2,3,90)
into ipl_batsman_score values (1,4,45)
into ipl_batsman_score values (1,5,155)
into ipl_batsman_score values (1,6,60)
into ipl_batsman_score values (2,4,10)
into ipl_batsman_score values (2,5,20)
into ipl_batsman_score values (2,6,100)
select 1 from dual;
```

<details>
<summary>Answer</summary>
  
```sql
select * from ipl_batsman_score ibs1 where ibs1.score > ALL(select ibs2.score from ipl_batsman_score ibs2 where ibs1.batsman_id=ibs2.batsman_id and ibs1.match_id > ibs2.match_id);
```
</details>


## Begin and End Price 

```sql
drop table price ;
create table price ( product_id varchar2(50), price number , effective_date date);

insert into price values (1,400,to_Date('01-Jan-2019','DD-MON-YYYY'));
insert into price values (2,300,to_Date('01-Jan-2019','DD-MON-YYYY'));
insert into price values (1,400,to_Date('02-Jan-2019','DD-MON-YYYY'));
insert into price values (2,300,to_Date('02-Jan-2019','DD-MON-YYYY'));
insert into price values (1,500,to_Date('03-Jan-2019','DD-MON-YYYY'));
insert into price values (2,300,to_Date('03-Jan-2019','DD-MON-YYYY'));
insert into price values (1,500,to_Date('04-Jan-2019','DD-MON-YYYY'));
insert into price values (2,300,to_Date('04-Jan-2019','DD-MON-YYYY'));
insert into price values (1,600,to_Date('05-Jan-2019','DD-MON-YYYY'));
insert into price values (2,400,to_Date('05-Jan-2019','DD-MON-YYYY'));
insert into price values (1,600,to_Date('06-Jan-2019','DD-MON-YYYY'));
insert into price values (2,400,to_Date('06-Jan-2019','DD-MON-YYYY'));
```

<details>
<summary>Answer</summary>
  
```sql
select distinct product_id , price ,ranges , first_value(effective_Date) over(partition by ranges , product_id order by effective_Date rows between unbounded preceding and UNBOUNDED FOLLOWING ) begin_dt , 
last_value(effective_Date) over(partition by ranges, product_id order by effective_Date rows between unbounded preceding and UNBOUNDED FOLLOWING ) end_dt from(
select product_id , price , sum(case when price <> lag then 1 else 0 end)  over(partition by product_id order by effective_date) ranges , effective_Date from (
select product_id , price , lag(price) over(partition by product_id order by effective_date) lag, effective_Date from price )) order by 3;
```
</details>

## Sort in Python
#### Input arr=[8,6,10,4,1]

<details>
<summary>Answer</summary>
  
```python
def heapify(arr,n,i):
    largest = i
    l=2*i+1
    r=2*i+2
    if l < n and arr[largest] < arr[l]:
        largest=l
    if r < n and arr[largest] < arr[r]:
        largest=r
    if largest != i:
        arr[i],arr[largest]=arr[largest],arr[i]
        heapify(arr,n,largest)
def heapsort(arr):
    n=len(arr)
    for i in range(n,-1,-1):
        heapify(arr,n,i)
    for i in range(n-1,0,-1):
        arr[i],arr[0]=arr[0],arr[i]
        heapify(arr,i,0)
```
</details>

<details>
<summary>Answer</summary>
  
```python
def merge(arr,p,q,r):
    n1=q-p+1
    n2=r-q
    L,R=[None]*(n1+1),[None]*(n2+1)
    for i in range(0,n1):
        L[i]=arr[p+i]
    for j in range(0,n2):
        R[j]=arr[q+j+1]
    L[n1],R[n2]=math.inf,math.inf
    i,j=0,0
    for k in range(p,r+1):
        if L[i] <= R[j]:
            arr[k]=L[i]
            i=i+1
        else:
            arr[k]=R[j]
            j=j+1
def merge_sort(A,p,r):
    if p < r:
        q=(p+r)//2
        merge_sort(A,p,q)
        merge_sort(A,q+1,r)
        merge(A,p,q,r)  
```
</details>

## Execute commands on all servers and return if timeout occurs

<details>
<summary>Answer</summary>
  
```bash
ssh user1@server1 timeout 5 date
```
</details>

## Consecutive winning streak

```sql
create table MATCH_RESULTS
(
MATCH_ID NUMBER GENERATED ALWAYS AS IDENTITY,
TEAMA VARCHAR2(255),
TEAMB VARCHAR2(255),
WINNER VARCHAR2(255)
);

insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('IND','WI','WI');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('SA','WI','WI');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('IND','AUS','IND');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('WI','ENG','WI');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('PAK','WI','PAK');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('IND','SA','IND');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('SA','ENG','ENG');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('ENG','AUS','AUS');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('AUS','PAK','AUS');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('PAK','WI','WI');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('WI','IND','IND');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('IND','ENG','IND');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('IND','PAK','IND');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('WI','AUS','WI');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('ENG','WI','WI');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('ENG','SA','ENG');
insert into MATCH_RESULTS (TEAMA, TEAMB, WINNER) VALUES('ENG','IND','ENG');
```

<details>
<summary>Answer</summary>
  
```sql
select 
team ,
lag_winner,
sum(case when winner <> team or winner <> lag_winner then 1 else 0 end) over(partition by team order by match_id rows unbounded preceding) cond ,
match_id,
winner 
from (
with teams as
(select teama team from match_results mr1  
union 
select teamb team from match_results mr2 )
select 
team ,
lag(mr.winner) over(partition by team order by match_id) lag_winner,
--sum(case when mr.winner <> team  or mr.winner <> lag(mr.winner) over(partition by team order by match_id)  then 1 else 0 end) over(partition by team order by match_id rows unbounded preceding) cond ,
match_id,
mr.winner 
from teams t cross join match_results mr 
where t.team = mr.teama or t.team=mr.teamb 
)
order by 1,match_id;
```
</details>

## Users were active in the month of January.

```sql
create table user_details 
( user_id NUMBER GENERATED ALWAYS AS IDENTITY,
start_dt DATE , 
end_dt DATE );

insert into user_Details(start_dt,end_dt) values ('01-Jul-2019','01-Aug-2019');
insert into user_Details(start_dt,end_dt) values ('01-Jul-2019','01-Jan-2020');
insert into user_Details(start_dt,end_dt) values ('01-Jan-2020','31-Jan-2020');
insert into user_Details(start_dt,end_dt) values ('15-Jan-2020','01-Feb-2020');
insert into user_Details(start_dt,end_dt) values ('01-Feb-2020','10-Feb-2020');
insert into user_Details(start_dt,end_dt) values ('01-Jul-2019','10-Feb-2020');
```

<details>
<summary>Answer</summary>
  
```sql
select * from user_details where end_dt >= '01-Jan-2020' and start_dt <='31-Jan-2020' ; 
```
</details>

## Worst case and Best case with joins 

```sql
DROP TABLE tb1;
DROP TABLE tb2;
CREATE TABLE TB1(
pk NUMBER);

CREATE TABLE TB2(
pk NUMBER);

insert into tb1 values(1);
insert into tb1 values(1);
insert into tb1 values(1);
insert into tb1 values(2);
insert into tb1 values(3);

insert into tb2 values(1);
insert into tb2 values(1);
insert into tb2 values(3);
insert into tb2 values(4);

select * from tb1 inner join tb2 using(pk) ;
select * from tb1 left outer join tb2 using(pk) ;
select * from tb1 right outer join tb2 using(pk) ;
select * from tb1 full outer join tb2 using(pk) ;
select * from tb1 cross join tb2 ;
```
<details>
<summary>Answer</summary>
7, 8, 8, 9, 20
</details>

## Find a file and archive it 

<details>
<summary>Answer</summary>
 
```bash
find <Path_To_Old_Files> -type f -mtime +30 | xargs rm -f
```
</details>

## Incremental data load

<details>
<summary>Answer</summary>
 
Use audit dimension or hash values to compare data.

</details>

## Difference between range and xrange.

<details>
<summary>Answer</summary>
 
range returns list and xrange returns generator objects.

</details>

## Use awk to divide the file between even and odd number of lines


<details>
<summary>Answer</summary>
 
```bash
awk '{ if ( NR % 2 == 0 ) print $0 >> "even.txt";if ( NR % 2 == 1 ) print $0 >> "odd.txt" }' traceexecution.py
awk '{ if ( NR % 2 == 0 ) print $0 >> "even.txt" ;else print $0 >> "odd.txt" }' traceexecution.py
```

</details>

## Delete empty records in a file

<details>
<summary>Answer</summary>
 
```bash
sed '/^$/d' even.txt
```

</details>

## Three consecutive seats availability in theatres.

```sql
create table seats(
seat_id NUMBER GENERATED ALWAYS AS IDENTITY,
AVAILABILITY VARCHAR2(1)
);

insert into seats(availability) values('Y');
insert into seats(availability) values('Y');
insert into seats(availability) values('N');
insert into seats(availability) values('Y');
insert into seats(availability) values('N');
insert into seats(availability) values('N');
insert into seats(availability) values('Y');
insert into seats(availability) values('Y');
insert into seats(availability) values('Y');
insert into seats(availability) values('Y');
insert into seats(availability) values('N');
insert into seats(availability) values('N');
insert into seats(availability) values('Y');
insert into seats(availability) values('Y');
insert into seats(availability) values('Y');
insert into seats(availability) values('N');
```

<details>
<summary>Answer</summary>
 
```sql
select * from seats s1 where  s1.availability ='Y' and 'Y'= ALL(select s2.availability from seats s2 where s2.seat_id between s1.seat_id and s1.seat_id + 2 ) order by 1 ;
```

</details>

## Create all combinations of teams from one table.

```sql
create table teams(
team varchar2(10)
);

insert into teams values('IND');
insert into teams values('SA');
insert into teams values('ENG');
insert into teams values('AUS');
```
<details>
<summary>Answer</summary>
 
```sql
select t1.team, t2.team from teams t1 cross join teams t2 where t1.team <> t2.team and t1.team < t2.team ;
```

</details>

## How to handle late arriving dimensions ?

<details>
<summary>Answer</summary>
By extracting source system id and create other attributes as UNKNOWN and update it later.
</details>

## How to reload corrupt data as quickly as possible ?

<details>
<summary>Answer</summary>
By using audit dimension and delete them .
</details>

## Frequently brought products in a table.

```sql
create table order_line
(
order_id number ,
order_line_id number,
product varchar2(1)
);

insert into order_line values(1, 1, 'A');
insert into order_line values(1, 2, 'B');
insert into order_line values(1, 3, 'C');
insert into order_line values(1, 4, 'D');

insert into order_line values(2, 1, 'B');
insert into order_line values(2, 2, 'C');

insert into order_line values(3, 1, 'D');
insert into order_line values(3, 2, 'A');

insert into order_line values(4, 1, 'B');
insert into order_line values(4, 2, 'A');

insert into order_line values(5, 1, 'C');
insert into order_line values(5, 2, 'A');

insert into order_line values(6, 1, 'C');
insert into order_line values(6, 2, 'A');
insert into order_line values(6, 3, 'B');

insert into order_line values(7, 1, 'A');
insert into order_line values(7, 2, 'C');
insert into order_line values(7, 3, 'D');
```
<details>
<summary>Answer</summary>
  
```sql
select 
case when ol1.product < ol2.product then ol1.product else ol2.product end prodA , 
case when ol1.product < ol2.product then ol2.product else ol1.product end prodB,
count(*)  from order_line ol1 inner join  order_line ol2 
on ol1.order_id=ol2.order_id where ol1.order_line_id<ol2.order_line_id 
group by 
case when ol1.product < ol2.product then ol1.product else ol2.product end , 
case when ol1.product < ol2.product then ol2.product else ol1.product end ;
```
</details>

## Design a movie booking system

<details>
<summary>Answer</summary>
  
```sql
create table movie(
movie_id number,
movie_name varchar2(255),
movie_language varchar2(100));

create table auditorium(
auditorium_id number,
theatre_id number);

create table theatre(
theatre_id number,
theatre_name varchar2(100));

create table seats(
seat_id number,
row_number number,
column_no varchar2(1),
auditorium_id number);

create table screening(
screening_id number,
auditorium_id number,
movie_id number,
starttime date);

create table reservation(
reservation_id number,
customer_id number,
payment_type number 
);

create table seat_booking(
seat_id number,
screening_id number,
reservation_id number);

```
</details>

## Create a data model for IPL considering we have to calculate how many batsman have scored more than 15% runs as compared to last year.

<details>
<summary>Answer</summary>
  
```sql
select 
batsmen_id,
year
from(
select 
batsmen_id, 
year, 
(lag(sum_run) over(partition by batsmen_id, year) - sum_run)/sum_run *100 inc_pct
(
select sum(runs) sum_run , batsmen_id, year  from runs 
group by batsmen_id, year )) where inc_pct > 15;

```
</details>

## Check which department has more combined salary than 500000.

<details>
<summary>Answer</summary>
  
```sql
select d.department_id from 
employees e 
join departments d 
on e.department_id=d.department_id
group by d.department
having sum(e.salary) > 500000;
```
</details>

## Student class and schools.

<details>
<summary>Answer</summary>
Preaggregate class and school attendance.
</details>

## Role Playing Dimensions.

<details>
<summary>Answer</summary>
One dimension which can be used in multiple ways just like Date dimension.
</details>

## Compare current week’s data to previous year’s  corresponding week’s data.

<details>
<summary>Answer</summary>
  
```sql
select (next_day(trunc(sysdate+10),'MONDAY') - next_day(trunc(sysdate,'YYYY')- 7 , 'MONDAY'))/7 from dual;
```
</details>

## SQL for Merge SCD2 Type

```sql

create table stg_dim(
natural_key varchar2(100),
col1 varchar2(100),
col2 varchar2(100));

create table dim(
dim_key number,
natural_key varchar2(100),
col1 varchar2(100),
col2 varchar2(100),
eff_dt date ,
exp_dt date default to_date('99991231','YYYYMMDD') NOT NULL);

drop table dim ;
drop table stg_dim ;

insert into stg_dim values ('ABC','a','b');
insert into stg_dim values ('XYZ','g','r');
insert into stg_dim values ('YUI','d','o');

insert into dim values(1,'ABC','a','c',to_date('20190701','YYYYMMDD'),to_date('20190708','YYYYMMDD'));
insert into dim values(2,'ABC','a','d',to_date('20190709','YYYYMMDD'),to_date('99991231','YYYYMMDD'));
insert into dim values(3,'XYZ','g','s',to_date('20190706','YYYYMMDD'),to_date('99991231','YYYYMMDD'));

```

<details>
<summary>Answer</summary>
  
```sql

merge into dim
using 
(
select s1.*, 'I' upd_ind  from stg_dim s1
union all
select s2.*, 'U' upd_ind  from stg_dim s2
) stg
on (stg.natural_key = dim.natural_key and dim.exp_dt=to_date('99991231','YYYYMMDD') and stg.upd_ind='U')
when matched then 
update set dim.col1=stg.col1, dim.col2=stg.col2
when not matched then
insert (dim.dim_key, dim.natural_key, dim.col1, dim.col2, dim.eff_dt, dim.exp_dt ) values(1,stg.natural_key,stg.col1,stg.col2,trunc(sysdate),to_date('99991231','YYYYMMDD'));

select * from dim ;
```

</details>
  
## Hotel reservation data model

<details>
<summary>Answer</summary>

```sql
create table hotel_tables(
hotel_table_id number,
hotel_table_capacity number);

select * 
from 
hotel_tables 
left outer join 
booking 
on ht.hotel_table_id=b.hotel_table_id
and b.date=:date
and b.time between :time-15 and :time+15
where 
b.hotel_table_id is null
and ht.hotel_table_capacity = :capacity;
```

</details>

## Last 3 orders of a customer

<details>
<summary>Answer</summary>

```sql
create table orders(
order_id number,
customer_id number,
order_date date,
order_time timestamp(6),
order_amount number);

create table order_line(
order_id number,
order_line_id number,
product_id number,
delivery_address_id number);

create table deliver_address(
delivery_address_id number,
address_line_1 varchar2(50),
address_line_2 varchar2(50),
address_line_3 varchar2(50),
pin number,
state varchar2(50));

select * from 
orders join
order_line  using(order_id)
join 
deliver_address using(delivery_address_id)
where customer_id= :customer_id ;

```
</details>

## Top five views per year

<details>
<summary>Answer</summary>

```sql
create table song_dim (
song_id number,
song_name varchar2(255));

create table views_fact(
song_id number,
views number,
eff_dt date)

create table date_dim(
eff_dt date,
eff_year number
);


select song_id , eff_year , song_pop from(
select song_id , eff_year , dense_rank() over(partition by song_id, eff_year order by song_pop desc) rnk, song_pop from(
select 
vf.song_id , dd.eff_year , count(vf.views) song_pop
from views_fact vf
join date_dim dd
using(eff_dt)
-- join song_dim sd
-- using(song_id)
group by vf.song_id , dd.eff_year)) where rnk >= 5;
```

</details>