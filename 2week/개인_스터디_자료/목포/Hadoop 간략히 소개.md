# Hadoop 소개

## 하둡이란?

분산 환경에서 빅데이터를 저장하고 처리할 수 있는 자바 기반의 오픈 소스 프레임워크

![Hadoop 구조](https://s3-ap-northeast-2.amazonaws.com/opentutorials-user-file/module/2926/6496.png)

1. 블록 구조의 파일 시스템으로 저장하는 파일은 특정 사이즈의 블록으로 나눠져 분산된 서버에 저장된다.
2. HDFS에는 네임노드 서버 한 대와, 슬레이브 역할을 하는 데이터노드 서버가 여러 대로 구성된다.
3. 네임 노드는 HDFS의 모든 메타데이터(블록들이 저장되는 디렉토리 이름, 파일명 등..)를 관리하고, 클라이언트가 이를 이용해 저장된 파일에 접근할 수 있다.
4. 데이터 노드는 파일을 저장하는 역할을 한다. 또한, 주기적으로 네임노드에 하트비트와 블록리포트를 전달한다. (하트비트는 데이터노드의 동작 여부를 판단하는데 이용)

[출처]
<br/>
https://wikidocs.net/23582#_5

https://opentutorials.org/course/2908/17055
