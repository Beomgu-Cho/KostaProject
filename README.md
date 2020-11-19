# KostaProject - DBSet
Home IOT Service development




# Ubuntu에 MySQL, MariaDB 설치 및 실행 -- 2020/11/17
![screenshots mariadb_image.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/mariadb_image.png)
![screenshots ubuntu_image.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/ubuntu_image.png)
#### 실행 환경 Oracle VM VirtualBox 런처 Ubuntu 20.04.1

## 라즈베리파이에서 MySQL 환경을 생성하기 전에 테스트를 위해 환경을 구성하였습니다.

## 1. update & upgrade 진행
---------
#### `apt-get update`, `apt-get upgrade` 명령어를 우선 진행합니다.
```
  $ sudo apt-get update -y
  $ sudo apt-get upgrade -y
```

## 2. 설치되어있을지도 모르는 `mysql`, `mariadb` 및 디렉터리 제거
----------
```
  $ sudo apt-get purge mysql* mariadb* -y
  $ sudo apt-get autoremove -y
  $ sudo apt-get autoclean -y
  $ sudo rm -rf /usr/sbin/mysqld /var/run/mysqld /etc/mysql
```
###### 주의!! `rm -rf` 명령어는 강제 삭제 명령이기 때문에 오타에 신경써야합니다.

## 3. MariaDB 설치
-----------
#### MariaDB와 관련된 패키지는 `apt-cache search mariadb` 로 확인이 가능합니다.
```
  $ sudo apt-get install mariadb-server -y
```

## 4. MySQL 실행하기
---------
#### 초기 root 사용자 접속허용 및 비밀번호 설정이 되어있지 않기 때문에 관리자 권한을 이용하여 실행하여야 합니다.
```
  $ sudo mysql
```
![screenshots sudo_mysql.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/sudo_mysql.PNG)
##### mysql 실행 후 sql 명령어를 이용할 땐 반드시 세미콜론(`;`)을 넣어야 해당 문장이 실행됩니다

#### DB 리스트 확인
```
  MariaDB[(none)]> show databases;
```
![screenshots show_databese.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/show_databases.PNG)
#### mysql 데이터베이스에 접속
```
  MariaDB[(none)]> use mysql;
```
#### 접속에 성공하면 MariaDB[(none)] 이
#### MariaDB[mysql] 로 바뀝니다.
![screenshots use_mysql.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/use_mysql.PNG)

```
  MariaDB[mysql]> select host, user, password from user;
```
#### user 테이블의 host, user, password 가 출력됩니다.

|host|user|password|
|---|---|---|
|localhost|root||


![screenshots select_password_1.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/select_password_1.PNG)
## 5. 비밀번호 변경
--------------
#### 이제 root의 password를 변경시켜주기 위해 update 명령어를 사용합니다.
```
  MariaDB[mysql]> update user set password=password('0000') where user='root';
```
#### root 의 password를 0000으로 변경합니다.
###### 원하는 비밀번호로 자유롭게 수정 가능합니다.

```
  MariaDB[mysql]> select host, user, password from user;
```

|host|user|password|
|---|---|---|
|localhost|root|\*97E7471D816A37E38510728AEA47440F9C6E2585|

![screenshots update_password.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/update_password.PNG)
#### 여기에서 보이는 password 는 암호화가 된 password 이기 때문에 원래 입력한 문자와는 다르게 보입니다.

## 6. 접근 가능한 host 추가
------------
#### localhost 이외의 사용자가 사용할 수 있게 접근 가능한 host 를 리스트에 추가해주어야 합니다.
```
  MariaDB[mysql]> grant all privileges on *.* to 'root'@'%' identified by '0000';
```
|host|user|password|
|---|---|---|
|localhost|root|\*97E7471D816A37E38510728AEA47440F9C6E2585|
|%|root|\*97E7471D816A37E38510728AEA47440F9C6E2585|


