# postgresqlha

0. 공통적으로 처리할 것
```
a. /etc/hosts 파일 셋팅
10.213.194.33 dbmaster
10.213.194.9 dbslave-01
10.213.194.32 pgpool-01

b. apt upgrade, update
```

1. postgresql 설치

```bash
#root로 실행
apt update
apt upgrade

#최신 버전 레포지토리
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'

sudo apt install wget ca-certificates

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

apt update
apt upgrade

apt show postgresql
#13version 

sudo apt -y install postgresql postgresql-contrib

#설치확인
service postgresql status

```
1.1 master server
```bash
su - postgres
psql
create user replicator with replication encrypted password 'new1234!';
```
db config file 설정
위치
/etc/postgresql/13/main/

설정 후, DB 재시작
```
systemctl restart postgresql
```

1.2 slave server 설정
config 설정 완료 후, DB down
```
systemctl stop postgresql

su postgres

#Data 폴더 삭제
rm -rf /var/lib/postgresql/13/main

pg_basebackup -h dbmaster -U replicator -p 5432 -D /var/lib/postgresql/13/main -Xs -P -R

sudo systemctl start postgresql
```

1.3 정상 동작 확인
마스터 서버
```
#마스터에는 walsender 슬레이브에는 walreceiver가 동작
ps -ef|grep post

create table repl_test(name char(10));
```
슬레이브에서 생성 확인
```
su postgres
psql
select * from repl_test;
or
\dt
```

2. pgpool 설치
apt update, upgrade 해주고,
```bash
wget https://www.pgpool.net/mediawiki/download.php?f=pgpool-II-4.2.3.tar.gz
tar xvfz 파일명

apt install libpq-dev
apt install gcc
apt install make


sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'

sudo apt install wget ca-certificates

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

apt update

#13version client 
apt show postgresql-client

apt install postgresql-client-common
apt install postgresql-client

#압축 푼 폴더에서 인스톨
./configure
make
sudo make install

mkdir /var/run/pgpool
```

아래 위치에서 파일 설정(failover.sh, pgpool.conf)
/usr/local/etc
```
cp pool_hba.conf.sample pool_hba.conf
chown 744 failover.sh
```

ssh key 생성해서, DB 서버의 postgres 계정 .ssh/authorized_keys에 입력
```
ssh-keygen -t rsa
```
혹시, 접속 문제 생길때 /etc/ssh/sshd_config 확인 - 퍼블릭키인증 yes


패스워드 암호화 키 생성
```
pg_md5 -m -p -u postgres
#패스워드 입력
#/usr/local/etc/pool_passwd 파일이 생성됨

```

```
#작동 명령어
pgpool
#shutdown
pgpool -m shutdown stop
```

동작 확인
```
psql -h localhost -U postgres -p 9999
show pool_nodes;
# server 둘다 status up  되어 있는지 확인
```

마스터 DB 서버에서 서비스 내려보고, 자동 failover 확인
다시 원복 시킬때는 슬레이브 DB에 데이터 폴더 지우고, db_baseback~ 명령어 처리하는 부분부터 다시 셋팅






