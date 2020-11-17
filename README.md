# KostaProject
Home IOT Service development


# Ubuntu에 MySQL, MariaDB 설치 및 실행 -- 2020/11/17

#### 실행 환경 Oracle VM VirtualBox 런처 Ubuntu 20.04.1

## 라즈베리파이에서 MySQL 환경을 생성하기 전에 테스트를 위해 환경을 구성하였습니다.


### 1. update & upgrade 진행
---------
#### apt-get update, apt-get upgrade 명령어를 우선 진행합니다.
```
  >sudo apt-get update -y
  >sudo apt-get upgrade -y
```


### 2. 설치되어있을지도 모르는 mysql, mariadb 및 디렉터리 제거
----------
```
  >sudo apt-get purge mysql* mariadb* -y
  >sudo apt-get autoremove -y
  >sudo apt-get autoclean -y
  >sudo rm -rf /usr/sbin/mysqld /var/run/mysqld /etc/mysql
```
### 주의!! rm -rf 명령어는 강제 삭제 명령이기 때문에 오타에 신경써야합니다.



### 3. MariaDB 설치
-----------
#### MariaDB와 관련된 패키지는 apt-cache search mariadb 로 확인이 가능합니다.
```
  sudo apt-get install mariadb-server -y
```


### 4. MySQL 실행하기
---------
#### 초기 root 사용자 접속허용 및 비밀번호 설정이 되어있지 않기 때문에 관리자 권한을 이용하여 실행하여야 합니다.
```
  >sudo mysql
```
#### DB 리스트 확인
 ```
  >show databases;
```
##### sql 명령어를 이용할 땐 반드시 세미콜론(';')을 넣어야 해당 문장이 실행됩니다.
```
  >MariaDB[(none)]> use mysql;
```
#### mysql 데이터베이스에 접속
#### 접속에 성공하면 MariaDB[(none)] 이
#### MariaDB[mysql] 로 바뀝니다.
```
  >MariaDB[mysql]> select host, user, password from user;
```
#### user 테이블의 host, user, password 가 출력됩니다.



### 5. 비밀번호 변경
--------------
#### 이제 root의 password를 변경시켜주기 위해 update 명령어를 사용합니다.
```
  >MariaDB[mysql]> update user set password=password('[패스워드]') where user='root';
```
#### root 의 password를 [패스워드]로 변경합니다.
```
  >MariaDB[mysql]> select host, user, password from user;
```
#### 여기에서 보이는 password 는 암호화가 된 password 이기 때문에 원래 입력한 문자와는 다르게 보입니다.



### 6. 접근 가능한 host 추가
------------
#### localhost 이외의 사용자가 사용할 수 있게 접근 가능한 host 를 리스트에 추가해주어야 합니다.
```
  >MariaDB[mysql]> grant all privileges on *.*'root'@'%' identified by '[패스워드]';
```
#### [패스워드] 는 외부 사용자가 접근할 때 사용될 패스워드 입니다.
#### 참고로 특정 IP의 접근만을 허용할 경우 % 대신 해당 IP를 넣으면 됩니다.
#### 업데이트가 끝나면 flush 명령어로 적용시킵니다.
```
  >MariaDB[mysql]> flush privileges;
```


### 7. 50-server.cnf 수정
-----------
```
  >sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
#### nano 편집기를 이용하여 bind-address = 127.0.0.1 로 되어있는 부분을 주석처리 합니다.
#### bind-address = 127.0.0.1 ==> # bind-address = 127.0.0.1

#### 수정이 완료되면 ctrl + x, y, enter를 차례로 입력하여 빠져나옵니다.



### 8. Plugin 확인
----------
#### 다시 sql로 들어가 위에서 확인했던 table 에서 plugin을 확인합니다.
```
  >sudo mysql
```
```
  >MariaDB[(none)]> use mysql
  >MariaDB[mysql]> select host, user, plugin from user;
```
