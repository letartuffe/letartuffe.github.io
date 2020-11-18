우분투 톰캣 설치

※ 우분투 아래 과정은 root로 진행했으므로 root 아니면 sudo 붙여야함 


| 설치할것 | 버전 |
| ------ | ------ |
| jdk | openjdk-8  |
| 톰캣 | 9 |



# 1. 자바설치

자바필요하므로 자바 설치 되었는지 확인 후 없으면 설치

`java -version`

자바 버전 나오면 설치된 것이므로 아래로 점프
아니면 설치 ㄱ ㄱ ㄱ

`apt install openjdk-8-jdk`


`java -version`


해서 버전 나오면 java 설치 된거임



# 2. 톰캣 유저, 그룹 추가 

그룹 추가

`groupadd tomcat`

유저 추가 ( /opt/tomcat 를 tomcat 홈으로....)

`useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat`


# 3. 이제 톰캣 설치

[톰캣 다운로드 페이지 https://tomcat.apache.org/download-90.cgi](https://tomcat.apache.org/download-90.cgi)

`wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.40/bin/apache-tomcat-9.0.40.tar.gz`

파일 압축해제 하고

`tar -xzvf apache-tomcat-9.0.40.tar.gz`

/opt 아래로 이동

`mv apache-tomcat-9.0.40 /opt/tomcat`

권한주기 

```
chgrp -R tomcat /opt/tomcat
chown -R tomcat /opt/tomcat
chmod -R 755 /opt/tomcat
```





# 4. 서비스 등록


## - 일단 java가 설치된 경로 찾기(JAVA_HOME)

`type -p java|xargs readlink -f|xargs dirname|xargs dirname`

요렇게 해서  나온 경로는 일단 복사해 둘것

'/usr/lib/jvm/java-8-openjdk-amd64/jre'


## - systemd service 파일 만들기
/etc/systemd/system/ 안에 tomcat.service 파일 생성

`vi /etc/systemd/system/tomcat.service`

아래 내용 넣기

```
[Unit]
Description=Apache Tomcat Web Server
After=network.target

[Service]
Type=forking
Environment=JAVA_HOME=**여기는 저기 위에서 복사해 놓은 경로 적을 것**
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat 
Environment=CATALINA_BASE=/opt/tomcat 
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC' 
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom' 

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh
 
User=tomcat
Group=tomcat
UMask=0007

RestartSec=15
Restart=always

[Install]
WantedBy=multi-user.target

```


`JAVA_HOME` 부분 주의 할것!!!!!



systemd 데몬 재시작

`systemctl daemon-reload`



톰캣 시작, 종료 

`systemctl start tomcat`

`systemctl status tomcat`

`systemctl stop tomcat`


톰캣 시작한 후에 


`systemctl status tomcat`


이거 찍었을 때


이렇게 나오면 정상작동

![systemctl_status_tomcat](https://i.ibb.co/xgbzB3L/systemctl-status-tomcat.png)




~~톰캣 자동 시작 등록  --> 이건 안되었음~~

~~`systemctl enable tomcat`~~




# 5. 방화벽 포트 열기

만약 방화벽 있으면 8080포트 열어줘야함. 톰캣 사용 기본 포트 임

`sudo ufw allow 8080`





# 6. 톰캣 고양이 보이는지 브라우저에서 확인


브라우저 열어서 

http://서버아이피주소:8080 

치고 들어갔을 때 고양이 보이면 성공,

안보이면 ... 아까 저위에 /opt/tomcat 디렉토리안에 로그 폴더 가서 로그 보고 원인 찾아야함..






# 7. 관리콘솔 이용하기 (웹에서 설정)

`tomcat-users.xml` 파일 수정 



`vi /opt/tomcat/conf/tomcat-users.xml`

롤과 사용자 추가
manager-gui, admin-gui

아래쪽  </tomcat-users> 태그 앞에 아래 내용 추가

```
<role rolename="manager-gui"/>
<role rolename="admin-gui"/>
<user username="관리자아이디" password="비밀번호" roles="manager-gui,admin-gui"/>
```

설정변경하고 재시작


`systemctl restart tomcat`




context.xml 파일을 수정하면 관리자 접근 아이피도 설정 가능함




참고한 곳 
[https://devops.ionos.com/tutorials/how-to-install-and-configure-tomcat-8-on-ubuntu-1604/](https://devops.ionos.com/tutorials/how-to-install-and-configure-tomcat-8-on-ubuntu-1604/)



