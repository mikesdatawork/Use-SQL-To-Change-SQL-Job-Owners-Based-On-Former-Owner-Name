![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Use SQL To Change SQL Job Owners Based On Former Owner Name
**Post Date: December 9, 2015**        



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>The following logic checks for a particular user names that are possible SQL Job owners. Then returns a list to you of the found Jobs with said owners. Then it changes those Job owners to 'sa', and runs the list again. If no jobs are returned then the Job owners were changed accordingly. It's always a good idea to run the query again using the new 'sa' to ensure all Jobs return the proper 'sa' as the owner name.</p>      


## SQL-Logic
```SQL
use master;
set nocount on
 
-- create temp table to hold job info
if object_id('tempdb..#job_owners') is not null
drop table #job_owners
create table #job_owners
(
server_name varchar(255)
, job_owner_name varchar(255)
, job_name varchar(255)
, date_created datetime
, is_enabled varchar(255)
)
insert into #job_owners
select
'server_name' = @@servername
, 'job_owner_name' = upper(suser_sname(owner_sid))
, 'job_name' = upper(sj.name)
, 'date_created' = date_created
, 'is_enabled' =
case enabled
when '0' then 'NOPE'
when '1' then 'YES'
end
from
msdb..sysjobs sj
 
-- query indo directly from temp table
select
server_name
, job_owner_name
, job_name
, 'date_created' = left(date_created, 12) + ' ' + datename(dw, date_created)
, is_enabled
from
#job_owners
where
job_owner_name in ('MyDomain\MyUserName1', 'MyDomain\MyUserName2') -- &lt;-- find jobs with these owners
order by
date_created
, job_name asc
 
-- change database owner names for the above found jobs with said owners. change job owners to 'sa'
use msdb;
set nocount on
declare @change_job_owners varchar(max)
set @change_job_owners = ''
select @change_job_owners = @change_job_owners +
'exec msdb..sp_update_job @Job_name = ''' + job_name + ''', @owner_login_name = ''sa'';' + char(10)
from #job_owners
exec (@change_job_owners)
 
-- run the query again to confirm jobs have been changed, and nothing is returned.
 
if object_id('tempdb..#job_owners') is not null
drop table #job_owners
create table #job_owners
(
server_name varchar(255)
, job_owner_name varchar(255)
, job_name varchar(255)
, date_created datetime
, is_enabled varchar(255)
)
insert into #job_owners
select
'server_name' = @@servername
, 'job_owner_name' = upper(suser_sname(owner_sid))
, 'job_name' = upper(sj.name)
, 'date_created' = date_created
, 'is_enabled' =
case enabled
when '0' then 'NOPE'
when '1' then 'YES'
end
from
msdb..sysjobs sj
 
select
server_name
, job_owner_name
, job_name
, 'date_created' = left(date_created, 12) + ' ' + datename(dw, date_created)
, is_enabled
from
#job_owners
where
job_owner_name in ('MyDomain\MyUserName1', 'MyDomain\MyUserName2') -- &lt;-- find jobs with these owners
order by
date_created
, job_name asc
 
drop table #job_owners
```




[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

 
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

