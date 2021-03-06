# Instagram Copy

## 개요

> Instagram을 토대로 데이터를 설계(ERD)해보고, 데이터베이스를 구축하여 Instagram의 각 화면을 구성하는 데이터를 Query문을 통해 가져오자.

## 목적

> 우리 눈에 보여지는 화면 속 데이터들의 이면에 감춰진 전반적인 DB 구축 및 조건에 맞는 데이터 Select 과정을 이해해보자.   
> 여기서 오는 고충과 고민을 이해해보자.   
> 효율적인 데이터 설계를 고민해보자.   
> MySQL의 Query문에 익숙해지자.

## 데이터 설계 및 데이터베이스 구축 과정

1. `AQueryTool`을 이용해 [**테이블 설계**](https://github.com/khyup0629/Instagram-copy/blob/main/Instagram_Database_Copy/Instagram%20DB%20Table%20MySQL.txt)
2. `Datagrip`으로 **테이블 생성 및 데이터 작성**
3. Instagram을 토대로 **각 화면에 띄워야 할 칼럼**을 `Excel`로 정리
  - 타임라인   
![image](https://user-images.githubusercontent.com/43658658/134375698-a4c946d9-8674-4e6b-bbdf-f15d75d9473f.png)
  - 활동 리스트   
![image](https://user-images.githubusercontent.com/43658658/134375799-f7221423-49ad-4b61-b4e2-6e072377027a.png)
  - 유저 프로필   
![image](https://user-images.githubusercontent.com/43658658/134376169-7a8ec8cf-6e06-411a-babb-20abb3ab975c.png)  
  - 팔로우 & 팔로워 목록   
![image](https://user-images.githubusercontent.com/43658658/134376366-2450f760-a00d-4532-98f2-123ef6887464.png)
  - 피드   
![image](https://user-images.githubusercontent.com/43658658/134376447-35b757b9-97f6-494f-b045-5b0279639e54.png)
  - 스토리   
![image](https://user-images.githubusercontent.com/43658658/134376586-a97c1c88-e148-4fd8-81b5-411847e0e56a.png)
  - 채팅 목록   
![image](https://user-images.githubusercontent.com/43658658/134376655-2ea6621b-3273-4a72-bf37-d82c0b924a6f.png)
4. `Query문`을 통한 **조건에 맞는 데이터 select**
  - [타임라인](https://github.com/khyup0629/Instagram-copy/blob/main/Instagram_Database_Copy/Instagram%20DB%20%ED%83%80%EC%9E%84%EB%9D%BC%EC%9D%B8.md#%ED%83%80%EC%9E%84%EB%9D%BC%EC%9D%B8-%ED%99%94%EB%A9%B4)
  - [활동 리스트](https://github.com/khyup0629/Instagram-copy/blob/main/Instagram_Database_Copy/Instagram%20DB%20%ED%99%9C%EB%8F%99%20%EB%A6%AC%EC%8A%A4%ED%8A%B8.md#%ED%99%9C%EB%8F%99-%EB%A6%AC%EC%8A%A4%ED%8A%B8)
  - [유저 프로필](https://github.com/khyup0629/Instagram-copy/blob/main/Instagram_Database_Copy/Instagram%20DB%20%ED%94%84%EB%A1%9C%ED%95%84%20%ED%99%94%EB%A9%B4.md#%ED%94%84%EB%A1%9C%ED%95%84-%ED%99%94%EB%A9%B4)
  - [팔로우 & 팔로워 목록](https://github.com/khyup0629/Instagram-copy/blob/main/Instagram_Database_Copy/Instagram%20DB%20%ED%8C%94%EB%A1%9C%EC%9B%8C%2C%20%ED%8C%94%EB%A1%9C%EC%9E%89%20%EB%A6%AC%EC%8A%A4%ED%8A%B8.md#%ED%8C%94%EB%A1%9C%EC%9B%8C-%EB%A6%AC%EC%8A%A4%ED%8A%B8)
  - [피드](https://github.com/khyup0629/Instagram-copy/blob/main/Instagram_Database_Copy/Instagram%20DB%20%ED%94%BC%EB%93%9C.md#%ED%94%BC%EB%93%9C)
  - [스토리](https://github.com/khyup0629/Instagram-copy/blob/main/Instagram_Database_Copy/Instagram%20DB%20%EC%8A%A4%ED%86%A0%EB%A6%AC%20%ED%99%94%EB%A9%B4.md#%EC%8A%A4%ED%86%A0%EB%A6%AC-%ED%99%94%EB%A9%B4)
  - [채팅 목록](https://github.com/khyup0629/Instagram-copy/blob/main/Instagram_Database_Copy/Instagram%20DB%20%EC%B1%84%ED%8C%85%EB%B0%A9%20%EB%AA%A9%EB%A1%9D%20%ED%99%94%EB%A9%B4.md#%EC%B1%84%ED%8C%85%EB%B0%A9-%EB%AA%A9%EB%A1%9D-%ED%99%94%EB%A9%B4)

## 배운 점

- 데이터 설계가 효율적으로 이루어지지 않으면 Query문을 작성할 때 복잡해질 수 있다.
- 구현해야 할 기능을 먼저 생각한 뒤 그 조건에 따라 데이터 설계를 해야 효율적인 데이터 설계가 가능한 것 같다. 
- 만약 데이터 개수가 기하급수적으로 많다면, 지금의 Query문보다 더 효율적인 Query문을 작성해야 할 것 같다.
- 어떤 테이블을 이용하느냐에 따라 Query문 작성 난이도가 달라진다.
- 평상시 무심코 사용했던 기능들이 그 데이터를 Select할 땐 여러 조건들을 꼼꼼하게 고려해서 나온 결과물임을 알게 되었다.

---
