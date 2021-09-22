# 채팅방 목록 화면

![image](https://user-images.githubusercontent.com/43658658/133213737-06a7aaba-54d4-4684-8b02-2d2727751ae0.png)

``` mysql
-- 채팅방마다 가장 마지막 메세지의 번호
select chatRoomUserName, max(chatNo)
from ChatTable
group by chatRoomUserName;

-- 가장 마지막 메세지 번호와 대응되는 칼럼
-- 지금 활동 중이면 '지금 활동 중'을 가장 최근 메세지로 띄우기
-- 마지막 활동이 1시간 이내라면 '최근 활동 : m분 전'을 가장 최근 메세지로 띄우기
-- 스토리에 공감했으면, '회원님의 스토리에 공감했습니다. n일'을 띄우기
-- 마지막 메세지를 보낸 시간이 하루를 넘었을 때,
--      마지막 말한 사람이 나라면 메세지를 보낸 요일 표시, 마지막 말한 사람이 상대방이라면
--          내가 메세지를 확인했다면 확인한 요일 표시, 확인하지 않았다면 마지막 메세지 그대로 띄우기
-- 이 모든 경우가 아니라면 마지막 메세지 그대로 띄우기
select profileImageUrl,
       if(chatRoomUserName in (select profileName
                               from StoryTable
                                        inner join ProfileTable
                                                   on profileUserID = storyUserID
                               where timestampdiff(day, storyCreatedAt, '2021-09-11 00:00:00') < 1
                                 and 'No' =
                                     if(storyNo in (select storyWatchingStoryNo
                                                    from StoryWatchingTable
                                                    where storyWatchingUserID = 'bllumusic'),
                                        'Yes', 'No')),
          'No',
          'Yes')                               as isStoryWatching,
       chatRoomUserName,
       if(profileActNow = 'Y', '지금 활동 중',
          if(timestampdiff(hour, profileLastActTime, '2021-09-10 06:00:00') < 1,
             concat('최근 활동: ', timestampdiff(minute, profileLastActTime, '2021-09-10 06:00:00'), '분 전'),
             if(chatContent = '회원님의 스토리에 공감했습니다.', concat(chatContent,
                                                          case
                                                              when timestampdiff(second, chatCreatedAt, '2021-09-11 00:00:00') < 60
                                                                  then concat(
                                                                      timestampdiff(second, chatCreatedAt, '2021-09-11 00:00:00'),
                                                                      '초')
                                                              when timestampdiff(minute, chatCreatedAt, '2021-09-11 00:00:00') < 60
                                                                  then concat(
                                                                      timestampdiff(minute, chatCreatedAt, '2021-09-11 00:00:00'),
                                                                      '분')
                                                              when timestampdiff(hour, chatCreatedAt, '2021-09-11 00:00:00') < 24
                                                                  then concat(
                                                                      timestampdiff(hour, chatCreatedAt, '2021-09-11 00:00:00'),
                                                                      '시간')
                                                              when timestampdiff(day, chatCreatedAt, '2021-09-11 00:00:00') < 7
                                                                  then concat(
                                                                      timestampdiff(day, chatCreatedAt, '2021-09-11 00:00:00'),
                                                                      '일')
                                                              else concat(
                                                                      timestampdiff(week, chatCreatedAt, '2021-09-11 00:00:00'),
                                                                      '주')
                                                              end),
                if(chatContent = '메세지를 좋아합니다.', concat('메세지를 좋아합니다. ', case
                                                                           when timestampdiff(second, chatCreatedAt, '2021-09-11 00:00:00') < 60
                                                                               then concat(
                                                                                   timestampdiff(second, chatCreatedAt, '2021-09-11 00:00:00'),
                                                                                   '초')
                                                                           when timestampdiff(minute, chatCreatedAt, '2021-09-11 00:00:00') < 60
                                                                               then concat(
                                                                                   timestampdiff(minute, chatCreatedAt, '2021-09-11 00:00:00'),
                                                                                   '분')
                                                                           when timestampdiff(hour, chatCreatedAt, '2021-09-11 00:00:00') < 24
                                                                               then concat(
                                                                                   timestampdiff(hour, chatCreatedAt, '2021-09-11 00:00:00'),
                                                                                   '시간')
                                                                           when timestampdiff(day, chatCreatedAt, '2021-09-11 00:00:00') < 7
                                                                               then concat(
                                                                                   timestampdiff(day, chatCreatedAt, '2021-09-11 00:00:00'),
                                                                                   '일')
                                                                           else concat(
                                                                                   timestampdiff(week, chatCreatedAt, '2021-09-11 00:00:00'),
                                                                                   '주')
                    end),
                   if(timestampdiff(day, chatCreatedAt, '2021-09-11 00:00:00') > 1,
                      if(chatTalker = '김협', concat(
                              case
                                  when weekday(chatCreatedAt) = 0 then '월'
                                  when weekday(chatCreatedAt) = 1 then '화'
                                  when weekday(chatCreatedAt) = 2 then '수'
                                  when weekday(chatCreatedAt) = 3 then '목'
                                  when weekday(chatCreatedAt) = 4 then '금'
                                  when weekday(chatCreatedAt) = 5 then '토'
                                  when weekday(chatCreatedAt) = 6 then '일' end, '요일에 보냄'),
                         if(chatCheckTime = null, chatContent, concat(
                                 case
                                     when weekday(chatCheckTime) = 0 then '월'
                                     when weekday(chatCheckTime) = 1 then '화'
                                     when weekday(chatCheckTime) = 2 then '수'
                                     when weekday(chatCheckTime) = 3 then '목'
                                     when weekday(chatCheckTime) = 4 then '금'
                                     when weekday(chatCheckTime) = 5 then '토'
                                     when weekday(chatCheckTime) = 6 then '일' end, '요일에 확인됨'))),
                      concat(chatContent)))))) as lastChat

from ChatTable
         inner join ProfileTable
                    on profileName = chatRoomUserName
where chatNo in (select max(chatNo)
                 from ChatTable
                 group by chatRoomUserName);
```

![image](https://user-images.githubusercontent.com/43658658/133213606-d3e7f6a9-3611-4ad9-8cbf-09d0a60a562f.png)
