# 활동 리스트

![image](https://user-images.githubusercontent.com/43658658/133030941-f9fb1d1a-cc58-4ab0-bf62-d6d3f010a7ba.png)

> <h3>좋아요 활동 리스트(한 명씩)</h3>

``` mysql
-- 블루뮤직의 피드를 좋아요한 유저(한 명씩), 시간 순서대로 내림차순
select clickLikeFeedNo,
       clickLikeUserID,
       case
           when timestampdiff(second, clickLikeCreatedAt, current_timestamp) < 60
               then concat(timestampdiff(second, clickLikeCreatedAt, current_timestamp), '초')
           when timestampdiff(minute, clickLikeCreatedAt, current_timestamp) < 60
               then concat(timestampdiff(minute, clickLikeCreatedAt, current_timestamp), '분')
           when timestampdiff(hour, clickLikeCreatedAt, current_timestamp) < 24
               then concat(timestampdiff(hour, clickLikeCreatedAt, current_timestamp), '시간')
           when timestampdiff(day, clickLikeCreatedAt, current_timestamp) < 7
               then concat(timestampdiff(day, clickLikeCreatedAt, current_timestamp), '일')
           else date_format(clickLikeCreatedAt, '%m월 %d일')
           end as lastFeedTimestamp
from ClickLikeTable
where clickLikeFeedNo in (select feedNo
                          from FeedTable
                          where feedHost = 'bllumusic')
order by clickLikeNo DESC;

-- 유저 프로필 이미지, 피드 이미지를 추가한 좋아요 활동 리스트(한 명씩)
select profileImageUrl,
       concat(clickLikeUserID, '님이 회원님의 게시물을 좋아합니다.') as actClickLikeUser,
       lastFeedTimestamp,
       firstUrlPerFeed
from (select clickLikeNo,
             clickLikeFeedNo,
             clickLikeUserID,
             case
                 when timestampdiff(second, clickLikeCreatedAt, current_timestamp) < 60
                     then concat(timestampdiff(second, clickLikeCreatedAt, current_timestamp), '초')
                 when timestampdiff(minute, clickLikeCreatedAt, current_timestamp) < 60
                     then concat(timestampdiff(minute, clickLikeCreatedAt, current_timestamp), '분')
                 when timestampdiff(hour, clickLikeCreatedAt, current_timestamp) < 24
                     then concat(timestampdiff(hour, clickLikeCreatedAt, current_timestamp), '시간')
                 when timestampdiff(day, clickLikeCreatedAt, current_timestamp) < 7
                     then concat(timestampdiff(day, clickLikeCreatedAt, current_timestamp), '일')
                 else date_format(clickLikeCreatedAt, '%m월 %d일')
                 end as lastFeedTimestamp
      from ClickLikeTable
      where clickLikeFeedNo in (select feedNo
                                from FeedTable
                                where feedHost = 'bllumusic')) as clickLikeOnePerson
         inner join ProfileTable
                    on ProfileTable.profileUserID = clickLikeOnePerson.clickLikeUserID
         inner join (select feedImageFeedNo, feedImageUrl as firstUrlPerFeed
                     from (select *
                           from FeedImageTable
                           where feedimageno in
                                 (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
                             and feedImageUserID = 'bllumusic'
                           union all
                           select *
                           from FeedVideoTable
                           where feedVideoNo in
                                 (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
                             and feedvideoUserID = 'bllumusic') as contentUrlPerFeed
                     where feedImageNo in (select min(feedImageNo)
                                           from (select *
                                                 from FeedImageTable
                                                 where feedimageno in
                                                       (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
                                                   and feedImageUserID = 'bllumusic'
                                                 union all
                                                 select *
                                                 from FeedVideoTable
                                                 where feedVideoNo in
                                                       (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
                                                   and feedvideoUserID = 'bllumusic') as onlyOneUrlPerFeed
                                           group by feedImageFeedNo)) as firstContentPerFeed
                    on firstContentPerFeed.feedImageFeedNo = clickLikeOnePerson.clickLikeFeedNo
order by clickLikeNo DESC;
```

![image](https://user-images.githubusercontent.com/43658658/133036981-1adfcdc8-335f-423c-88f6-b2c9de38e09b.png)

> <h3>게시글 별 좋아요 활동 리스트</h3>

![image](https://user-images.githubusercontent.com/43658658/133059140-6988406b-e0ae-4654-9aac-c55c38f93d3c.png)

``` mysql
-- 블루뮤직 피드별 이미지 URL
select *
from FeedImageTable
where feedimageno in (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
  and feedImageUserID = 'bllumusic';

-- 블루뮤직 피드별 동영상 URL
select *
from FeedVideoTable
where feedVideoNo in (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
  and feedvideoUserID = 'bllumusic';

-- 블루뮤직 피드별 이미지 + 동영상 번호
select *
from FeedImageTable
where feedimageno in
      (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
  and feedImageUserID = 'bllumusic'
union all
select *
from FeedVideoTable
where feedVideoNo in
      (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
  and feedvideoUserID = 'bllumusic';

-- 블루뮤직의 피드별 제일 첫 번째로 올린 이미지 or 동영상 번호
select feedImageFeedNo, feedImageUrl as firstUrlPerFeed
from (select *
      from FeedImageTable
      where feedimageno in
            (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
        and feedImageUserID = 'bllumusic'
      union all
      select *
      from FeedVideoTable
      where feedVideoNo in
            (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
        and feedvideoUserID = 'bllumusic') as contentUrlPerFeed
where feedImageNo in (select min(feedImageNo)
                      from (select *
                            from FeedImageTable
                            where feedimageno in
                                  (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
                              and feedImageUserID = 'bllumusic'
                            union all
                            select *
                            from FeedVideoTable
                            where feedVideoNo in
                                  (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
                              and feedvideoUserID = 'bllumusic') as onlyOneUrlPerFeed
                      group by feedImageFeedNo);

-- 블루뮤직의 피드를 좋아요 누른 유저들
select *
from ClickLikeTable
where clickLikeFeedNo in (select feedNo
                          from FeedTable
                          where feedHost = 'bllumusic');

-- 블루뮤직의 피드별 가장 최근 좋아요 누른 번호
select clickLikeFeedNo, max(clickLikeNo) as lastClickLikeNo
from (select *
      from ClickLikeTable
      where clickLikeFeedNo in (select feedNo
                                from FeedTable
                                where feedHost = 'bllumusic')) as lastClickLikeTable
group by clickLikeFeedNo;

-- 좋아요 고유 번호와 대응되는 유저 ID와 좋아요 누른 시간
select clickLikeFeedNo,
       clickLikeNo,
       clickLikeUserID,
       case
           when timestampdiff(second, clickLikeCreatedAt, current_timestamp) < 60
               then concat(timestampdiff(second, clickLikeCreatedAt, current_timestamp), '초')
           when timestampdiff(minute, clickLikeCreatedAt, current_timestamp) < 60
               then concat(timestampdiff(minute, clickLikeCreatedAt, current_timestamp), '분')
           when timestampdiff(hour, clickLikeCreatedAt, current_timestamp) < 24
               then concat(timestampdiff(hour, clickLikeCreatedAt, current_timestamp), '시간')
           when timestampdiff(day, clickLikeCreatedAt, current_timestamp) < 7
               then concat(timestampdiff(day, clickLikeCreatedAt, current_timestamp), '일')
           else date_format(clickLikeCreatedAt, '%m월 %d일')
           end as lastFeedTimestamp
from ClickLikeTable
where clickLikeNo in (select max(clickLikeNo) as lastClickLikeNo
                      from (select *
                            from ClickLikeTable
                            where clickLikeFeedNo in (select feedNo
                                                      from FeedTable
                                                      where feedHost = 'bllumusic')) as lastClickLikeTable
                      group by clickLikeFeedNo)
order by clickLikeNo DESC;

-- 블루 뮤직의 피드별 좋아요 수
select clickLikeFeedNo, count(clickLikeNo) as lastClickLikeCnt
from (select *
      from ClickLikeTable
      where clickLikeFeedNo in (select feedNo
                                from FeedTable
                                where feedHost = 'bllumusic')) as lastClickLikeTable
group by clickLikeFeedNo;

-- 가장 최근 좋아요를 기준으로 내림차순 정렬하면서 블루 뮤직의 피드별 좋아요 현황
select lastClickLikeUserAndTimestamp.clickLikeFeedNo                                           as actClickLikeFeed,
       if(lastClickLikeCnt = 1, concat(clickLikeUserID, '님이 좋아합니다.'),
          concat(clickLikeUserID, ' 외 ', clickLikeCntByFeed.lastClickLikeCnt - 1, '명이 좋아합니다')) as actClickLikeUserCnt,
       actRecentClickLikeTime
from (select clickLikeFeedNo,
             clickLikeNo,
             clickLikeUserID,
             case
                 when timestampdiff(second, clickLikeCreatedAt, current_timestamp) < 60
                     then concat(timestampdiff(second, clickLikeCreatedAt, current_timestamp), '초')
                 when timestampdiff(minute, clickLikeCreatedAt, current_timestamp) < 60
                     then concat(timestampdiff(minute, clickLikeCreatedAt, current_timestamp), '분')
                 when timestampdiff(hour, clickLikeCreatedAt, current_timestamp) < 24
                     then concat(timestampdiff(hour, clickLikeCreatedAt, current_timestamp), '시간')
                 when timestampdiff(day, clickLikeCreatedAt, current_timestamp) < 7
                     then concat(timestampdiff(day, clickLikeCreatedAt, current_timestamp), '일')
                 else date_format(clickLikeCreatedAt, '%m월 %d일')
                 end as actRecentClickLikeTime
      from ClickLikeTable
      where clickLikeNo in (select max(clickLikeNo) as lastClickLikeNo
                            from (select *
                                  from ClickLikeTable
                                  where clickLikeFeedNo in (select feedNo
                                                            from FeedTable
                                                            where feedHost = 'bllumusic')) as lastClickLikeTable
                            group by clickLikeFeedNo)) as lastClickLikeUserAndTimestamp
         inner join
     (select clickLikeFeedNo, count(clickLikeNo) as lastClickLikeCnt
      from (select *
            from ClickLikeTable
            where clickLikeFeedNo in (select feedNo
                                      from FeedTable
                                      where feedHost = 'bllumusic')) as lastClickLikeTable
      group by clickLikeFeedNo) as clickLikeCntByFeed
     on clickLikeCntByFeed.clickLikeFeedNo = lastClickLikeUserAndTimestamp.clickLikeFeedNo
order by clickLikeNo DESC;

-- 유저 프로필 이미지, 피드 이미지가 추가된 좋아요 현황
select ProfileTable.profileImageUrl,
       if(lastClickLikeCnt = 1, concat(clickLikeUserID, '님이 회원님의 게시글을 좋아합니다.'),
          concat(clickLikeUserID, ' 외 ', clickLikeCntByFeed.lastClickLikeCnt - 1,
                 '명이 회원님의 게시글을 좋아합니다.')) as actClickLikeUserCnt,
       actRecentClickLikeTime,
       firstFeedUrlTable.firstUrlPerFeed as actClickLikeFeed
from (select clickLikeFeedNo,
             clickLikeNo,
             clickLikeUserID,
             case
                 when timestampdiff(second, clickLikeCreatedAt, current_timestamp) < 60
                     then concat(timestampdiff(second, clickLikeCreatedAt, current_timestamp), '초')
                 when timestampdiff(minute, clickLikeCreatedAt, current_timestamp) < 60
                     then concat(timestampdiff(minute, clickLikeCreatedAt, current_timestamp), '분')
                 when timestampdiff(hour, clickLikeCreatedAt, current_timestamp) < 24
                     then concat(timestampdiff(hour, clickLikeCreatedAt, current_timestamp), '시간')
                 when timestampdiff(day, clickLikeCreatedAt, current_timestamp) < 7
                     then concat(timestampdiff(day, clickLikeCreatedAt, current_timestamp), '일')
                 else date_format(clickLikeCreatedAt, '%m월 %d일')
                 end as actRecentClickLikeTime
      from ClickLikeTable
      where clickLikeNo in (select max(clickLikeNo) as lastClickLikeNo
                            from (select *
                                  from ClickLikeTable
                                  where clickLikeFeedNo in (select feedNo
                                                            from FeedTable
                                                            where feedHost = 'bllumusic')) as lastClickLikeTable
                            group by clickLikeFeedNo)) as lastClickLikeUserAndTimestamp
         inner join
     (select clickLikeFeedNo, count(clickLikeNo) as lastClickLikeCnt
      from (select *
            from ClickLikeTable
            where clickLikeFeedNo in (select feedNo
                                      from FeedTable
                                      where feedHost = 'bllumusic')) as lastClickLikeTable
      group by clickLikeFeedNo) as clickLikeCntByFeed
     on clickLikeCntByFeed.clickLikeFeedNo = lastClickLikeUserAndTimestamp.clickLikeFeedNo
         inner join (select feedImageFeedNo, feedImageUrl as firstUrlPerFeed
                     from (select *
                           from FeedImageTable
                           where feedimageno in
                                 (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
                             and feedImageUserID = 'bllumusic'
                           union all
                           select *
                           from FeedVideoTable
                           where feedVideoNo in
                                 (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
                             and feedvideoUserID = 'bllumusic') as contentUrlPerFeed
                     where feedImageNo in (select min(feedImageNo)
                                           from (select *
                                                 from FeedImageTable
                                                 where feedimageno in
                                                       (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
                                                   and feedImageUserID = 'bllumusic'
                                                 union all
                                                 select *
                                                 from FeedVideoTable
                                                 where feedVideoNo in
                                                       (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
                                                   and feedvideoUserID = 'bllumusic') as onlyOneUrlPerFeed
                                           group by feedImageFeedNo)) as firstFeedUrlTable
                    on firstFeedUrlTable.feedImageFeedNo = lastClickLikeUserAndTimestamp.clickLikeFeedNo
         inner join ProfileTable
                    on ProfileTable.profileUserID = lastClickLikeUserAndTimestamp.clickLikeUserID
order by clickLikeNo DESC;
```

![image](https://user-images.githubusercontent.com/43658658/133030803-8e0b1aef-ff87-4366-8e4d-22e9865f8f47.png)

> <h3>팔로우 활동 리스트(한 명씩), 스토리 시청 여부에 따라 프로필 이미지 테두리 표시</h3>

``` mysql
-- 블루뮤직을 팔로우한 유저 목록(한 명씩), 시간 순으로 내림차순 정렬
select *
from FollowerTable
where followerHost = 'bllumusic'
order by followerNo DESC;

-- 24시간 이내(9월 11일 기준)로 스토리를 올린 리스트
select *
from StoryTable
where timestampdiff(day, storyCreatedAt, '2021-09-11 00:00:00') < 1;

-- 블루뮤직이 보지 않은 스토리, 보지 않은 스토리('No')는 프로필 이미지 테두리가 생김.
select storyUserID
from StoryTable
where timestampdiff(day, storyCreatedAt, '2021-09-11 00:00:00') < 1
  and 'No' =
      if(storyNo in (select storyWatchingStoryNo from StoryWatchingTable where storyWatchingUserID = 'bllumusic'),
         'Yes', 'No');

-- 유저 프로필 이미지를 포함한 팔로우 유저 목록(한 명씩), 프로필 이미지는 스토리를 봤는지 여부가 포함되어야 함
select profileImageUrl,
       if(followerUserID in (select storyUserID
                             from StoryTable
                             where timestampdiff(day, storyCreatedAt, '2021-09-11 00:00:00') < 1
                               and 'No' =
                                   if(storyNo in (select storyWatchingStoryNo
                                                  from StoryWatchingTable
                                                  where storyWatchingUserID = 'bllumusic'),
                                      'Yes', 'No')),
          'No',
          'Yes') as isStoryWatching,
       concat(followerUserID, '님이 회원님을 팔로우합니다.'),
       case
           when timestampdiff(second, followerCreatedAt, '2021-09-11 00:00:00') < 60
               then concat(timestampdiff(second, followerCreatedAt, '2021-09-11 00:00:00'), '초')
           when timestampdiff(minute, followerCreatedAt, '2021-09-11 00:00:00') < 60
               then concat(timestampdiff(minute, followerCreatedAt, '2021-09-11 00:00:00'), '분')
           when timestampdiff(hour, followerCreatedAt, '2021-09-11 00:00:00') < 24
               then concat(timestampdiff(hour, followerCreatedAt, '2021-09-11 00:00:00'), '시간')
           when timestampdiff(day, followerCreatedAt, '2021-09-11 00:00:00') < 7
               then concat(timestampdiff(day, followerCreatedAt, '2021-09-11 00:00:00'), '일')
           else date_format(followerCreatedAt, '%m월 %d일')
           end   as lastFeedTimestamp
from (select *
      from FollowerTable
      where followerHost = 'bllumusic') as followerListTable
         inner join ProfileTable
                    on ProfileTable.profileUserID = followerListTable.followerUserID
order by followerNo DESC;
```

![image](https://user-images.githubusercontent.com/43658658/133047395-75cc7422-944a-4f8d-824e-182de09809e6.png)

> <h3>일별 팔로우 활동 리스트(가장 최근 유저와 두 번째 최근 유저 나타내기)</h3>

![image](https://user-images.githubusercontent.com/43658658/133059242-bbe34790-b5ad-4772-adb1-047d3ac32173.png)

``` mysql
-- 일별로 블루뮤직을 팔로우한 가장 최근 번호
select max(followerNo), date_format(followerCreatedAt, '%Y-%m-%d')
from (select *
      from FollowerTable
      where followerHost = 'bllumusic') as followerBllumusic
group by date_format(followerCreatedAt, '%Y-%m-%d');

-- 가장 큰 번호와 대응되는 유저
select followerNo, followerUserID, followerCreatedAt
from FollowerTable
where followerNo in (select max(followerNo)
                     from (select *
                           from FollowerTable
                           where followerHost = 'bllumusic') as followerBllumusic
                     group by date_format(followerCreatedAt, '%Y-%m-%d'));

-- 블루 뮤직을 팔로우한 가장 최근 팔로우 번호보다 작은 리스트
select *
from FollowerTable
where followerNo not in (select max(followerNo)
                         from (select *
                               from FollowerTable
                               where followerHost = 'bllumusic') as followerBllumusic
                         group by date_format(followerCreatedAt, '%Y-%m-%d'))
  and followerHost = 'bllumusic';

-- 블루 뮤직을 팔로우한 가장 최근 팔로우 번호보다 작은 리스트 중에서 최대
-- 즉, 두 번째로 최근인 팔로우 번호
select max(followerNo), date_format(followerCreatedAt, '%Y-%m-%d')
from (select *
      from FollowerTable
      where followerNo not in (select max(followerNo)
                               from (select *
                                     from FollowerTable
                                     where followerHost = 'bllumusic') as followerBllumusic
                               group by date_format(followerCreatedAt, '%Y-%m-%d'))
        and followerHost = 'bllumusic') as lastRecentTwoFollowerNo
group by date_format(followerCreatedAt, '%Y-%m-%d');

-- 두 번째로 큰 번호와 대응되는 유저
select followerNo, followerUserID, followerCreatedAt
from FollowerTable
where followerNo in (select max(followerNo)
                     from (select *
                           from FollowerTable
                           where followerNo not in (select max(followerNo)
                                                    from (select *
                                                          from FollowerTable
                                                          where followerHost = 'bllumusic') as followerBllumusic
                                                    group by date_format(followerCreatedAt, '%Y-%m-%d'))
                             and followerHost = 'bllumusic') as lastRecentTwoFollowerNo
                     group by date_format(followerCreatedAt, '%Y-%m-%d'));

-- 블루 뮤직의 일별 팔로우 수
select date_format(followerCreatedAt, '%Y-%m-%d'), count(followerNo)
from (select * from FollowerTable where followerHost = 'bllumusic') as followerBllumusic
group by date_format(followerCreatedAt, '%Y-%m-%d');

-- 일별 블루뮤직의 팔로우 활동 리스트(유저 프로필 이미지 2개, 신규 팔로우 유저 수, 가장 최근 팔로우 시간
select lastFirstProfile.profileImageUrl      as actRecentUserProfileImage1,
       lastSecondProfile.profileImageUrl     as actRecentUserProfileImage2,
       if(followerCnt = 2, concat(lastFirstFollowerTable.followerUserID, '님, ',
                                  lastSecondFollowerTable.followerUserID, '님이 팔로우합니다.'),
          concat(lastFirstFollowerTable.followerUserID, '님, ',
                 lastSecondFollowerTable.followerUserID, '님 외 ',
                 followerCnt, '명이 회원님을 팔로우하기 시작했습니다.')) as actFollowUserCnt,
       case
           when timestampdiff(second, followerBllumusic.groupPerDay, '2021-09-11 00:00:00') < 60
               then concat(timestampdiff(second, followerBllumusic.groupPerDay, '2021-09-11 00:00:00'), '초')
           when timestampdiff(minute, followerBllumusic.groupPerDay, '2021-09-11 00:00:00') < 60
               then concat(timestampdiff(minute, followerBllumusic.groupPerDay, '2021-09-11 00:00:00'), '분')
           when timestampdiff(hour, followerBllumusic.groupPerDay, '2021-09-11 00:00:00') < 24
               then concat(timestampdiff(hour, followerBllumusic.groupPerDay, '2021-09-11 00:00:00'), '시간')
           when timestampdiff(day, followerBllumusic.groupPerDay, '2021-09-11 00:00:00') < 7
               then concat(timestampdiff(day, followerBllumusic.groupPerDay, '2021-09-11 00:00:00'), '일')
           else date_format(followerBllumusic.groupPerDay, '%m월 %d일')
           end                               as actRecentFollowTime
from (select date_format(followerCreatedAt, '%Y-%m-%d') as groupPerDay, count(followerNo) as followerCnt
      from (select * from FollowerTable where followerHost = 'bllumusic') as followerBllumusic
      group by date_format(followerCreatedAt, '%Y-%m-%d')) as followerBllumusic
         inner join (select followerNo, followerUserID, date_format(followerCreatedAt, '%Y-%m-%d') as groupPerDay
                     from FollowerTable
                     where followerNo in (select max(followerNo)
                                          from (select *
                                                from FollowerTable
                                                where followerHost = 'bllumusic') as followerBllumusic
                                          group by date_format(followerCreatedAt, '%Y-%m-%d'))) as lastFirstFollowerTable
                    on lastFirstFollowerTable.groupPerDay = followerBllumusic.groupPerDay
         inner join (select followerNo, followerUserID, date_format(followerCreatedAt, '%Y-%m-%d') as groupPerDay
                     from FollowerTable
                     where followerNo in (select max(followerNo)
                                          from (select *
                                                from FollowerTable
                                                where followerNo not in (select max(followerNo)
                                                                         from (select *
                                                                               from FollowerTable
                                                                               where followerHost = 'bllumusic') as followerBllumusic
                                                                         group by date_format(followerCreatedAt, '%Y-%m-%d'))
                                                  and followerHost = 'bllumusic') as lastRecentTwoFollowerNo
                                          group by date_format(followerCreatedAt, '%Y-%m-%d'))) as lastSecondFollowerTable
                    on lastSecondFollowerTable.groupPerDay = followerBllumusic.groupPerDay
         inner join ProfileTable as lastFirstProfile
                    on lastFirstProfile.profileUserID = lastFirstFollowerTable.followerUserID
         inner join ProfileTable as lastSecondProfile
                    on lastSecondProfile.profileUserID = lastSecondFollowerTable.followerUserID
order by lastSecondFollowerTable.followerNo DESC;
```

![image](https://user-images.githubusercontent.com/43658658/133059753-190806ed-34e4-436f-a75a-bfb79eab5538.png)

> <h3>피드별 댓글 활동 리스트</h3>

![image](https://user-images.githubusercontent.com/43658658/133075105-35ccf7b1-9439-4f42-bb08-207a5ea65d94.png)

``` mysql
-- 피드별 댓글 활동 리스트, 스토리 시청 여부 확인
select profileImageUrl                       as actUserProfileImage,
       if(replyUserID in (select storyUserID
                          from StoryTable
                          where timestampdiff(day, storyCreatedAt, '2021-09-11 00:00:00') < 1
                            and 'No' =
                                if(storyNo in (select storyWatchingStoryNo
                                               from StoryWatchingTable
                                               where storyWatchingUserID = 'bllumusic'),
                                   'Yes', 'No')),
          'No',
          'Yes')                             as isStoryWatching,
       concat(replyUserID, '님이 댓글을 남겼습니다: ') as actReplyUser,
       replyContent                          as actReplyContent,
       case
           when timestampdiff(second, replyCreatedAt, '2021-09-11 00:00:00') < 60
               then concat(timestampdiff(second, replyCreatedAt, '2021-09-11 00:00:00'), '초')
           when timestampdiff(minute, replyCreatedAt, '2021-09-11 00:00:00') < 60
               then concat(timestampdiff(minute, replyCreatedAt, '2021-09-11 00:00:00'), '분')
           when timestampdiff(hour, replyCreatedAt, '2021-09-11 00:00:00') < 24
               then concat(timestampdiff(hour, replyCreatedAt, '2021-09-11 00:00:00'), '시간')
           when timestampdiff(day, replyCreatedAt, '2021-09-11 00:00:00') < 7
               then concat(timestampdiff(day, replyCreatedAt, '2021-09-11 00:00:00'), '일')
           else date_format(replyCreatedAt, '%m월 %d일')
           end                               as actReplyTime,
       replyIsHeart,    
       feedFirstUrl.firstUrlPerFeed          as actClickLikeFeedImage
from ReplyTable
         inner join ProfileTable
                    on profileUserID = ReplyTable.replyUserID
         inner join (select feedImageFeedNo, feedImageUrl as firstUrlPerFeed
                     from (select *
                           from FeedImageTable
                           where feedimageno in
                                 (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
                             and feedImageUserID = 'bllumusic'
                           union all
                           select *
                           from FeedVideoTable
                           where feedVideoNo in
                                 (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
                             and feedvideoUserID = 'bllumusic') as contentUrlPerFeed
                     where feedImageNo in (select min(feedImageNo)
                                           from (select *
                                                 from FeedImageTable
                                                 where feedimageno in
                                                       (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
                                                   and feedImageUserID = 'bllumusic'
                                                 union all
                                                 select *
                                                 from FeedVideoTable
                                                 where feedVideoNo in
                                                       (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
                                                   and feedvideoUserID = 'bllumusic') as onlyOneUrlPerFeed
                                           group by feedImageFeedNo)) as feedFirstUrl
                    on feedFirstUrl.feedImageFeedNo = ReplyTable.replyFeedNo
where replyFeedNo in (select feedNo from FeedTable where feedHost = 'bllumusic')
order by replyNo DESC;
```

![image](https://user-images.githubusercontent.com/43658658/133075033-016b9c7d-2598-4879-80ba-9f5c130c0206.png)
