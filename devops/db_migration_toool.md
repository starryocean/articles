
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
  1. 设置.env环境变量，主要是数据库链接的URL，需要在.env文件中添加mysql的链接:  
     DATABASE_URL="mysql://user:password@192.168.1.1:3306/Test"    
  2. dbmate命令默认在执行该命令的当前目录下查找.env文件，如果没有，会报"Error: unsupported driver: "

#### dbmate new(n)命令，创建一个迁移文件  
   1. 命令:dbmate new description   
   这个命令默认会在当前目录的db/migrations/目录下创建一个类似与$version_$description这个格式的文件，其中$version是以日期为单位的版本号，
   $description是本地迁移内容的描述.以下是一个具体的样例:  
   2. 样例:dbmate new issue_100_create_a_new_user_table  
   命令成功后会提示"Creating migration: db/migrations/20190530075614_issue_100_add_a_new_user_table.sql"
   3. 解释:文件中有两段，一段是以-- migrate:up开始，表明这部分的脚本是需要此次需要迁移到数据库的脚本，-- migrate:down表明是本地需要回滚的脚本，
   这两个标记很重要，大家按照规范进行书写。  
   4. 一般情况下java项目中migrations的脚本会放在src/main/resources/db/migrations目录下。进入到src/main/resources目录下，将.env文件放到该目录下，并执行dbmate命令。也可以通过-d参数指定migrations的脚本所在目录。
     dbmate new issue_100_create_a_new_user_table 
   
   new issue_100_create_a_new_user_table的脚本内容如下:
   ```
   -- migrate:up
   create table user(
      id INT(10),
      name varchar(100) NOT NULL DEFAULT ""
   )

   -- migrate:down
   drop table user;
   ```
   
#### dbmate create命令，创建一个数据库
   dbmate -s db/schema-$ENV.sql create 
   创建一个数据库，如果已经存在则提示数据库已经存在。执行该命令时，dbmate的用户必须用户数据库创建和删除的权限。  

#### dbmate drop命令，删除一个数据库
    dbmate -s db/schema-$ENV.sql drop 
    谨慎执行该命令。执行该命令时，dbmate的用户必须用户数据库创建和删除的权限。
    
#### dbmate up, 执行迁移目录下的所有的数据库脚本，如果没有数据库则会优先创建数据
    dbmate -s db/schema-$ENV.sql up
    
#### dbmate migrate,对目录下所有的脚本文件进行迁移
    dbmate -s db/schema-$ENV.sql migrate
    
#### dbmate rollback,对最近的迁移脚本进行回滚，这里面需要在迁移文件中手动添加回滚脚本 -- migrate:down
    dbmate -s db/schema-$ENV.sql rollback
    注意:虽然官方文档有说明 -- migrate:up transaction:true 默认值为true，但实际验证事物不生效
    
#### dbmate dump，将schema_migrations的内容写到本地的db/schema.sql文件中
    dbmate -s db/schema-$ENV.sql dump


### dbmate开发数据库迁移过程
   1. 安装dbmate，如上描述;
   2. cd upstream/bees360web/backend/bees360-mapper/src/main/resources,执行dbmate new issue_100_create_a_new_user_table;  
   3. 在新生成的文件中，在"-- migrate:up" 行下添加新增脚本，在 "-- migrate:download"行下添加回滚脚本;  
   4. 执行dbmate -s db/schema-$ENV.sql migrate;   
   5. 将db/migrations下的20190530075614_issue_100_add_a_new_user_table.sql和schema.sql(记录迁移历史记录),并commit到服务器上  
   6. 如果执行失败先dbmate -s db/schema-$ENV.sql rollback,然后手动清除，直至所有脚本都执行成功  
   
### dbmate测试|生产环境迁移
   1. 执行迁移命令前需要备份数据库;
   2. 安装dbmate，如上描述;
   3. 修改upstream/bees360web/backend/bees360-mapper/src/main/resources/.env文件，配置DATABASE_URL;
   4. dbmate -s db/schema-$ENV.sql migrate;
   5. 提交schema-$ENV.sql(记录迁移历史记录)到gitlab服务器上;
   6. 如果执行失败，需要手动处理;
