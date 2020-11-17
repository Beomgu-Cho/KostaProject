# KostaProject
Home IOT Service development

# Ubuntu에 MySQL, MariaDB 설치 및 실행 -- 2020/11/17

#### 실행 환경 Oracle VM VirtualBox 런처 Ubuntu 20.04.1

## 라즈베리파이에서 MySQL 환경을 생성하기 전에 테스트를 위해 환경을 구성하였습니다.

### 1. update & upgrade 진행
#### apt-get update, apt-get upgrade 명령어를 우선 진행합니다.
'''
  sudo apt-get update -y
  sudo apt-get upgrade -y
'''
### 2. 설치되어있을지도 모르는 mysql, mariadb 및 디렉터리 제거
'''
  sudo apt-get purge mysql* mariadb* -y
  sudo apt-get autoremove -y
  sudo apt-get autoclean -y
  sudo rm -rf /usr/sbin/mysqld /var/run/mysqld /etc/mysql
'''
### 주의!! rm -rf 명령어는 강제 삭제 명령이기 때문에 오타에 신경써야합니다.

### 3. MariaDB 설치
#### MariaDB와 관련된 패키지는 apt-cache search mariadb 로 확인이 가능합니다.
'''
  sudo apt-get install mariadb-server -y
'''
