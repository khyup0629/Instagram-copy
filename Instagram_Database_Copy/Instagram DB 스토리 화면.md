# 스토리 화면

![image](https://user-images.githubusercontent.com/43658658/133195761-25d00a90-9c40-4024-a62d-c0cf6c58b864.png)

> <h3>스토리 화면(다른 사람, 24시간 이내에 올린 스토리)</h3>

``` mysql
-- 스토리 화면(다른 사람, 24시간 이내 올린 스토리)
select profileImageUrl as storyProfileImage,
       storyImageUserID as storyUserID,
       case
           when timestampdiff(second, storyImageCreatedAt, '2021-09-11 00:00:00') < 60
               then concat(timestampdiff(second, storyImageCreatedAt, '2021-09-11 00:00:00'), '초')
           when timestampdiff(minute, storyImageCreatedAt, '2021-09-11 00:00:00') < 60
               then concat(timestampdiff(minute, storyImageCreatedAt, '2021-09-11 00:00:00'), '분')
           when timestampdiff(hour, storyImageCreatedAt, '2021-09-11 00:00:00') < 24
               then concat(timestampdiff(hour, storyImageCreatedAt, '2021-09-11 00:00:00'), '시간')
           when timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 7
               then concat(timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00'), '일')
           else date_format(storyImageCreatedAt, '%m월 %d일')
           end as storyUploadTime,
       storyImageUrl as storyContent
from (select *
      from StoryImageTable
      where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
      union all
      select *
      from StoryVideoTable
      where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1) as allContentStory
         inner join ProfileTable
                    on profileUserID = storyImageUserID
where storyImageUserID != 'bllumusic'
order by storyImageStoryNo DESC;
```

![image](https://user-images.githubusercontent.com/43658658/133197177-88f928b3-0d1a-4f25-a49f-94bdb23b9c57.png)

> <h3>내 스토리 화면(24시간 이내에 올린 것만)</h3>

``` mysql
-- 24시간 이내에 올린 내 스토리의 스토리 번호
select storyImageStoryNo
from StoryImageTable
where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
  and storyImageUserID = 'bllumusic'
union all
select storyVideoStoryNo
from StoryVideoTable
where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
  and storyVideoUserID = 'bllumusic';

-- 내 스토리를 본 시청자 목록
select *
from StoryWatchingTable
where storyWatchingStoryNo = (select storyImageStoryNo
                              from StoryImageTable
                              where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
                                and storyImageUserID = 'bllumusic'
                              union all
                              select storyVideoStoryNo
                              from StoryVideoTable
                              where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
                                and storyVideoUserID = 'bllumusic');

-- 내 스토리를 본 시청자 수
select storyWatchingStoryNo, count(storyWatchingUserID)
from (select *
      from StoryWatchingTable
      where storyWatchingStoryNo = (select storyImageStoryNo
                                    from StoryImageTable
                                    where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
                                      and storyImageUserID = 'bllumusic'
                                    union all
                                    select storyVideoStoryNo
                                    from StoryVideoTable
                                    where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
                                      and storyVideoUserID = 'bllumusic')) as bllumusicStoryWatcher
group by storyWatchingStoryNo;

-- 내 스토리를 가장 최근에 시청한 시청 여부 고유 번호
select storyWatchingStoryNo, max(storyWatchingNo) as lastBllumusicStoryWatching
from (select *
      from StoryWatchingTable
      where storyWatchingStoryNo = (select storyImageStoryNo
                                    from StoryImageTable
                                    where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
                                      and storyImageUserID = 'bllumusic'
                                    union all
                                    select storyVideoStoryNo
                                    from StoryVideoTable
                                    where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
                                      and storyVideoUserID = 'bllumusic')) as bllumusicStoryWatcher
group by storyWatchingStoryNo;

-- 내 스토리를 가장 최근에 시청한 유저 프로필 이미지
select storyWatchingNo, storyWatchingStoryNo, storyWatchingUserID, profileImageUrl
from StoryWatchingTable
         inner join ProfileTable
                    on profileUserID = storyWatchingUserID
where storyWatchingNo in (select max(storyWatchingNo) as lastBllumusicStoryWatching
                          from (select *
                                from StoryWatchingTable
                                where storyWatchingStoryNo = (select storyImageStoryNo
                                                              from StoryImageTable
                                                              where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
                                                                and storyImageUserID = 'bllumusic'
                                                              union all
                                                              select storyVideoStoryNo
                                                              from StoryVideoTable
                                                              where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
                                                                and storyVideoUserID = 'bllumusic')) as bllumusicStoryWatcher
                          group by storyWatchingStoryNo);

-- 내 스토리를 가장 최근에 시청한 유저를 제외한 시청 여부 리스트
select *
from StoryWatchingTable
where storyWatchingNo not in (select max(storyWatchingNo) as lastBllumusicStoryWatching
                              from (select *
                                    from StoryWatchingTable
                                    where storyWatchingStoryNo = (select storyImageStoryNo
                                                                  from StoryImageTable
                                                                  where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
                                                                    and storyImageUserID = 'bllumusic'
                                                                  union all
                                                                  select storyVideoStoryNo
                                                                  from StoryVideoTable
                                                                  where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
                                                                    and storyVideoUserID = 'bllumusic')) as bllumusicStoryWatcher
                              group by storyWatchingStoryNo);

-- 내 스토리를 가장 최근에서 두 번째로 시청한 고유 번호
select storyWatchingStoryNo, max(storyWatchingNo)
from (select *
      from StoryWatchingTable
      where storyWatchingNo not in (select max(storyWatchingNo) as lastBllumusicStoryWatching
                                    from (select *
                                          from StoryWatchingTable
                                          where storyWatchingStoryNo = (select storyImageStoryNo
                                                                        from StoryImageTable
                                                                        where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
                                                                          and storyImageUserID = 'bllumusic'
                                                                        union all
                                                                        select storyVideoStoryNo
                                                                        from StoryVideoTable
                                                                        where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
                                                                          and storyVideoUserID = 'bllumusic')) as bllumusicStoryWatcher
                                    group by storyWatchingStoryNo)
        and storyWatchingHost = 'bllumusic') as bllumusicSecondStoryWatcher
group by storyWatchingStoryNo;

-- 내 스토리를 가장 최근에서 두 번째로 본 유저의 프로필 이미지
select storyWatchingNo, storyWatchingStoryNo, storyWatchingUserID, profileImageUrl
from (select *
      from StoryWatchingTable
      where storyWatchingNo not in (select max(storyWatchingNo) as lastBllumusicStoryWatching
                                    from (select *
                                          from StoryWatchingTable
                                          where storyWatchingStoryNo = (select storyImageStoryNo
                                                                        from StoryImageTable
                                                                        where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
                                                                          and storyImageUserID = 'bllumusic'
                                                                        union all
                                                                        select storyVideoStoryNo
                                                                        from StoryVideoTable
                                                                        where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
                                                                          and storyVideoUserID = 'bllumusic')) as bllumusicStoryWatcher
                                    group by storyWatchingStoryNo)) as notInlastBllumusicStoryWatching
         inner join ProfileTable
                    on profileUserID = storyWatchingUserID
where storyWatchingNo in (select max(storyWatchingNo)
                          from (select *
                                from StoryWatchingTable
                                where storyWatchingNo not in (select max(storyWatchingNo) as lastBllumusicStoryWatching
                                                              from (select *
                                                                    from StoryWatchingTable
                                                                    where storyWatchingStoryNo =
                                                                          (select storyImageStoryNo
                                                                           from StoryImageTable
                                                                           where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
                                                                             and storyImageUserID = 'bllumusic'
                                                                           union all
                                                                           select storyVideoStoryNo
                                                                           from StoryVideoTable
                                                                           where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
                                                                             and storyVideoUserID = 'bllumusic')) as bllumusicStoryWatcher
                                                              group by storyWatchingStoryNo)
                                  and storyWatchingHost = 'bllumusic') as bllumusicSecondStoryWatcher
                          group by storyWatchingStoryNo);

-- 블루뮤직의 스토리별 가장 최근에 본 시청자 프로필 이미지 묶기
select storyWatchingStoryNo, group_concat(profileImageUrl, '') as lastBllumusicStoryWatcher
from (select storyWatchingNo, storyWatchingStoryNo, storyWatchingUserID, profileImageUrl
      from StoryWatchingTable
               inner join ProfileTable
                          on profileUserID = storyWatchingUserID
      where storyWatchingNo in (select max(storyWatchingNo) as lastBllumusicStoryWatching
                                from (select *
                                      from StoryWatchingTable
                                      where storyWatchingStoryNo = (select storyImageStoryNo
                                                                    from StoryImageTable
                                                                    where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
                                                                      and storyImageUserID = 'bllumusic'
                                                                    union all
                                                                    select storyVideoStoryNo
                                                                    from StoryVideoTable
                                                                    where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
                                                                      and storyVideoUserID = 'bllumusic')) as bllumusicStoryWatcher
                                group by storyWatchingStoryNo)
      union all
      select storyWatchingNo, storyWatchingStoryNo, storyWatchingUserID, profileImageUrl
      from (select *
            from StoryWatchingTable
            where storyWatchingNo not in (select max(storyWatchingNo) as lastBllumusicStoryWatching
                                          from (select *
                                                from StoryWatchingTable
                                                where storyWatchingStoryNo = (select storyImageStoryNo
                                                                              from StoryImageTable
                                                                              where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
                                                                                and storyImageUserID = 'bllumusic'
                                                                              union all
                                                                              select storyVideoStoryNo
                                                                              from StoryVideoTable
                                                                              where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
                                                                                and storyVideoUserID = 'bllumusic')) as bllumusicStoryWatcher
                                          group by storyWatchingStoryNo)) as notInlastBllumusicStoryWatching
               inner join ProfileTable
                          on profileUserID = storyWatchingUserID
      where storyWatchingNo in (select max(storyWatchingNo)
                                from (select *
                                      from StoryWatchingTable
                                      where storyWatchingNo not in
                                            (select max(storyWatchingNo) as lastBllumusicStoryWatching
                                             from (select *
                                                   from StoryWatchingTable
                                                   where storyWatchingStoryNo =
                                                         (select storyImageStoryNo
                                                          from StoryImageTable
                                                          where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
                                                            and storyImageUserID = 'bllumusic'
                                                          union all
                                                          select storyVideoStoryNo
                                                          from StoryVideoTable
                                                          where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
                                                            and storyVideoUserID = 'bllumusic')) as bllumusicStoryWatcher
                                             group by storyWatchingStoryNo)
                                        and storyWatchingHost = 'bllumusic') as bllumusicSecondStoryWatcher
                                group by storyWatchingStoryNo)) as lastBllumusicStoryWatcherTable
group by storyWatchingStoryNo;

-- 내 스토리 화면(24시간 이내 올린 스토리)
select ProfileTable.profileImageUrl as storyProfileImage,
       storyImageUserID             as storyUserID,
       case
           when timestampdiff(second, storyImageCreatedAt, '2021-09-11 00:00:00') < 60
               then concat(timestampdiff(second, storyImageCreatedAt, '2021-09-11 00:00:00'), '초')
           when timestampdiff(minute, storyImageCreatedAt, '2021-09-11 00:00:00') < 60
               then concat(timestampdiff(minute, storyImageCreatedAt, '2021-09-11 00:00:00'), '분')
           when timestampdiff(hour, storyImageCreatedAt, '2021-09-11 00:00:00') < 24
               then concat(timestampdiff(hour, storyImageCreatedAt, '2021-09-11 00:00:00'), '시간')
           when timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 7
               then concat(timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00'), '일')
           else date_format(storyImageCreatedAt, '%m월 %d일')
           end                      as storyUploadTime,
       storyImageUrl                as storyContent,
       lastBllumusicStoryWatcher,
       bllumusicStoryWatcherCnt
from (select *
      from StoryImageTable
      where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
        and storyImageUserID = 'bllumusic'
      union all
      select *
      from StoryVideoTable
      where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
        and storyVideoUserID = 'bllumusic') as allContentStory
         inner join ProfileTable
                    on profileUserID = storyImageUserID
         inner join (select storyWatchingStoryNo, group_concat(profileImageUrl, '') as lastBllumusicStoryWatcher
                     from (select storyWatchingNo, storyWatchingStoryNo, storyWatchingUserID, profileImageUrl
                           from StoryWatchingTable
                                    inner join ProfileTable
                                               on profileUserID = storyWatchingUserID
                           where storyWatchingNo in (select max(storyWatchingNo) as lastBllumusicStoryWatching
                                                     from (select *
                                                           from StoryWatchingTable
                                                           where storyWatchingStoryNo = (select storyImageStoryNo
                                                                                         from StoryImageTable
                                                                                         where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
                                                                                           and storyImageUserID = 'bllumusic'
                                                                                         union all
                                                                                         select storyVideoStoryNo
                                                                                         from StoryVideoTable
                                                                                         where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
                                                                                           and storyVideoUserID = 'bllumusic')) as bllumusicStoryWatcher
                                                     group by storyWatchingStoryNo)
                           union all
                           select storyWatchingNo, storyWatchingStoryNo, storyWatchingUserID, profileImageUrl
                           from (select *
                                 from StoryWatchingTable
                                 where storyWatchingNo not in (select max(storyWatchingNo) as lastBllumusicStoryWatching
                                                               from (select *
                                                                     from StoryWatchingTable
                                                                     where storyWatchingStoryNo =
                                                                           (select storyImageStoryNo
                                                                            from StoryImageTable
                                                                            where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
                                                                              and storyImageUserID = 'bllumusic'
                                                                            union all
                                                                            select storyVideoStoryNo
                                                                            from StoryVideoTable
                                                                            where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
                                                                              and storyVideoUserID = 'bllumusic')) as bllumusicStoryWatcher
                                                               group by storyWatchingStoryNo)) as notInlastBllumusicStoryWatching
                                    inner join ProfileTable
                                               on profileUserID = storyWatchingUserID
                           where storyWatchingNo in (select max(storyWatchingNo)
                                                     from (select *
                                                           from StoryWatchingTable
                                                           where storyWatchingNo not in
                                                                 (select max(storyWatchingNo) as lastBllumusicStoryWatching
                                                                  from (select *
                                                                        from StoryWatchingTable
                                                                        where storyWatchingStoryNo =
                                                                              (select storyImageStoryNo
                                                                               from StoryImageTable
                                                                               where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
                                                                                 and storyImageUserID = 'bllumusic'
                                                                               union all
                                                                               select storyVideoStoryNo
                                                                               from StoryVideoTable
                                                                               where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
                                                                                 and storyVideoUserID = 'bllumusic')) as bllumusicStoryWatcher
                                                                  group by storyWatchingStoryNo)
                                                             and storyWatchingHost = 'bllumusic') as bllumusicSecondStoryWatcher
                                                     group by storyWatchingStoryNo)) as lastBllumusicStoryWatcherTable
                     group by storyWatchingStoryNo) as lastStoryWatcherTable
                    on lastStoryWatcherTable.storyWatchingStoryNo = allContentStory.storyImageStoryNo
         inner join (select storyWatchingStoryNo, count(storyWatchingUserID) as bllumusicStoryWatcherCnt
                     from (select *
                           from StoryWatchingTable
                           where storyWatchingStoryNo = (select storyImageStoryNo
                                                         from StoryImageTable
                                                         where timestampdiff(day, storyImageCreatedAt, '2021-09-11 00:00:00') < 1
                                                           and storyImageUserID = 'bllumusic'
                                                         union all
                                                         select storyVideoStoryNo
                                                         from StoryVideoTable
                                                         where timestampdiff(day, storyVideoCreatedAt, '2021-09-11 00:00:00') < 1
                                                           and storyVideoUserID = 'bllumusic')) as bllumusicStoryWatcher
                     group by storyWatchingStoryNo) as bllumusicStoryWatcherCntTable
                    on bllumusicStoryWatcherCntTable.storyWatchingStoryNo = allContentStory.storyImageStoryNo
order by storyImageStoryNo DESC;
```

![image](https://user-images.githubusercontent.com/43658658/133201316-fa0234ff-cdee-43d6-9e44-1f55315858b7.png)

# 스토리 시청자 화면

![image](https://user-images.githubusercontent.com/43658658/133206845-46ae9837-222a-4dfa-9e16-73389bc377c9.png)

> <h3>스토리 시청자 목록 화면 상단(컨텐츠, 시청자 수)</h3>

![image](https://user-images.githubusercontent.com/43658658/133205672-a6b5cad6-6ed0-4c55-89eb-ee32475807ff.png)

``` mysql
-- 스토리 별 컨텐츠 URL
select storyImageStoryNo, storyImageUrl
from StoryImageTable
union all
select storyVideoStoryNo, storyVideoUrl
from StoryVideoTable
order by storyImageStoryNo DESC;

-- 내 스토리(12번)를 본 시청자 수
select storyWatchingStoryNo, count(storyWatchingUserID) as storyWatcher12
from (select *
      from StoryWatchingTable
      where storyWatchingStoryNo = (select storyImageStoryNo
                                    from StoryImageTable
                                    where StoryImageTable.storyImageStoryNo = 12
                                    union all
                                    select storyVideoStoryNo
                                    from StoryVideoTable
                                    where StoryVideoTable.storyVideoStoryNo = 12)) as bllumusicStoryWatcher
group by storyWatchingStoryNo;

-- 내 스토리(12번) 시청자 목록 화면 상단(컨텐츠와 시청자 수)
select storyImageUrl, storyWatcher12
from (select storyImageStoryNo, storyImageUrl
      from StoryImageTable
      union all
      select storyVideoStoryNo, storyVideoUrl
      from StoryVideoTable
      order by storyImageStoryNo DESC) as allStoryContent
         inner join (select storyWatchingStoryNo, count(storyWatchingUserID) as storyWatcher12
                     from (select *
                           from StoryWatchingTable
                           where storyWatchingStoryNo = (select storyImageStoryNo
                                                         from StoryImageTable
                                                         where StoryImageTable.storyImageStoryNo = 12
                                                         union all
                                                         select storyVideoStoryNo
                                                         from StoryVideoTable
                                                         where StoryVideoTable.storyVideoStoryNo = 12)) as bllumusicStoryWatcher
                     group by storyWatchingStoryNo) as storyWatcher12Table
                    on storyWatcher12Table.storyWatchingStoryNo = allStoryContent.storyImageStoryNo
where storyImageStoryNo = 12;
```

![image](https://user-images.githubusercontent.com/43658658/133204811-121c0e59-e723-4401-b229-7b1a4f2431f6.png)

> <h3>스토리 시청자 목록</h3>

![image](https://user-images.githubusercontent.com/43658658/133205713-2f735aea-6f09-4149-892c-07f20dc5e429.png)

``` mysql
-- 내 스토리(12번)를 본 시청자 목록
select profileImageUrl,
       if(storyWatchingUserID in (select storyUserID
                                  from StoryTable
                                  where timestampdiff(day, storyCreatedAt, '2021-09-11 00:00:00') < 1
                                    and 'No' =
                                        if(storyNo in (select storyWatchingStoryNo
                                                       from StoryWatchingTable
                                                       where storyWatchingUserID = 'bllumusic'),
                                           'Yes', 'No')),
          'No',
          'Yes') as isStoryWatching,
       profileUserID,
       profileName
from StoryWatchingTable
         inner join ProfileTable
                    on profileUserID = storyWatchingUserID
where StoryWatchingTable.storyWatchingStoryNo = (select storyImageStoryNo
                                                 from StoryImageTable
                                                 where StoryImageTable.storyImageStoryNo = 12
                                                 union all
                                                 select storyVideoStoryNo
                                                 from StoryVideoTable
                                                 where StoryVideoTable.storyVideoStoryNo = 12)
order by storyWatchingNo DESC;
```

![image](https://user-images.githubusercontent.com/43658658/133205007-e284a723-e583-4475-8e70-7e298628410f.png)

> <h3>하이라이트 화면</h3>

![image](https://user-images.githubusercontent.com/43658658/133205834-c83d840e-3399-4912-99bc-eab837b38f02.png)

``` mysql
-- 하이라이트(3번) 화면
select hlCoverImageUrl,
       hlTitle,
       case
           when timestampdiff(second, hlStoryCreatedAt, '2021-09-11 00:00:00') < 60
               then concat(timestampdiff(second, hlStoryCreatedAt, '2021-09-11 00:00:00'), '초')
           when timestampdiff(minute, hlStoryCreatedAt, '2021-09-11 00:00:00') < 60
               then concat(timestampdiff(minute, hlStoryCreatedAt, '2021-09-11 00:00:00'), '분')
           when timestampdiff(hour, hlStoryCreatedAt, '2021-09-11 00:00:00') < 24
               then concat(timestampdiff(hour, hlStoryCreatedAt, '2021-09-11 00:00:00'), '시간')
           when timestampdiff(day, hlStoryCreatedAt, '2021-09-11 00:00:00') < 7
               then concat(timestampdiff(day, hlStoryCreatedAt, '2021-09-11 00:00:00'), '일')
           else concat(timestampdiff(week, hlStoryCreatedAt, '2021-09-11 00:00:00'), '주')
           end                      as hlStoryCreatedAt,
       hlContentUrl
from HighlightTable
where hlNo = 3;
```

![image](https://user-images.githubusercontent.com/43658658/133205558-ea98344b-b0d5-4d03-8468-6824649150d1.png)
