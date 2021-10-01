# Docker의 run 옵션들과 exec



`이 글은 생활코딩(egoing)의 Docker 입문수업을 참고하여 정리한 자료입니다. (하단 참고자료 표기)`

----
# 1) 포트포워딩 (run -p 옵션)


단지 'docker run httpd' 로 웹서버(아파치) 컨테이너를 생성하면
외부 클라이언트가 해당 서버에 바로 접속할 수 있을까?

 없다... 왜냐하면 호스트의 포트와 컨테이너의 포트가 연결되어 있지 않기 때문이다.

 따라서 컨테이너 실행 시에 포트 연결 옵션을 주어야 한다. 이것을 **포트 포워딩**이라고 한다.
 ```bash
 # host의 80번 포트와 container의 80번 포트를 연결
 $ docker run -p 80:80 httpd
 
 # host의 8000번 포트와 container의 80번 포트를 연결
 $ docker run -p 8000:80 httpd
 ```
![](https://images.velog.io/images/monadk/post/ff31bb6c-6f72-4722-8cba-4e7cccb5f766/image.png)



# 2) 명령어 실행 (exec)



컨테이너에 명령어를 전달하여 실행시킬 수 있다.

```bash
# docker exec <컨테이너명> <명령어>
$ docker exec monad_apache pwd
```
컨테이너와 지속적으로 연결을 유지하면서 명령어를 전달할 수도 있다.
**-it**는 터미널과 컨테이너가 지속적으로 연결되도록 하는 옵션이다.
(STDIN 표준 입출력을 열고 가상 tty를 통해 접속)
```bash
# docker exec -it <컨테이너명> /bin/sh (본쉘)
# docker exec -it <컨테이너명> /bin/bash (bash쉘)
$ docker exec -it monad_apache /bin/bash
# 위 커맨드를 실행하고 나면 monad_apache의 bash쉘에 지속적으로 명령어를 전달하게 된다.
# exit 하면 연결이 종료됨
```



# 3) Host와 Container의 File System 연결



호스트의 로컬 파일시스템을 컨테이너 파일시스템에 연결할 수 있다.
로컬에서 변경된 내용이 컨테이너에도 반영되어 개발이 보다 쉬워지고, 소스코드 버전관리에도 용이해질 것이다.
실행환경은 컨테이너에 맡기고, 파일을 수정하는 것은 호스트에서 진행할 수 있게 되는 것이다.👏
**-v** : 볼륨 옵션!

 ```bash
 # docker run -v <host파일경로>:<container파일경로> <이미지>
 $ docker run -p 8888:80 -v ~/Desktop/htdocs:usr/local/apache2/htdocs/ httpd

 ```

![](https://images.velog.io/images/monadk/post/70b499cd-2a91-4e93-9692-b1fb805d5b11/image.png)

---

`※참고 자료※`
<https://www.youtube.com/watch?v=SJFO2w5Q2HI&list=PLuHgQVnccGMDeMJsGq2O-55Ymtx0IdKWf&index=5>