# 팔로워 리스트

![image](https://user-images.githubusercontent.com/43658658/133086306-de49012e-b58a-45ac-b5d9-e2cd203a7386.png)

> <h3>팔로워 수</h3>

![image](https://user-images.githubusercontent.com/43658658/133091465-5b1eaf5d-f833-4ba3-a388-4a7e15f77e49.png)

``` mysql
-- 블루뮤직의 팔로워 수
select count(followerNo) as bllumusicFollower
from FollowerTable
where followerHost = 'bllumusic';
```

> <h3>팔로워 리스트</h3>

![image](https://user-images.githubusercontent.com/43658658/133091507-dab50d03-8131-46e8-93f7-401b770f3884.png)

``` mysql
-- 블루뮤직과 맞팔하지 않은 유저 목록
select profileImageUrl as followerUserProfileImage,
       if(followerUserID in (select storyUserID
                             from StoryTable
                             where timestampdiff(day, storyCreatedAt, '2021-09-11 00:00:00') < 1
                               and 'No' =
                                   if(storyNo in (select storyWatchingStoryNo
                                                  from StoryWatchingTable
                                                  where storyWatchingUserID = 'bllumusic'),
                                      'Yes', 'No')),
          'No',
          'Yes')       as isStoryWatching,
       followerUserID,
       profileName     as followerName,
       '팔로우'           as followerIsFollowing
from (select * from FollowerTable where followerHost = 'bllumusic') as followerBllumusic
         inner join ProfileTable
                    on profileUserID = followerUserID
where followerUserID not in (select followerHost from FollowerTable where followerUserID = 'bllumusic');

-- 블루뮤직과 맞팔한 유저 목록
select profileImageUrl as followerUserProfileImage,
       if(followerUserID in (select storyUserID
                             from StoryTable
                             where timestampdiff(day, storyCreatedAt, '2021-09-11 00:00:00') < 1
                               and 'No' =
                                   if(storyNo in (select storyWatchingStoryNo
                                                  from StoryWatchingTable
                                                  where storyWatchingUserID = 'bllumusic'),
                                      'Yes', 'No')),
          'No',
          'Yes')       as isStoryWatching,
       followerUserID,
       profileName     as followerName,
       ''              as followerIsFollowing
from (select followerUserID from FollowerTable where followerHost = 'bllumusic') as followerBllumusic
         inner join ProfileTable
                    on profileUserID = followerUserID
where followerUserID in (select followerHost from FollowerTable where followerUserID = 'bllumusic');

-- 블루뮤직의 전체 팔로워 목록(맞팔하지 않은 유저가 상위에 위치)
select profileImageUrl as followerUserProfileImage,
       if(followerUserID in (select storyUserID
                             from StoryTable
                             where timestampdiff(day, storyCreatedAt, '2021-09-11 00:00:00') < 1
                               and 'No' =
                                   if(storyNo in (select storyWatchingStoryNo
                                                  from StoryWatchingTable
                                                  where storyWatchingUserID = 'bllumusic'),
                                      'Yes', 'No')),
          'No',
          'Yes')       as isStoryWatching,
       followerUserID,
       profileName     as followerName,
       '팔로우'           as followerIsFollowing
from (select * from FollowerTable where followerHost = 'bllumusic') as followerBllumusic
         inner join ProfileTable
                    on profileUserID = followerUserID
where followerUserID not in (select followerHost from FollowerTable where followerUserID = 'bllumusic')
union
select profileImageUrl as followerUserProfileImage,
       if(followerUserID in (select storyUserID
                             from StoryTable
                             where timestampdiff(day, storyCreatedAt, '2021-09-11 00:00:00') < 1
                               and 'No' =
                                   if(storyNo in (select storyWatchingStoryNo
                                                  from StoryWatchingTable
                                                  where storyWatchingUserID = 'bllumusic'),
                                      'Yes', 'No')),
          'No',
          'Yes')       as isStoryWatching,
       followerUserID,
       profileName     as followerName,
       ''              as followerIsFollowing
from (select followerUserID from FollowerTable where followerHost = 'bllumusic') as followerBllumusic
         inner join ProfileTable
                    on profileUserID = followerUserID
where followerUserID in (select followerHost from FollowerTable where followerUserID = 'bllumusic');
```

![image](https://user-images.githubusercontent.com/43658658/133090413-d10f93c0-9b0c-40ac-bceb-36af7a2fe9b4.png)

# 팔로잉 리스트

![image](https://user-images.githubusercontent.com/43658658/133091588-d734077c-98d5-4bab-b09e-17b140196ab9.png)

> <h3>팔로잉 수</h3>

![image](https://user-images.githubusercontent.com/43658658/133091680-8377f7e1-1aaa-4cc3-abb4-ebbcbc48d8a5.png)

``` mysql
-- 블루뮤직이 팔로우하는 유저 수
select count(followerNo) as bllumusicFollowing
from FollowerTable
where followerUserID = 'bllumusic';
```

![image](https://user-images.githubusercontent.com/43658658/133087592-8e8c8dc7-2668-429b-8a21-5a68d855c35a.png)

> <h3>팔로잉 리스트</h3>

![image](https://user-images.githubusercontent.com/43658658/133091720-d4de9f40-658c-4f1f-ad28-3d2647cb882d.png)

``` mysql
-- 블루뮤직의 전체 팔로잉 유저 목록
select profileImageUrl as followingUserProfileImage,
       if(followerHost in (select storyUserID
                           from StoryTable
                           where timestampdiff(day, storyCreatedAt, '2021-09-11 00:00:00') < 1
                             and 'No' =
                                 if(storyNo in (select storyWatchingStoryNo
                                                from StoryWatchingTable
                                                where storyWatchingUserID = 'bllumusic'),
                                    'Yes', 'No')),
          'No',
          'Yes')       as isStoryWatching,
       followerHost    as followingUserID,
       profileName     as followingName
from FollowerTable
         inner join ProfileTable
                    on profileUserID = followerHost
where followerUserID = 'bllumusic';
```

![image](https://user-images.githubusercontent.com/43658658/133091281-46c6000e-a99c-4dfb-8700-b331eb42812e.png)
