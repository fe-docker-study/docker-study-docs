# Redis의 기본 자료구조와 명령어

## Strings

Redis에서 가장 다양한 데이터타입이다.
텍스트 문자열 외에도 <u>정수(Integer) 또는 부동소수점(Float), 비트맵 값</u>이 기반이고 연관 커맨드를 사용함으로써 동작한다.
문자열은 텍스트(HTML, JSON 등), 숫자형뿐만 아니라 바이너리 데이터(이미지, 비디오, 오디오 파일)까지 저장 가능하다.
문자열이 값은 512MB를 초과할 수 없다.

### 사용 예시

- 자동 만료되는 캐시 : 데이터베이스 질의 실행이 오래걸리고 일정 시간 동안 캐시되어야 할 떄 매우 유용하다. (만료 시간을 갖는 데이터)
  - 커맨드 : SETEX, EXPIRE, EXPIREAT
  ```
  > setex key 5 value
  OK
  > ttl key
  2 // 남은 시간 (second)
  > get key
  (nil) // 만료됨
  ```
- 개수 계산 : 페이지 뷰, 비디오 뷰, 좋아요 같은 개수 계산
  - 커맨드 : INCR, INCRBY, DECR, DECRBY, INCRFLOATBY

### 인터넷 기사 좋아요 시스템 코드 예제

```JavaScript
var redis = require("redis");
var client = redis.createClient();

function upLike(id) {
    var key = "article:" + id + ":likes";
    client.incr(key);
}

function downLike(id) {
    var key = "article:" + id + ":likes";
    client.decr(key);
}

function showResults(id) {
    var headlineKey = "article:" + id + ":headline";
    var voteKey = "article:" + id + ":likes";

    //redis에서 값 가져오기
    // key를 전달하면 각 key의 value를 순서대로 반환
    client.mget([headlineKey, voteKey], function(err, replies) {
        console.log("The article " + replies[0] + " has "  replies[1] + " likes");
    });
}


upLike(12345);
upLike(12345);
upLike(12345);

showResults(12345);
client.quit();
```

#### 결과

```JavaScript
The article Google Wants to Turn Your Clothes Into a Computer has 3 likes
```
