# postgresqlha

0. /etc/hosts 파일
10.213.194.33 dbmaster
10.213.194.9 dbslave-01
10.213.194.32 pgpool-01

1. postgresql 설치

```bash
sudo apt -y install update
sudo apt -y install upgrade

#최신 버전 레포지토리
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'

sudo apt install wget ca-certificates

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

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

1.2 master-slave 각각 config file 설정
위치
/etc/postgresql/13/main/

설정 후, DB 재시작
```
systemctl restart postgresql
```

1.3 slave server 설정
config 설정 완료 후, DB down
```
systemctl stop postgresql

su postgres

#Data 폴더 삭제
rm -rf /var/lib/postgresql/13/main

pg_basebackup -h masterIP -U replicator -p 5432 -D /var/lib/postgresql/13/main

sudo systemctl start postgresql
```

1.4 정상 동작 확인
마스터 서버
```
#마스터에는 walsender 슬레이브에는 walreceiver가 동작
ps -ef|grep post

create table repl_test(name char(10));
```
슬레이브에서 생성 확인



2. pgpool 설치
```bash
wget https://www.pgpool.net/mediawiki/download.php?f=pgpool-II-4.2.3.tar.gz
tar -xvfz

apt install libpq-dev

#압축 푼 폴더
./configure
make
sudo make install

mkdir /var/run/pgpool

#config setting 해주고, 작동 명령어
pgpool

pgpool -m shutdown stop
```









