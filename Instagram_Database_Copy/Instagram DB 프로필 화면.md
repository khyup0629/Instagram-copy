# 프로필 화면

![image](https://user-images.githubusercontent.com/43658658/133081066-108645d1-4917-4ef3-8fe2-c2e168802a03.png)

> <h3>유저 프로필 정보</h3>

![image](https://user-images.githubusercontent.com/43658658/133085757-fd6c2a40-d0c6-400b-9d35-2d15bc4c9a3e.png)

``` mysql
-- 블루뮤직의 피드 게시물 개수
select count(feedNo) as bllumusicFeedCnt from FeedTable where feedHost = 'bllumusic';

-- 블루뮤직의 팔로워 수
select count(followerNo) as bllumusicFollower from FollowerTable where followerHost = 'bllumusic';

-- 블루뮤직이 팔로우하는 유저 수
select count(followerNo) as bllumusicFollowing from FollowerTable where followerUserID = 'bllumusic';

-- 블루뮤직의 정보
select profileUserID,
       profileImageUrl, (select count(feedNo) as bllumusicFeedCnt from FeedTable where feedHost = 'bllumusic') as profileFeedCnt,
       (select count(followerNo) as bllumusicFollower from FollowerTable where followerHost = 'bllumusic') as profileFollowerCnt,
       (select count(followerNo) as bllumusicFollowing from FollowerTable where followerUserID = 'bllumusic') as profileFollowingCnt,
       profileName, profileGenre, profileContent, profileLink
from ProfileTable
where profileUserID = 'bllumusic';
```

![image](https://user-images.githubusercontent.com/43658658/133081176-9eb61840-9f31-40b0-a1a2-fd48dbe70275.png)

> <h3>하이라이트</h3>

![image](https://user-images.githubusercontent.com/43658658/133085790-bd4b898c-73b0-4fdf-b8b0-1be0ba7ce51b.png)

``` mysql
-- 블루뮤직의 하이라이트
select hlCoverImageUrl as highlightCover, hlTitle as highlightTitle
from HighlightTable
where hlHost = 'bllumusic';
```

![image](https://user-images.githubusercontent.com/43658658/133081505-4fd9e4c4-3c3b-414d-9e84-dc863efdece1.png)

> <h3>유저 피드(전체)</h3>

![image](https://user-images.githubusercontent.com/43658658/133085846-aeb93bca-d253-42ef-a4e1-bbee8d399df2.png)

``` mysql
-- 블루뮤직의 전체 피드, 컨텐츠 이미지 or 동영상 구분
select *, 'Image' as Type
from FeedImageTable
where feedimageno in
      (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
  and feedImageUserID = 'bllumusic'
union all
select *, 'Video' as Type
from FeedVideoTable
where feedVideoNo in
      (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
  and feedvideoUserID = 'bllumusic';

-- 블루뮤직의 피드별 첫 번째 이미지 or 동영상
select feedImageFeedNo, feedImageUrl as firstUrlPerFeed, Type
from (select *, 'Image' as Type
      from FeedImageTable
      where feedimageno in
            (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
        and feedImageUserID = 'bllumusic'
      union all
      select *, 'Video' as Type
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

-- 블루뮤직의 피드별 컨텐츠 수
select feedImageFeedNo, count(feedImageNo)
from (select *, 'Image' as Type
      from FeedImageTable
      where feedimageno in
            (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
        and feedImageUserID = 'bllumusic'
      union all
      select *, 'Video' as Type
      from FeedVideoTable
      where feedVideoNo in
            (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
        and feedvideoUserID = 'bllumusic') as bllumusicAllContentCnt
group by feedImageFeedNo;

-- 블루뮤직의 피드 전체 나열, 'Type'은 한 피드에 컨텐츠가 2개 이상일 경우('Multi')와 이미지('Image'), 동영상('Video')로 나눈다.
select contentUrlPerFeed.feedImageFeedNo,
       feedImageUrl                              as firstUrlPerFeed,
       if(contentCntPerFeed >= 2, 'Multi', Type) as Type
from (select *, 'Image' as Type
      from FeedImageTable
      where feedimageno in
            (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
        and feedImageUserID = 'bllumusic'
      union all
      select *, 'Video' as Type
      from FeedVideoTable
      where feedVideoNo in
            (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
        and feedvideoUserID = 'bllumusic') as contentUrlPerFeed
         inner join (select feedImageFeedNo, count(feedImageNo) as contentCntPerFeed
                     from (select *, 'Image' as Type
                           from FeedImageTable
                           where feedimageno in
                                 (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
                             and feedImageUserID = 'bllumusic'
                           union all
                           select *, 'Video' as Type
                           from FeedVideoTable
                           where feedVideoNo in
                                 (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
                             and feedvideoUserID = 'bllumusic') as bllumusicAllContentCnt
                     group by feedImageFeedNo) as bllumusicFeedCnt
                    on bllumusicFeedCnt.feedImageFeedNo = contentUrlPerFeed.feedImageFeedNo
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
                      group by feedImageFeedNo)
order by contentUrlPerFeed.feedImageFeedNo desc;
```

![image](https://user-images.githubusercontent.com/43658658/133085369-efb5c102-0786-4739-9b84-c137bf540a57.png)

> <h3>유저 피드(동영상)</h3>

``` mysql
-- 블루 뮤직의 전체 피드 중 동영상 컨텐츠로만 이루어진 피드만 나열
select *
from (select contentUrlPerFeed.feedImageFeedNo,
             feedImageUrl                              as firstUrlPerFeed,
             if(contentCntPerFeed >= 2, 'Multi', Type) as Type
      from (select *, 'Image' as Type
            from FeedImageTable
            where feedimageno in
                  (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
              and feedImageUserID = 'bllumusic'
            union all
            select *, 'Video' as Type
            from FeedVideoTable
            where feedVideoNo in
                  (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
              and feedvideoUserID = 'bllumusic') as contentUrlPerFeed
               inner join (select feedImageFeedNo, count(feedImageNo) as contentCntPerFeed
                           from (select *, 'Image' as Type
                                 from FeedImageTable
                                 where feedimageno in
                                       (select min(feedImageNo) from FeedImageTable group by feedImageFeedNo)
                                   and feedImageUserID = 'bllumusic'
                                 union all
                                 select *, 'Video' as Type
                                 from FeedVideoTable
                                 where feedVideoNo in
                                       (select min(feedVideoNo) from FeedVideoTable group by feedvideoFeedNo)
                                   and feedvideoUserID = 'bllumusic') as bllumusicAllContentCnt
                           group by feedImageFeedNo) as bllumusicFeedCnt
                          on bllumusicFeedCnt.feedImageFeedNo = contentUrlPerFeed.feedImageFeedNo
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
                            group by feedImageFeedNo)) as grip
where Type = 'Video';
```

![image](https://user-images.githubusercontent.com/43658658/133085514-3baaf388-0489-43e2-8bc0-c496269cc174.png)
