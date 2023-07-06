---
title: "postgresql 아카이브 backup 설정"
date: 2023-07-05T00:19:00+09:00
categories: 
    - DB
share: false
---

# 페이지 목차
1. [postgresql 아카이브 backup 설정]({% post_url /db/2023-07-05-postgresql 아카이브 backup 설정 %})

## postgresql 백업
어떠한 장애나 문제발생으로 인해, 데이터베이스 장애 발생시 백업 데이터로 복구가 필요하다.
postgresql에서 제공하는 아카이브 백업 설정 방법은 다음과 같다.

1. postgresql.conf 파일 옵션 설정
 - archive_command: WAL 파일을 아카이브 시키는 명령어(%p : WAL 파일의 절대경로, %f: 저장할 로그 파일의 이름)
 example) cp %p /var/lib/pgsql/14/data/backup/%f
 - archive_mode : on -WAL 파일이 archive_command 설정에 따라 아카이브 저장소로 전달.

다음과 같은 옵션 적용
wal_level = replica
archive_command = 'cp %p /var/lib/pgsql/14/data/backup/%f'
archive_mode = on
archive_timeout = 1 #minute


2. postgresql container 재 실행

3. 아카이브 설정 확인
 - psql 접속
 - show archive_mode;  
    on
 - show archive_command;
  'cp %p /var/lib/pgsql/14/data/backup/%f'
옵션 정상 확인

- references:
- Written by: 이재봉 (jblee6110@gmail.com)
- reporting date: 2023-07-05