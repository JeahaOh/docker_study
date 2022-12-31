# Docker PostgreSQL 설치 및 셋팅으로 실습하는 Docker Volume

## PostgreSQL 설치 및 실행

```shell
  docker run -p 5432:5432 --name postgres -e POSTGRES_PASSWORD={password} -d postgres
  Unable to find image 'postgres:latest' locally
  latest: Pulling from library/postgres
  24e221e92a36: Pull complete
  85b20a3ade36: Pull complete
  dfccb87eabcd: Pull complete
  c2b36c21cfca: Pull complete
  b6773915c6e6: Pull complete
  0f1622bae22c: Pull complete
  1fa0f3ae7f27: Pull complete
  90e24ee29327: Pull complete
  5b5e3d50b597: Pull complete
  0109b546d717: Pull complete
  77071f9d5376: Pull complete
  a03caa73b840: Pull complete
  70606bcdcc01: Pull complete
  9911f6f158a5: Pull complete
  Digest: sha256:e2135391c55eb2ecabaaaeef4a9538bb8915c1980953fb6ce41a2d6d3e4b5695
  Status: Downloaded newer image for postgres:latest
  f90882d6c33e6eae670795437bf4a405a549211ddd3466a3cf99c57ceec4dfce



  docker ps -a
  CONTAINER ID   IMAGE      COMMAND                  CREATED              STATUS          PORTS                    NAMES
  f90882d6c33e   postgres   "docker-entrypoint.s…"   About a minute ago   Up 59 seconds   0.0.0.0:5432->5432/tcp   postgres
```

## 컨테이너 접속, USER, DATABASE, TABLE 생성

기본적인 데이터베이스 생성 과정.

```shell
  docker exec -it postgres /bin/bash


  root@f90882d6c33e:/# psql -U postgres
  psql (16.1 (Debian 16.1-1.pgdg120+1))
  Type "help" for help.


  postgres=# CREATE USER initializer PASSWORD '{password}' SUPERUSER;
  CREATE ROLE


  postgres=# CREATE DATABASE init_test OWNER initializer;
  CREATE DATABASE


  postgres=# \c init_test initializer
  You are now connected to database "init_test" as user "initializer".


  init_test=# CREATE TABLE test (
  init_test(#   id integer not null,
  init_test(#   name character varying(32),
  init_test(#   CONSTRAINT test_pk PRIMARY KEY (id)
  init_test(# );
  CREATE TABLE


  init_test=# \dt
            List of relations
  Schema | Name | Type  |    Owner
  --------+------+-------+-------------
  public | test | table | initializer
  (1 row)


  init_test=# SELECT now();
                now
  -------------------------------
  2023-12-23 03:58:20.142533+00
  (1 row)

```

## 컨테이너 재시작시

위 절차를 진행, 단순히 컨테이너를 종료 후 재시작시 데이터가 살아 있음을 확인.

```shell
  docker ps
  CONTAINER ID   IMAGE      COMMAND                  CREATED          STATUS          PORTS                    NAMES
  f90882d6c33e   postgres   "docker-entrypoint.s…"   13 minutes ago   Up 15 seconds   0.0.0.0:5432->5432/tcp   postgres


  docker stop f90882d6c33e
  f90882d6c33e


  docker start f90882d6c33e
  f90882d6c33e


  docker exec -it postgres /bin/bash


  root@f90882d6c33e:/# psql -U postgres
  psql (16.1 (Debian 16.1-1.pgdg120+1))
  Type "help" for help.


  postgres=# \du
                                List of roles
    Role name  |                         Attributes
  -------------+------------------------------------------------------------
  initializer | Superuser
  postgres    | Superuser, Create role, Create DB, Replication, Bypass RLS


  postgres=# \c init_test initializer
  You are now connected to database "init_test" as user "initializer".


  init_test=# \dt
            List of relations
  Schema | Name | Type  |    Owner
  --------+------+-------+-------------
  public | test | table | initializer
  (1 row)

```

## 컨테이너 삭제 후 재 실행시 데이터가 없음 확인

```shell
  docker stop postgres
  postgres


  docker rm postgres
  postgres


  docker exec -it postgres /bin/bash
  Error response from daemon: No such container: postgres


  docker run -p 5432:5432 --name postgres -e POSTGRES_PASSWORD={password} -d postgres
  7f2f0e65e2f9ab0b73831c26fbee3e543bed285a67b5f323ee9d6313ed43d00a


  docker exec -it postgres /bin/bash


  root@7f2f0e65e2f9:/# psql -U postgres
  psql (16.1 (Debian 16.1-1.pgdg120+1))
  Type "help" for help.



  postgres=# SELECT * FROM PG_USER;
  usename  | usesysid | usecreatedb | usesuper | userepl | usebypassrls |  passwd  | valuntil | useconfig
  ----------+----------+-------------+----------+---------+--------------+----------+----------+-----------
  postgres |       10 | t           | t        | t       | t            | ******** |          |
  (1 row)


  postgres=# \du
                              List of roles
  Role name |                         Attributes
  -----------+------------------------------------------------------------
  postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS


  postgres=# \c init_test initializer
  connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  role "initializer" does not exist
  Previous connection kept
```

생성했던 계정 자체가 존재하지 않음.

- 해당 컨테이너를 생성하고 데이터를 변경했을 경우.
  - 컨테이너만 종료 했을 경우  
    컨테이너 내부에 있는 데이터는 그대로 남아있음.
  - 컨테이너를 삭제하는 경우  
    해당 이미지만 재사용하기 때문에 더 이상 데이터가 남아있지 않음.

---

## docker volume

### 볼륨 생성 및 조회 (docker volume create)

