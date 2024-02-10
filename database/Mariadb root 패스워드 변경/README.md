# Mac 환경에서 mariadb root 패스워드 변경

## brew를 통한 mariadb 설치
Mac 환경에서 brew install manager를 통하여 mariadb를 설치하고 mariadb 명령어를 실행하는 mysql command 클라이언트를 통하여 mariadb에서 접속할 수 있ㄷ다

## brew 패키지 매니저를 통하여 mariadb 설치
```shell
brew install mariadb
```

## command에서 mariadb 접속
```shell
mariadb
```

root 패스워드 설정은 최초 설치 이후 default password가 설정된 상태에서 mariadb에 mysql 클라이언트를 통하여 접속 한 이후에 설정이 가능하다

## root 계정 패스워드설정
아래와 같이 root 계정 패스워드를 설정한후 변경사항을 반영한다
```shell
set password for 'root'@'localhost'=password('설정할 패스워드')
flush privileges
```

변경이후 user 테이블에서 root 계정에 대한 패스워드를 확인해 보면 Hash된 패스워드 값으로 변경된 것을 확인할 수 있다
```shell
select host, user, password from user;
```

변경된 패스워드를 통하여 정상적으로 접속이 되는지 Database Client로 접속을 시도해봐도 되고, mariadb 클라이언트를 통하여 접속 테스트를 하여 정상적으로 database에 접속되는지 확인해 볼 수 있다
```shell
mariadb -u root -p
```