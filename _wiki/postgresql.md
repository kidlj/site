---
title: PostgreSQL
---

### Authentication

PostgreSQL 使用 `pg_hba.conf` 文件控制客户端的连接认证。

更改此文件后需要发送 SIGHUP 信号 reload 进程：

    $ pg_ctl reload

举例：允许某个 ip 的 demo 用户连接 demo 数据库，使用 密码：

        TYPE    DATABASE    USER    IP              METHOD
        Host    demo        demo    172.17.8.1/24   md5

从上到下，匹配到一个项就开始进行认证，而后连接或者拒绝连接，不会 fallthrough。

### 初始用户

PostgreSQL 初始化以后，一般会以 `postgres` 用户运行，同时系统内建了该用户为 superuser，连接认证方式为 `local` 和 `peer`(即与操作系统 user 对应)。

    $ sudo su postgres
    $ psql

### 管理员用户

一般会建立一个管理员用户，而不是拥有超级权限的 `postgres` 用户来管理数据库。

建立一个名为 `vagrant` 的用户：

    (postgres) $ psql // as `postgres` user
    => create role vagrant WITH LOGIN CREATEROLE CREATEDB;
    => create database owner vagrant // 需要一个同名的数据库

默认 `pg_hba.conf` 配置了一条允许任何操作系统本地用户 peer 登录的条目：

    local   all      all  <>  peer

如果系统此时用户为 vagrant，则可以直接登录：

    $ (vagrant) psql // as `vagrant` user

### 创建业务用户

    CREATE ROLE name WITH LOGIN CREATEROLE CREATEDB;

### 创建数据库

    $ (vagrant) psql // as `vagrant` user
    vagrant => CREATE ROLE demo;
    vagrant => GRANT demo TO vagrant; // 将 vagrant 加入 demo role 才可以进行下一步操作
    vagrant => CREATE DATABASE demo OWNER demo;

### GRANT PRIVILEGES

ref: https://www.postgresql.org/docs/11/sql-grant.html

> If WITH GRANT OPTION is specified, the recipient of the privilege can in turn grant it to others. Without a grant option, the recipient cannot do that. Grant options cannot be granted to PUBLIC.

> There is no need to grant privileges to the owner of an object (usually the user that created it), as the owner has all privileges by default. (The owner could, however, choose to revoke some of their own privileges for safety.)

> GRANT and REVOKE can also be done by a role that is not the owner of the affected object, but is a member of the role that owns the object, or is a member of a role that holds privileges WITH GRANT OPTION on the object. In this case the privileges will be recorded as having been granted by the role that actually owns the object or holds the privileges WITH GRANT OPTION. For example, if table t1 is owned by role g1, of which role u1 is a member, then u1 can grant privileges on t1 to u2, but those privileges will appear to have been granted directly by g1. Any other member of role g1 could revoke them later.

### 查看信息

列出数据库：

    => \l;

切换数据库：

    => \c dbname;

列出表：

    => \dt;

查看 roles：

    => \du;

### 