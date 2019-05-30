
## dbmate迁移工具项目实践

### Ubuntu 下载安装
```
    sudo curl -fsSL -o /usr/local/bin/dbmate https://github.com/amacneil/dbmate/releases/download/v1.6.0/dbmate-linux-amd64
    sudo chmod +x /usr/local/bin/dbmate
```

### dbmate 命令和参数介绍
  通过dbmate -h 命令查看 dbmate 有9个命令和6个全局的参数，下面一一介绍
  ```
    dbmate -h
NAME:
   dbmate - A lightweight, framework-independent database migration tool.

USAGE:
   dbmate [global options] command [command options] [arguments...]

VERSION:
   1.6.0

COMMANDS:
     new, n          Generate a new migration file
     up              Create database (if necessary) and migrate to the latest version
     create          Create database
     drop            Drop database (if it exists)
     migrate         Migrate to the latest version
     rollback, down  Rollback the most recent migration
     dump            Write the database schema to disk
     wait            Wait for the database to become available
     help, h         Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --env value, -e value             specify an environment variable containing the database URL (default: "DATABASE_URL")
   --migrations-dir value, -d value  specify the directory containing migration files (default: "./db/migrations")
   --schema-file value, -s value     specify the schema file location (default: "./db/schema.sql")
   --no-dump-schema                  don't update the schema file on migrate/rollback
   --help, -h                        show help
   --version, -v                     print the version

  ```
  
#### dbmate命令执行的前提条件
  1. 设置.env环境变量，主要是数据库链接的URL，如mysql的.env文件中需要添加以下配置:  
     DATABASE_URL="mysql://user:password@192.168.1.1:3306/Test"    
  2. dbmate命令默认在执行该命令的当前目录下查找.env文件，如果没有，会报"Error: unsupported driver: "
  3. 一般情况下可将.env文件放到当前用户的根目录下，如~/dbmate.env,这时需要在dbmate 命令行中指定.env文件的位置，如下所示:
     dbmate -e ~/dbmate.env

#### dbmate new(n)命令，创建一个迁移文件  
   1. 命令:dbmate new description   
   这个命令默认会在当前目录的db/migrations/目录下创建一个类似与$version_$description这个格式的文件，其中$version是以日期为单位的版本号，
   $description是本地迁移内容的描述.以下是一个具体的样例:  
   2. 样例:dbmate new issue_100_create_a_new_user_table  
   命令成功后会提示"Creating migration: db/migrations/20190530075614_issue_100_add_a_new_user_table.sql"
   3. 解释:文件中有两段，一段是以-- migrate:up开始，表明这部分的脚本是需要此次需要迁移到数据库的脚本，-- migrate:down表明是本地需要回滚的脚本，
   这两个标记很重要，大家按照规范进行书写。  
   ```
   -- migrate:up
   create table user(
      id INT(10),
      name varchar(100) NOT NULL DEFAULT ""
   )

   -- migrate:down
   drop table user;
   ```
   
   