#### 0000은 외부 사용자가 접근할 때 사용될 패스워드 입니다.
###### 원하는 비밀번호로 자유롭게 수정 가능합니다.
###### 참고로 특정 IP의 접근만을 허용할 경우 % 대신 해당 IP를 넣으면 됩니다.
![screenshots grant_'%'.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/grant_'%25'.PNG)
#### 업데이트가 끝나면 `flush` 명령어로 적용시킵니다.
```
  MariaDB[mysql]> flush privileges;
```
![screenshots flush.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/flush.PNG)
#### 모든 작업이 완료되면 `exit` 명령으로 mysql을 종료합니다.
![screenshots db_exit.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/db_exit.PNG)
## 7. 50-server.cnf 수정
-----------
```
  $ sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
#### nano 편집기를 이용하여 bind-address = 127.0.0.1 로 되어있는 부분을 주석처리 합니다.
##### bind-address = 127.0.0.1 ==> `# bind-address = 127.0.0.1`

#### 수정이 완료되면 ctrl + x, y, enter를 차례로 입력하여 빠져나옵니다.

## 8. Plugin 확인 및 변경
----------
#### 다시 sql로 들어가 위에서 확인했던 table 에서 plugin을 확인합니다.
```
  $ sudo mysql
```
```
  MariaDB[(none)]> use mysql
  MariaDB[mysql]> select host, user, plugin from user;
```

#### unix_socket 플러그인을 설정한 root 비밀번호로 접속할 수 있도록 수정합니다.
```
  MariaDB[mysql]> update user set plugin='mysql_native_password' where user='root';
  MariaDB[mysql]> flush privileges;
```
![screenshots update_plugin.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/update_plugin.PNG)
#### 이로써 관리자 권한 없이도 root 비밀번호를 통해 데이터베이스에 접근이 가능해졌습니다.
```
  $ mysql -u root -p
  Enter password:
```
#### 비밀번호 입력 후 정상적으로 데이터베이스에 접근할 수 있습니다.
![screenshots mysql_-u_root_-p.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/mysql_-u_root_-p.PNG)
# Python 을 통한 MySQL 접근 테스트 -- 2020/11/18
#### python 3.8.5 ver 사용
#### vi 에디터 사용
#### 사용 라이브러리 : `pymysql`, `sqlalchemy`, `pandas`
## python 을 이용하여 실시간으로 데이터베이스 수정 및 접근을 위해 테스트 하였습니다.