[공식문서](https://docs.docker.com/engine/reference/commandline/volume_create/)

volume 생성

- 사용법

```shell
  docker volume create [OPTIONS] [VOLUME]
```

- 예제

```shell
  docker volume create pgdata
```

### docker volume ls

[공식문서](https://docs.docker.com/engine/reference/commandline/volume_ls/)

volume 목록 조회

- 사용법

```shell
  docker volume ls [OPTIONS]
```

- 예제

```shell
  docker volume ls
```

### docker volume inspect

[공식문서](https://docs.docker.com/engine/reference/commandline/volume_inspect/)

1개 이상의 volume 상세 정보 조회

- 사용법

```shell
  docker volume inspect [OPTIONS] VOLUME [VOLUME...]
```

- 예제

```shell
  docker volume inspext pgdata
```

### docer volume rm

[공식문서](https://docs.docker.com/engine/reference/commandline/volume_rm/)

볼륨을 제거함

- 사용법

```shell
  docker volume rm [OPTIONS] VOLUME [VOLUME...]
```

- 예제

```shell
  docker volume rm -f $(docker volume ls)
```

### docker volume prune

[공식문서](https://docs.docker.com/engine/reference/commandline/volume_rm/)

사용하지 않는 모든 볼륨 제거

- 사용법

```shell
  docker volume prune [OPTIONS]
```

- 예제

```shell
  docker volume prune
```

---

## 데이터를 저장할 볼륨 생성 후 반복 테스트

데이터를 종속시키기 위해서 컨테이너 볼륨이나 호스트 컴퓨터에 데이터를 저장할 수 있는 공간을 따로 생성하여 외부에 저장할 수 있는 상태로 만들어, 이를 통해 데이터를 보관하게 할 수 있다.

```shell
  docker stop $(docker ps -a -q)
  7f2f0e65e2f9


  docker rm $(docker ps -q -a)
  7f2f0e65e2f9


  docker volume create pgdata
  pgdata


  docker run -p 5432:5432 --name postgres -e POSTGRES_PASSWORD={password} -d -v pgdata:/var/lib/postgresql/data postgres
  13e22f0f8690282e58dd36609becb4edc7533664da852bb8bc5803b3cdafda43


  docker exec -it postgres /bin/bash
  root@749ee2cac784:/# psql -U postgres
  psql (16.1 (Debian 16.1-1.pgdg120+1))
  Type "help" for help.


  postgres=# CREATE USER initializer PASSWORD '{password}' SUPERUSER;
  CREATE ROLE


  postgres=# CREATE DATABASE init_test OWNER initializer;
  CREATE DATABASE


  postgres=# \du
                                List of roles
    Role name  |                         Attributes
  -------------+------------------------------------------------------------
  initializer | Superuser
  postgres    | Superuser, Create role, Create DB, Replication, Bypass RLS


  exit


  exit


  docker rm -f $(docker ps -qa)
  749ee2cac784


  docker run -p 5432:5432 --name postgres -e POSTGRES_PASSWORD={password} -d -v pgdata:/var/lib/postgresql/data postgres
  8b93d50181bda404e52d4a060d990187d308bafda1124b144552f9db22a8fc85


  docker exec -it postgres /bin/bash


  root@8b93d50181bd:/# psql -U postgres
  psql (16.1 (Debian 16.1-1.pgdg120+1))
  Type "help" for help.


  postgres=# \du
                                List of roles
    Role name  |                         Attributes
  -------------+------------------------------------------------------------
  initializer | Superuser
  postgres    | Superuser, Create role, Create DB, Replication, Bypass RLS
```

## 볼륨 정보

```shell
  docker volume list
  DRIVER    VOLUME NAME
  local     1ff707c2ef36e1dc5ee3e95d3d6ffa24b49ad81446b332a223a9cbdf1453f401
  local     49fe9cc9c4440ca798cf90562241195bcec97f6a829a3b34b38626d51135d54a
  local     944189208ea841ad8550f5c575d129988c848e3ebcddd08de0b6f3f4a3656642
  local     be97a1716f4d80c516ac093212ae25e786ded3a19dbc73ce467fee4c544cbcb6
  local     pgdata


  docker rm -f $(docker ps -qa)


  docker volume inspect pgdata
  [
    {
        "CreatedAt": "2023-12-23T04:34:03Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/pgdata/_data",
        "Name": "pgdata",
        "Options": null,
        "Scope": "local"
    }
  ]


  docker volume remove pgdata


  docker volume list
  DRIVER    VOLUME NAME
  local     1ff707c2ef36e1dc5ee3e95d3d6ffa24b49ad81446b332a223a9cbdf1453f401
  local     49fe9cc9c4440ca798cf90562241195bcec97f6a829a3b34b38626d51135d54a
  local     944189208ea841ad8550f5c575d129988c848e3ebcddd08de0b6f3f4a3656642
  local     be97a1716f4d80c516ac093212ae25e786ded3a19dbc73ce467fee4c544cbcb6


  docker volume prune
  WARNING! This will remove anonymous local volumes not used by at least one container.
  Are you sure you want to continue? [y/N] Y
  Deleted Volumes:
  944189208ea841ad8550f5c575d129988c848e3ebcddd08de0b6f3f4a3656642
  be97a1716f4d80c516ac093212ae25e786ded3a19dbc73ce467fee4c544cbcb6
  1ff707c2ef36e1dc5ee3e95d3d6ffa24b49ad81446b332a223a9cbdf1453f401
  49fe9cc9c4440ca798cf90562241195bcec97f6a829a3b34b38626d51135d54a

  Total reclaimed space: 230.8MB


  docker volume list
  DRIVER    VOLUME NAME
```
