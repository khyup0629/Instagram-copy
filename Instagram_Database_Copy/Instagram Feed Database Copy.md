# Instagram Feed Database Copy

## ERD 설계

![image](https://user-images.githubusercontent.com/43658658/132510879-06e0f7e9-c373-4580-9941-f3c2b9f5d135.png)

테이블은 크게 5개로 생각해 볼 수 있습니다.

1. User : 인스타의 유저에 관한 정보가 담기는 테이블입니다.
2. Feed : 피드에 들어갈 컨텐츠 정보가 담기는 테이블입니다.
3. clickLike : '좋아요'를 누른 회원의 정보가 담기는 테이블입니다.
4. Reply : 댓글과 관련한 정보가 담기는 테이블입니다.
5. Rereply : 대댓글과 관련한 정보가 담기는 테이블입니다.

![image](https://user-images.githubusercontent.com/43658658/132552339-2b59f9fb-8550-49b8-b9b1-9f20b64563e3.png)

## Datagrip에 데이터 작성

한글 입/출력을 위해서 테이블을 콘솔로 옮겨서 실행하기 전에 반드시 끝에 `default charset=utf8`을 추가해 주어야합니다.   
![image](https://user-images.githubusercontent.com/43658658/132625663-b7cd574c-c3b2-4526-a7a4-5142a869cca5.png)

1. User   
![image](https://user-images.githubusercontent.com/43658658/132625372-95713037-80a7-4d40-bf5d-33bad646523b.png)

2. Feed   
![image](https://user-images.githubusercontent.com/43658658/132625417-751acd8c-0494-47b1-a152-ce87710dad78.png)

3. clickLike   
![image](https://user-images.githubusercontent.com/43658658/132625456-5b90e2de-5486-43f8-b12d-be13d40bf093.png)

4. Reply   
![image](https://user-images.githubusercontent.com/43658658/132625477-10623c75-cf17-4629-9935-b0d7493a5883.png)

5. Rereply   
![image](https://user-images.githubusercontent.com/43658658/132625502-14ee11e8-c688-4abe-93a0-ab3da25520e9.png)

## 한방 쿼리 작성

타임라인에 있는 많은 피드들 중 하나의 피드를 클릭했을 때 그 피드에 대한 정보를 출력하는 SQL문을 작성하겠습니다.   
(Tip : 작은 문제로 쪼개서 생각하고, 점진적으로 `한방 쿼리`로 합쳐나가면 차근차근 해나갈 수 있습니다)

- 꼭 나와야 하는 칼럼   
UserID, imageUrl, totalLikeUser(lastLikeUser, clickLikeCnt), content, ReplyCnt, LastReply(user, content, isheart), LastRereply(user, content, isheart), createdAtFeed

``` mysql
-- 피드 유저, 피드 이미지, 피드 내용, 피드 올린 시각
select userID, imageUrl as feedImage, content,
    case
        when timestampdiff(second, createdAt, current_timestamp) <= 59
        then '방금 전'
        when timestampdiff(minute, createdAt, current_timestamp) <= 59
        then concat(timestampdiff(minute, createdAt, current_timestamp),'분 전')
        when timestampdiff(hour, createdAt, current_timestamp) <= 23
        then concat(timestampdiff(hour, createdAt, current_timestamp), '시간 전')
        else date_format(createdAt, '%m월 %d일')
    end as createdAtFeed
from Feed
where userID='bllumusic';
```

![image](https://user-images.githubusercontent.com/43658658/132536182-76989683-53cf-418d-a12f-7c6181846dbc.png)

``` mysql
-- feed별 좋아요 개수
select feedNo, count(feedNo) as clickLikeCnt
from clickLike group by feedNo;
```

![image](https://user-images.githubusercontent.com/43658658/132538353-2d8e02ab-a397-40bb-9e93-b7f0957df4ab.png)

``` mysql
-- 좋아요 개수가 추가된 버전
select Feed.userID, imageUrl as feedImage, clickLikeCnt, content,
    case
        when timestampdiff(second, Feed.createdAt, current_timestamp) <= 59
        then '방금 전'
        when timestampdiff(minute, Feed.createdAt, current_timestamp) <= 59
        then concat(timestampdiff(minute, Feed.createdAt, current_timestamp),'분 전')
        when timestampdiff(hour, Feed.createdAt, current_timestamp) <= 23
        then concat(timestampdiff(hour, Feed.createdAt, current_timestamp), '시간 전')
        else date_format(Feed.createdAt, '%m월 %d일')
    end as createdAtFeed
from Feed
inner join (select feedNo, count(feedNo) as clickLikeCnt
from clickLike group by feedNo) as clickLike
on clickLike.feedNo = Feed.feedNo
where Feed.userID='bllumusic';
```

![image](https://user-images.githubusercontent.com/43658658/132538723-d8b3c728-585c-4c43-934b-477dce5c218e.png)

``` mysql
-- 각 피드별로 좋아요를 가장 마지막에 한 사람
select feedNo, userID, createdAt
from clickLike
where (feedNo, createdAt) in (select feedNo, max(createdAt) from clickLike group by feedNo);
```

![image](https://user-images.githubusercontent.com/43658658/132549654-224eb4c6-de77-4e4f-bd63-63843ba9605f.png)

``` mysql
-- 마지막 좋아요 한 사람 + 좋아요 개수
select clickLike.feedNo, concat(userID, '님 외 ', clickLikeCnt, '명이 좋아합니다.') as totalLikeUser
from clickLike
inner join (select feedNo, count(feedNo) as clickLikeCnt
from clickLike group by feedNo) as clickLikeCntTable
on clickLike.feedNo = clickLikeCntTable.feedNo
where (clickLike.feedNo, createdAt) in (select feedNo, max(createdAt) from clickLike group by feedNo);
```

![image](https://user-images.githubusercontent.com/43658658/132551262-55570ff8-d0ef-47d2-a58f-6b2d4f16c0db.png)

``` mysql
-- 마지막 좋아요 한 사람 + 좋아요 개수가 추가된 버전
select Feed.userID, imageUrl as feedImage, totalLikeUser, content,
    case
        when timestampdiff(second, Feed.createdAt, current_timestamp) <= 59
        then '방금 전'
        when timestampdiff(minute, Feed.createdAt, current_timestamp) <= 59
        then concat(timestampdiff(minute, Feed.createdAt, current_timestamp),'분 전')
        when timestampdiff(hour, Feed.createdAt, current_timestamp) <= 23
        then concat(timestampdiff(hour, Feed.createdAt, current_timestamp), '시간 전')
        else date_format(Feed.createdAt, '%m월 %d일')
    end as createdAtFeed
from Feed
inner join (select clickLike.feedNo, concat(userID, '님 외 ', clickLikeCnt, '명이 좋아합니다.') as totalLikeUser
from clickLike
inner join (select feedNo, count(feedNo) as clickLikeCnt
from clickLike group by feedNo) as clickLikeCntTable
on clickLike.feedNo = clickLikeCntTable.feedNo
where (clickLike.feedNo, createdAt) in (select feedNo, max(createdAt) from clickLike group by feedNo)) as clickLike
on clickLike.feedNo = Feed.feedNo
where Feed.userID='bllumusic';
```

![image](https://user-images.githubusercontent.com/43658658/132551816-abdd09a9-a5fa-4701-966f-36ac01840276.png)

``` mysql
-- 피드별 댓글 개수
select Reply.feedNo, concat('댓글 ', count(Reply.feedNo), '개 모두 보기') as ReplyCnt
from Reply group by Reply.feedNo;
```

![image](https://user-images.githubusercontent.com/43658658/132625087-75424698-bafb-4dee-abf4-3cc030b2d005.png)

``` mysql
-- 가장 최근 댓글
select Reply.replyNo, Reply.feedNo, Reply.userID as lastReplyUser, Reply.reply as lastReply,
       Reply.isHeart as replyIsHeart
from Reply
where (feedNo, createdAt) in (select feedNo, max(createdAt) from Reply group by feedNo);
```

![image](https://user-images.githubusercontent.com/43658658/132625121-f2241acb-b01a-40d5-b2a0-17582eb5e71f.png)

``` mysql
-- 댓글별 가장 최근 대댓글
select Rereply.replyNo, Rereply.feedNo, Rereply.userID, Rereply.reply
from Rereply
where (replyNo, no) in (select replyNo, max(no) from Rereply group by replyNo);
```

![image](https://user-images.githubusercontent.com/43658658/132625182-8f706484-f278-4c10-9af3-2417b51e4448.png)

``` mysql
-- 피드별 댓글 개수 + 가장 최근 댓글 + 대댓글이 추가된 버전
select Feed.userID, imageUrl as Image, totalLikeUser, content, ReplyCnt, lastReplyUser, lastReply, replyIsHeart,
       lastRereplyUser, lastRereply, rereplyIsHeart,
    case
        when timestampdiff(second, Feed.createdAt, current_timestamp) <= 59
        then '방금 전'
        when timestampdiff(minute, Feed.createdAt, current_timestamp) <= 59
        then concat(timestampdiff(minute, Feed.createdAt, current_timestamp),'분 전')
        when timestampdiff(hour, Feed.createdAt, current_timestamp) <= 23
        then concat(timestampdiff(hour, Feed.createdAt, current_timestamp), '시간 전')
        else date_format(Feed.createdAt, '%m월 %d일')
    end as createdAtFeed
from Feed
inner join (select clickLike.feedNo, concat(userID, '님 외 ', clickLikeCnt, '명이 좋아합니다.') as totalLikeUser
from clickLike
inner join (select feedNo, count(feedNo) as clickLikeCnt
from clickLike group by feedNo) as clickLikeCntTable
on clickLike.feedNo = clickLikeCntTable.feedNo
where (clickLike.feedNo, createdAt) in (select feedNo, max(createdAt) from clickLike group by feedNo)) as clickLike
on clickLike.feedNo = Feed.feedNo
inner join (select Reply.feedNo, concat('댓글 ', count(Reply.feedNo), '개 모두 보기') as ReplyCnt
from Reply group by Reply.feedNo) as totalReply
on totalReply.feedNo = Feed.feedNo
inner join (select Reply.replyNo, Reply.feedNo, Reply.userID as lastReplyUser, Reply.reply as lastReply,
       Reply.isHeart as replyIsHeart
from Reply
where (feedNo, createdAt) in (select feedNo, max(createdAt) from Reply group by feedNo)) as lastReplyTable
on lastReplyTable.feedNo = Feed.feedNo
inner join (select Rereply.replyNo, Rereply.feedNo, Rereply.userID as lastRereplyUser, Rereply.reply as lastRereply,
        Rereply.isHeart as rereplyIsHeart
from Rereply
where (replyNo, no) in (select replyNo, max(no) from Rereply group by replyNo)) as lastRereplyTable
on lastRereplyTable.feedNo = Feed.feedNo
where Feed.userID='bllumusic';
```

![image](https://user-images.githubusercontent.com/43658658/132625262-c8e77333-baed-459f-8587-ccc669c8b6a6.png)
