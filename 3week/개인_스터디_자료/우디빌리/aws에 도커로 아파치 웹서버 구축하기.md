### 1. docker hub에서 아파치 웹 서버 이미지 검색

docker search httpd

### 2. docker hub에서 도커 이미지 다운로드

docker pull httpd

### 3. 이미지 → 컨테이너 실행

docker run -d —name httpd-t -p 80:80 httpd

- -d : 백그라운드로 실행
- —name : 컨테이너 이름을 지정
- -p : 포트를 지정.  [호스트포트]:[컨테이너 포트] 포트 포워딩 설정

### 4. 실행 확인

docker ps

### 5. aws 포트 open

- 웹 브라우저로 웹 서버에 접속하기 위해서는 AWS의 포트를 열어줘야함
- AWS 인스턴스 → 인바운드 규칙 추가
