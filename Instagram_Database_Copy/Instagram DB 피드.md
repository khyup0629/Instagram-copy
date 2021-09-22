# 피드

![image](https://user-images.githubusercontent.com/43658658/133107762-9391acbd-d96c-4c22-97c7-c7a834bf157b.png)

``` mysql
-- 댓글이 달려있는 피드 번호
select replyFeedNo
from ReplyTable
group by replyFeedNo;

-- 블루뮤직의 피드별 가장 최근에서 첫 번째 좋아요 고유 번호
select clickLikeFeedNo, max(clickLikeNo) as lastClickLikeNo
from (select *
      from ClickLikeTable
      where clickLikeFeedNo in (select feedNo
                                from FeedTable
                                where feedHost = 'bllumusic')) as lastClickLikeTable
group by clickLikeFeedNo;

-- 블루뮤직의 피드별 가장 최근에서 첫 번째 좋아요 고유 번호에 대응되는 유저 프로필 이미지, ID
select clickLikeFeedNo,
       profileImageUrl,
       clickLikeUserID
from ClickLikeTable
         inner join ProfileTable
                    on profileUserID = clickLikeUserID
where clickLikeNo in (select max(clickLikeNo) as lastClickLikeNo
                      from (select *
                            from ClickLikeTable
                            where clickLikeFeedNo in (select feedNo
                                                      from FeedTable
                                                      where feedHost = 'bllumusic')) as lastClickLikeTable
                      group by clickLikeFeedNo)
order by clickLikeNo DESC;

-- 블루뮤직의 피드별 가장 최근 좋아요 고유 번호를 제외한 좋아요 리스트
select *
from ClickLikeTable
where clickLikeNo not in (select max(clickLikeNo) as lastClickLikeNo
                          from (select *
                                from ClickLikeTable
                                where clickLikeFeedNo in (select feedNo
                                                          from FeedTable
                                                          where feedHost = 'bllumusic')) as lastClickLikeTable
                          group by clickLikeFeedNo)
  and clickLikeFeedNo in (select feedNo from FeedTable where feedHost = 'bllumusic');

-- 블루뮤직의 피드별 가장 최근에서 두 번째 좋아요 고유 번호
select clickLikeFeedNo, max(clickLikeNo) as lastClickLikeNo
from (select *
      from ClickLikeTable
      where clickLikeNo not in (select max(clickLikeNo) as lastClickLikeNo
                                from (select *
                                      from ClickLikeTable
                                      where clickLikeFeedNo in (select feedNo
                                                                from FeedTable
                                                                where feedHost = 'bllumusic')) as lastClickLikeTable
                                group by clickLikeFeedNo)
        and clickLikeFeedNo in (select feedNo from FeedTable where feedHost = 'bllumusic')) as lastClickLikeTable
group by clickLikeFeedNo;

-- 블루뮤직 피드별 가장 최근에서 두 번째 좋아요 번호에 대응되는 유저 프로필 이미지, ID
select clickLikeFeedNo,
       profileImageUrl,
       clickLikeUserID
from ClickLikeTable
         inner join ProfileTable
                    on profileUserID = clickLikeUserID
where clickLikeNo in (select max(clickLikeNo) as lastClickLikeNo
                      from (select *
                            from ClickLikeTable
                            where clickLikeNo not in (select max(clickLikeNo) as lastClickLikeNo
                                                      from (select *
                                                            from ClickLikeTable
                                                            where clickLikeFeedNo in (select feedNo
                                                                                      from FeedTable
                                                                                      where feedHost = 'bllumusic')) as lastClickLikeTable
                                                      group by clickLikeFeedNo)
                              and clickLikeFeedNo in
                                  (select feedNo from FeedTable where feedHost = 'bllumusic')) as lastClickLikeTable
                      group by clickLikeFeedNo)
order by clickLikeNo DESC;

-- 블루뮤직 피드별 가장 최근에서 두 번째 좋아요 번호보다 작은 좋아요 번호
select *
from ClickLikeTable
where clickLikeNo not in (select max(clickLikeNo) as lastClickLikeNo
                          from (select *
                                from ClickLikeTable
                                where clickLikeFeedNo in (select feedNo
                                                          from FeedTable
                                                          where feedHost = 'bllumusic')) as lastClickLikeTable
                          group by clickLikeFeedNo)
  and clickLikeNo not in (select max(clickLikeNo) as lastClickLikeNo
                          from (select *
                                from ClickLikeTable
                                where clickLikeNo not in (select max(clickLikeNo) as lastClickLikeNo
                                                          from (select *
                                                                from ClickLikeTable
                                                                where clickLikeFeedNo in (select feedNo
                                                                                          from FeedTable
                                                                                          where feedHost = 'bllumusic')) as lastClickLikeTable
                                                          group by clickLikeFeedNo)
                                  and clickLikeFeedNo in
                                      (select feedNo from FeedTable where feedHost = 'bllumusic')) as lastClickLikeTable
                          group by clickLikeFeedNo)
  and clickLikeFeedNo in (select feedNo from FeedTable where feedHost = 'bllumusic');

-- 블루뮤직 피드별 가장 최근에서 세 번째 좋아요 번호
select clickLikeFeedNo, max(clickLikeNo)
from (select *
      from ClickLikeTable
      where clickLikeNo not in (select max(clickLikeNo) as lastClickLikeNo
                                from (select *
                                      from ClickLikeTable
                                      where clickLikeFeedNo in (select feedNo
                                                                from FeedTable
                                                                where feedHost = 'bllumusic')) as lastClickLikeTable
                                group by clickLikeFeedNo)
        and clickLikeNo not in (select max(clickLikeNo) as lastClickLikeNo
                                from (select *
                                      from ClickLikeTable
                                      where clickLikeNo not in (select max(clickLikeNo) as lastClickLikeNo
                                                                from (select *
                                                                      from ClickLikeTable
                                                                      where clickLikeFeedNo in (select feedNo
                                                                                                from FeedTable
                                                                                                where feedHost = 'bllumusic')) as lastClickLikeTable
                                                                group by clickLikeFeedNo)
                                        and clickLikeFeedNo in
                                            (select feedNo from FeedTable where feedHost = 'bllumusic')) as lastClickLikeTable
                                group by clickLikeFeedNo)
        and clickLikeFeedNo in (select feedNo from FeedTable where feedHost = 'bllumusic')) as lastClickLikeTable
group by clickLikeFeedNo;

-- 블루뮤직 피드별 가장 최근에서 세 번째 좋아요 번호에 대응되는 유저 프로필 이미지, ID
select clickLikeFeedNo,
       profileImageUrl,
       clickLikeUserID
from ClickLikeTable
         inner join ProfileTable
                    on profileUserID = clickLikeUserID
where clickLikeNo in (select max(clickLikeNo)
                      from (select *
                            from ClickLikeTable
                            where clickLikeNo not in (select max(clickLikeNo) as lastClickLikeNo
                                                      from (select *
                                                            from ClickLikeTable
                                                            where clickLikeFeedNo in (select feedNo
                                                                                      from FeedTable
                                                                                      where feedHost = 'bllumusic')) as lastClickLikeTable
                                                      group by clickLikeFeedNo)
                              and clickLikeNo not in (select max(clickLikeNo) as lastClickLikeNo
                                                      from (select *
                                                            from ClickLikeTable
                                                            where clickLikeNo not in
                                                                  (select max(clickLikeNo) as lastClickLikeNo
                                                                   from (select *
                                                                         from ClickLikeTable
                                                                         where clickLikeFeedNo in (select feedNo
                                                                                                   from FeedTable
                                                                                                   where feedHost = 'bllumusic')) as lastClickLikeTable
                                                                   group by clickLikeFeedNo)
                                                              and clickLikeFeedNo in
                                                                  (select feedNo from FeedTable where feedHost = 'bllumusic')) as lastClickLikeTable
                                                      group by clickLikeFeedNo)
                              and clickLikeFeedNo in
                                  (select feedNo from FeedTable where feedHost = 'bllumusic')) as lastClickLikeTable
                      group by clickLikeFeedNo)
order by clickLikeNo DESC;

-- 블루뮤직의 피드별 가장 최근에서 첫 번째 댓글 고유 번호
select replyFeedNo, max(replyNo) as lastReplyNo
from (select *
      from ReplyTable
      where replyFeedNo in (select feedNo
                            from FeedTable
                            where feedHost = 'bllumusic')) as lastReplyTable
group by replyFeedNo;

-- 블루뮤직의 피드별 가장 최근에서 첫 번째 댓글 고유 번호에 대응되는 유저 프로필 이미지, ID
select replyFeedNo,
       profileImageUrl,
       replyUserID,
       replyContent,
       replyIsHeart
from ReplyTable
         inner join ProfileTable
                    on profileUserID = replyUserID
where replyNo in (select max(replyNo) as lastReplyNo
                  from (select *
                        from ReplyTable
                        where replyFeedNo in (select feedNo
                                              from FeedTable
                                              where feedHost = 'bllumusic')) as lastReplyTable
                  group by replyFeedNo)
order by replyNo DESC;

-- 블루뮤직 피드별 이미지 + 동영상 번호
select feedImageFeedNo, group_concat(feedImageUrl, '')
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
        and feedvideoUserID = 'bllumusic') as allContentUrl
group by feedImageFeedNo
order by feedImageFeedNo DESC;

-- 블루뮤직의 피드
select profileImageUrl                                                  as feedProfileImage,
       feedHost,
       groupContentUrl,
       if(feedNo in (select clickLikeFeedNo from ClickLikeTable group by clickLikeFeedNo),
          if((select count(clickLikeNo) as clickLikeCnt
              from ClickLikeTable
              where clickLikeFeedNo = feedNo
              group by clickLikeFeedNo) >= 3,
             concat((select profileImageUrl
                     from (select clickLikeFeedNo,
                                  profileImageUrl,
                                  clickLikeUserID
                           from ClickLikeTable
                                    inner join ProfileTable
                                               on profileUserID = clickLikeUserID
                           where clickLikeNo in (select max(clickLikeNo) as lastClickLikeNo
                                                 from (select *
                                                       from ClickLikeTable
                                                       where clickLikeFeedNo in (select feedNo
                                                                                 from FeedTable
                                                                                 where feedHost = 'bllumusic')) as lastClickLikeTable
                                                 group by clickLikeFeedNo)) as lastClickLikeTable
                     where clickLikeFeedNo = feedNo), ', ', (select profileImageUrl
                                                             from (select clickLikeFeedNo,
                                                                          profileImageUrl,
                                                                          clickLikeUserID
                                                                   from ClickLikeTable
                                                                            inner join ProfileTable
                                                                                       on profileUserID = clickLikeUserID
                                                                   where clickLikeNo in
                                                                         (select max(clickLikeNo) as lastClickLikeNo
                                                                          from (select *
                                                                                from ClickLikeTable
                                                                                where clickLikeNo not in
                                                                                      (select max(clickLikeNo) as lastClickLikeNo
                                                                                       from (select *
                                                                                             from ClickLikeTable
                                                                                             where clickLikeFeedNo in
                                                                                                   (select feedNo
                                                                                                    from FeedTable
                                                                                                    where feedHost = 'bllumusic')) as lastClickLikeTable
                                                                                       group by clickLikeFeedNo)
                                                                                  and clickLikeFeedNo in
                                                                                      (select feedNo from FeedTable where feedHost = 'bllumusic')) as lastClickLikeTable
                                                                          group by clickLikeFeedNo)) as lastSecontClickLikeTable
                                                             where clickLikeFeedNo = feedNo), ', ',
                    (select profileImageUrl
                     from (select clickLikeFeedNo,
                                  profileImageUrl,
                                  clickLikeUserID
                           from ClickLikeTable
                                    inner join ProfileTable
                                               on profileUserID = clickLikeUserID
                           where clickLikeNo in (select max(clickLikeNo)
                                                 from (select *
                                                       from ClickLikeTable
                                                       where clickLikeNo not in
                                                             (select max(clickLikeNo) as lastClickLikeNo
                                                              from (select *
                                                                    from ClickLikeTable
                                                                    where clickLikeFeedNo in (select feedNo
                                                                                              from FeedTable
                                                                                              where feedHost = 'bllumusic')) as lastClickLikeTable
                                                              group by clickLikeFeedNo)
                                                         and clickLikeNo not in
                                                             (select max(clickLikeNo) as lastClickLikeNo
                                                              from (select *
                                                                    from ClickLikeTable
                                                                    where clickLikeNo not in
                                                                          (select max(clickLikeNo) as lastClickLikeNo
                                                                           from (select *
                                                                                 from ClickLikeTable
                                                                                 where clickLikeFeedNo in (select feedNo
                                                                                                           from FeedTable
                                                                                                           where feedHost = 'bllumusic')) as lastClickLikeTable
                                                                           group by clickLikeFeedNo)
                                                                      and clickLikeFeedNo in
                                                                          (select feedNo from FeedTable where feedHost = 'bllumusic')) as lastClickLikeTable
                                                              group by clickLikeFeedNo)
                                                         and clickLikeFeedNo in
                                                             (select feedNo from FeedTable where feedHost = 'bllumusic')) as lastClickLikeTable
                                                 group by clickLikeFeedNo)) as lastThirdClickLikeTable
                     where clickLikeFeedNo = feedNo)), ''), if((select count(clickLikeNo) as clickLikeCnt
                                                                from ClickLikeTable
                                                                where clickLikeFeedNo = feedNo
                                                                group by clickLikeFeedNo) >= 2,
                                                               concat((select profileImageUrl
                                                                       from (select clickLikeFeedNo,
                                                                                    profileImageUrl,
                                                                                    clickLikeUserID
                                                                             from ClickLikeTable
                                                                                      inner join ProfileTable
                                                                                                 on profileUserID = clickLikeUserID
                                                                             where clickLikeNo in
                                                                                   (select max(clickLikeNo) as lastClickLikeNo
                                                                                    from (select *
                                                                                          from ClickLikeTable
                                                                                          where clickLikeFeedNo in
                                                                                                (select feedNo
                                                                                                 from FeedTable
                                                                                                 where feedHost = 'bllumusic')) as lastClickLikeTable
                                                                                    group by clickLikeFeedNo)) as lastClickLikeTable
                                                                       where clickLikeFeedNo = feedNo), ', ',
                                                                      (select profileImageUrl
                                                                       from (select clickLikeFeedNo,
                                                                                    profileImageUrl,
                                                                                    clickLikeUserID
                                                                             from ClickLikeTable
                                                                                      inner join ProfileTable
                                                                                                 on profileUserID = clickLikeUserID
                                                                             where clickLikeNo in
                                                                                   (select max(clickLikeNo) as lastClickLikeNo
                                                                                    from (select *
                                                                                          from ClickLikeTable
                                                                                          where clickLikeNo not in
                                                                                                (select max(clickLikeNo) as lastClickLikeNo
                                                                                                 from (select *
                                                                                                       from ClickLikeTable
                                                                                                       where clickLikeFeedNo in
                                                                                                             (select feedNo
                                                                                                              from FeedTable
                                                                                                              where feedHost = 'bllumusic')) as lastClickLikeTable
                                                                                                 group by clickLikeFeedNo)
                                                                                            and clickLikeFeedNo in
                                                                                                (select feedNo from FeedTable where feedHost = 'bllumusic')) as lastClickLikeTable
                                                                                    group by clickLikeFeedNo)) as lastSecontClickLikeTable
                                                                       where clickLikeFeedNo = feedNo)),
                                                               if((select count(clickLikeNo) as clickLikeCnt
                                                                   from ClickLikeTable
                                                                   where clickLikeFeedNo = feedNo
                                                                   group by clickLikeFeedNo) >= 1,
                                                                  concat((select profileImageUrl
                                                                          from (select clickLikeFeedNo,
                                                                                       profileImageUrl,
                                                                                       clickLikeUserID
                                                                                from ClickLikeTable
                                                                                         inner join ProfileTable
                                                                                                    on profileUserID = clickLikeUserID
                                                                                where clickLikeNo in
                                                                                      (select max(clickLikeNo) as lastClickLikeNo
                                                                                       from (select *
                                                                                             from ClickLikeTable
                                                                                             where clickLikeFeedNo in
                                                                                                   (select feedNo
                                                                                                    from FeedTable
                                                                                                    where feedHost = 'bllumusic')) as lastClickLikeTable
                                                                                       group by clickLikeFeedNo)) as lastClickLikeTable
                                                                          where clickLikeFeedNo = feedNo)),
                                                                  ''))) as lastProfileImage,
       if(feedNo in (select clickLikeFeedNo from ClickLikeTable group by clickLikeFeedNo),
          if((select count(clickLikeNo) as clickLikeCnt
              from ClickLikeTable
              where clickLikeFeedNo = feedNo
              group by clickLikeFeedNo) >= 2,
             concat((select clickLikeUserID
                     from ClickLikeTable
                     where clickLikeNo =
                           (select max(clickLikeNo)
                            from ClickLikeTable
                            where clickLikeFeedNo = FeedTable.feedNo
                            group by clickLikeFeedNo)),
                    '외 ', (select count(clickLikeNo) as clickLikeCnt
                           from ClickLikeTable
                           where clickLikeFeedNo = feedNo
                           group by clickLikeFeedNo) - 1, '명이 좋아합니다.'),
             concat((select clickLikeUserID as clickLikeCnt
                     from ClickLikeTable
                     where clickLikeFeedNo = feedNo), '님이 좋아합니다.')), '')
                                                                        as clickLikeCnt,
       feedContent,
       if(feedNo in (select replyFeedNo from ReplyTable group by replyFeedNo),
          if((select count(replyFeedNo) as replyCnt
              from ReplyTable
              where replyFeedNo = feedNo
              group by replyFeedNo) >= 1,
             concat('댓글 ', (select count(replyFeedNo) as replyCnt
                            from ReplyTable
                            where replyFeedNo = feedNo
                            group by replyFeedNo), '개 모두 보기'), ''),
          '')                                                           as replyCnt,
       if(feedNo in (select replyFeedNo from ReplyTable group by replyFeedNo),
          if((select count(replyFeedNo) as replyCnt
              from ReplyTable
              where replyFeedNo = feedNo
              group by replyFeedNo) >= 1,
             concat((select replyContent
                     from (select replyFeedNo,
                                  profileImageUrl,
                                  replyUserID,
                                  replyContent
                           from ReplyTable
                                    inner join ProfileTable
                                               on profileUserID = replyUserID
                           where replyNo in
                                 (select max(replyNo) as lastReplyNo
                                  from (select *
                                        from ReplyTable
                                        where replyFeedNo in (select feedNo
                                                              from FeedTable
                                                              where feedHost = 'bllumusic')) as lastReplyTable
                                  group by replyFeedNo)) as lastReplyTable
                     where replyFeedNo = feedNo)), ''),
          '')                                                           as replyContent,
       if(feedNo in (select replyFeedNo from ReplyTable group by replyFeedNo),
          if((select count(replyFeedNo) as replyCnt
              from ReplyTable
              where replyFeedNo = feedNo
              group by replyFeedNo) >= 1,
             concat((select replyIsHeart
                     from (select replyFeedNo,
                                  profileImageUrl,
                                  replyUserID,
                                  replyIsHeart
                           from ReplyTable
                                    inner join ProfileTable
                                               on profileUserID = replyUserID
                           where replyNo in
                                 (select max(replyNo) as lastReplyNo
                                  from (select *
                                        from ReplyTable
                                        where replyFeedNo in (select feedNo
                                                              from FeedTable
                                                              where feedHost = 'bllumusic')) as lastReplyTable
                                  group by replyFeedNo)) as lastReplyTable
                     where replyFeedNo = feedNo)), ''),
          '')                                                           as replyIsHeart,
       case
           when timestampdiff(second, feedCreatedAt, '2021-09-11 00:00:00') <= 59
               then '방금 전'
           when timestampdiff(minute, feedCreatedAt, '2021-09-11 00:00:00') <= 59
               then concat(timestampdiff(minute, feedCreatedAt, '2021-09-11 00:00:00'), '분 전')
           when timestampdiff(hour, feedCreatedAt, '2021-09-11 00:00:00') <= 23
               then concat(timestampdiff(hour, feedCreatedAt, '2021-09-11 00:00:00'), '시간 전')
           else date_format(feedCreatedAt, '%m월 %d일')
           end                                                          as feedCreatedAt
from FeedTable
         inner join ProfileTable
                    on profileUserID = feedHost
         inner join (select feedImageFeedNo, group_concat(feedImageUrl, '') as groupContentUrl
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
                             and feedvideoUserID = 'bllumusic') as allContentUrl
                     group by feedImageFeedNo) as allContentUrl
                    on allContentUrl.feedImageFeedNo = feedNo
where feedHost = 'bllumusic'
order by feedNo DESC;
```

![image](https://user-images.githubusercontent.com/43658658/133111356-221da65e-2746-43ef-811d-e61634b50fd0.png)

![image](https://user-images.githubusercontent.com/43658658/133111507-fcb8f280-42bc-425e-b4cf-fc39766f7390.png)