## 1. MySQL 접속정보 체크
--------------------------------
#### 기본설정값을 이용하게 되지만 경우에 따라 PORT 번호가 주석처리 되어있을 수 있으므로 수정해야 합니다.
#### 임의로 PORT 번호를 수정하는 것도 가능합니다.
```
  $ cd /etc/mysql/mariadb.conf.d
  $ sudo nano 50-server.cnf
```
##### PORT = 3306 의 주석처리를 빼고 포트 번호 수정을 원한다면 원하는 번호로 수정이 가능합니다.
![screenshots #port.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/%23port.PNG)
## 2. MySQL DataBase 생성
------------------------------
#### mysql에 접속합니다.
```
  $ mysql -u root -p
  비밀번호 입력
```

#### sql `create` 문으로 데이터베이스를 생성합니다.
```
  MariaDB[(none)]> create database project;
```
#### project 라는 이름의 데이터베이스가 생성되었습니다.
#### project 대신 생성하고자 하는 데이터베이스명을 입력하면 원하는 이름으로 생성 가능합니다.
#### 데이터베이스 생성 확인
```
  MariaDB[(none)]> show databases;
```
![screenshots create_database.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/create_database.PNG)
## 3. PYTHON
-----------------------
### 3-1. 라이브러리 설치
#### python 실행에 앞서 필요 라이브러리를 다운 받아야 합니다.
```
  $ pip3 install pymysql
  $ pip3 install sqlalchemy
  $ pip3 install pandas
```
#### 만약 pip 에러가 발생한다면 pip 패키지의 설치 여부를 확인해보아야 합니다.
```
  $ sudo apt-get install python3-pip
```
### 3-2. 패키지 참조
```
  import pymysql
  import pandas as pd
  from sqlalchemy import create_engine
  from pandas import DataFrame
```

### 3-3. MySQL 연동 정보
#### 변수로 미리 연동 정보를 선언해 놓습니다.
```
  HOSTNAME = "localhost"
  PORT = 3306            # 변경했을 경우 변경한 포트번호
  USERNAME = "root"
  PASSWORD = "0000"      # MySQL 접속 유저 패스워드
  DATABASE = "project"
  CHARSET1 = "utf8"
  CHARSET2 = "utf-8"
```

### 3-4. 접속 문자열 생성
#### MySQL 접속을 위한 문자열을 미리 생성해놓습니다.
```
  con_str_fmt = "mysql+mysqldb://{0}:{1}@{2}:{3}/{4}?charset={5}"
  con_str = con_str_fmt.format(USERNAME, PASSWORD, HOSTNAME, PORT, DATABASE, CHARSET1)
```

-con_str 출력 결과
```
  > 'mysql+mysqldb://root:0000@localhost:3306:/project?charset=utf8'
```

### 3-5. pymysql을 사용하여 MySQL 연동 객체 설치
#### pymysql 라이브러리에서 제공하는 `install_as_MySQLdb()` 함수를 사용하여 MySQL과 연동합니다..
```
  pymysql.install_as_MySQLdb()
  import MySQLdb
```
#### sqlalchemy의 `create_engine()` 함수로 데이터베이스에 접속합니다.
###### 인수는 위에서 생성한 con_str과 CHARTSET2를 사용합니다.
```
  engine = create_engine(con_str, encoding=CHARSET2)
  conn = engine.connect()
```
#### 여기까지의 과정으로 python을 이용한 MySQL 접근이 완료 되었습니다.

## 4. 테이블 생성 및 수정
--------
### 4-1. 테이블 생성 및 수정
#### pandas 라이브러리로 쉽게 `DataFrame` 을 생성할 수 있습니다.
#### DataFrame 은 일반적인 DB Table과 동일한 형태를 가집니다.
```
  df1 = DataFrame([
              {"deptno": 300, "dname": "name1", "loc": "loc1"},
              {"deptno": 301, "dname": "name2", "loc": "loc2"},
              {"deptno": 302, "dname": "name3", "loc": "loc3"}
            ])
```
#### df1 의 출력 결과는 아래와 같습니다.
||deptno|dname|loc|
|---|---:|---:|---:|
|0|300|`name1`|`loc1`|
|1|301|`name2`|`loc2`|
|2|302|`name2`|`loc2`|
#### 생성한 DataFrame의 인덱스를 DataBase에 저장합니다.
```
  df1.to_sql(name="department_py", con=conn, if_exists='replace', index=True)
```

###### `name` : 생성할 테이블명|
###### `con` : sql connection 정보|
###### `if_exists` : 같은 이름의 테이블이 있을 때 수행할 명령입니다. 'replace'의 경우 덮어쓰기 입니다.|
###### `index` : 열의 기본적인 번호를 새겨주는 index 열 생성 정보 입니다. False 일 경우 생성되지 않습니다.|
#### 위 명령어로 인해 MySQL서버의 project 데이터베이스에 'department_py' 라는 table이 형성되었습니다.|

### 4-2. 테이블 조회
#### mysql로 직접 들어가 생성되었는지 확인해봅니다.
```
  $ mysql -u root -p
  비밀번호
```
```
  MariaDB[(none)]> use project
  MariaDB[project]> show tables;
```
#### `show` 명령어로 department_py 라는 테이블이 생성되었음을 확인할 수 있습니다.
#### `select` 문으로 내용을 확인합니다.
```
  MariaDB[project]> select * from department_py;
```
#### 생성된 테이블은 아래와 같습니다.
|index|deptno|dname|loc|
|---|---:|---:|---:|
|0|300|`name1`|`loc1`|
|1|301|`name2`|`loc2`|
|2|302|`name2`|`loc2`|


![screenshots department_py.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/department_py.PNG)
#### index열을 없애고 싶다면 `to_sql()`함수에서 `index=False`로 변경하면 index열 없이 생성됩니다.

### 4-3. 데이터베이스 연결 해제
#### 테이블 생성이 끝났다면 접속을 해제해야 합니다.
#### 위에서 선언한 conn을 닫는 것으로 접속 해제가 가능합니다.
```
  conn.close()
```

# RaspberryPi SQL환경 테스트
#### Raspberry 4 모델 사용 16GB Storage
#### 제어환경 windows 10 원격데스크톱

## 1. 라즈베리파이 MySQL 설치 및 테스트
-----------
#### Ubuntu 환경과 동일하게 진행한 결과 이상없이 환경 구성에 성공하였습니다.
![screenshots raspberry_test.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/20201119/raspberry_test.PNG)

## 2. 외부접근 테스트
----------
#### 접근 환경은 가상머신 Ubuntu 에서 RaspberryPi 로 접근하는 것을 테스트 하였습니다.

### 2-1. 라즈베리파이 IP 설정
#### 우선 현재 라즈베리파이의 IP를 체크합니다. 접속 환경은 Wi-Fi 입니다.
```
  $ ifconfig
```
![screenshots ifconfig.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/20201119/ifconfig.PNG)
#### ifconfig 정보에서 Wi-Fi 환경이 맞다면 wlan0에 보이는 ip 정보를 기억해야 합니다.
###### `inet`, `netmask`, `broadcast`
###### 일반적으로 network gateway는 xxx.xxx.xxx.1 이고 network ip 는 xxx.xxx.xxx.0 입니다.

#### 이제 라즈베리파이에 고정 IP를 할당해야 합니다. Wi-Fi 고정IP 할당을 하기 위해 파일의 텍스트를 수정해야 합니다.
```
  $ sudo nano /etc/network/interfaces
```
![screenshots nano_interfaces.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/20201119/nano_interfaces.PNG)
#### 위의 이미지와 같이 내용을 추가하시면 됩니다.
#### wlan0 는 고정할 IP의 정보
#### wlan1 은 만약 다른 네트워크 망에 접속할 경우 할당받을 공간을 남겨놓는 것입니다.

#### 정상적으로 완료했다면 재부팅을 하여 IP를 확인하면 됩니다.

### 2-2. 접근 허용 아이피 설정
```
  $ sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
#### bind_address 부분을 수정합니다.
#### 특정 IP의 접근을 원한다면 network ip 를 입력하면 됩니다.
#### bind_address = 0.0.0.0 으로 설정하면 모든 IP에서 접근이 가능합니다.
![screenshots bind_address.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/20201119/bind_address.PNG)

### 2-3. DataBase 수정
#### Ubuntu와 RaspberryPi를 구분하기 위해 추가 테이블을 생성합니다.
#### python 코드에 df2 부분을 추가하여 생성했습니다.
![screenshots new_table.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/20201119/raspi_new_table.PNG)

### 2-4. Ubuntu 에서 MySQL 원격 접속
#### MySQL 커멘드에 `-h [라즈베리파이 IP]` 를 추가하여 IP 접속을 요청합니다.
```
  $ mysql -u root -h 192.168.0.185 -p
```
#### 이후 DataBase 접속 및 Table 을 확인합니다.
```
  MariaDB[(none)]> use project
  MariaDB[project]> show tables;
```
|Tables_in_project|
|---|
|department_py|
|test_py|
```
  MariaDB[project]> select * from test_py;
```
|index|user:|input:|output:|
|---|---:|---:|---:|
|0|root|in|welcome|
|1|host|in|welcome|
|2|guest|out|goodbye|
![screenshots ubunto_to_raspi.png](https://github.com/Beomgu-Cho/KostaProject/blob/main/screenshots/20201119/ubuntu_to_raspi.PNG)
#### 정상적으로 확인이 완료되었습니다.
