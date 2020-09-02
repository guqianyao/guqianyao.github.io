### windows docker 安装记录

1. mysql 

   ```shell
   docker run -itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456! -d mysql
   docker exec -it mysql bash
   mysql -u root -p
   #修改加密规则 mysql 8.0 之后加密方式有修改。需要针对用户修改
   ALTER USER 'root'@'localhost' IDENTIFIED BY '123456!' PASSWORD EXPIRE NEVER;
   #修改密码
   ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456!';
   alter user 'root'@'localhost' identified by '123456!';
   alter user 'root'@'%' identified by '123456!' password expire never;
   alter user 'root'@'%' identified with mysql_native_password by'123456!';
   #刷新数据 
   FLUSH PRIVILEGES;
   # 最建议的方法 新建一个用户
   CREATE USER 'gqy'@'%' IDENTIFIED WITH mysql_native_password BY 'gqy123456!';
   GRANT ALL PRIVILEGES ON *.* TO 'gqy'@'%';
   
   
   navicat 
   修改配置文件
   
   default_authentication_plugin=caching_sha2_password 修改为：
   default_authentication_plugin=mysql_native_password
   ```

2. redis

   ```shell
   docker run -d --name redis -p 6379:6379 redis --requirepass "gqy123456"
   ```

   

3. kibana

4. elasticsearch(集群)

   ```shell
       docker run --name es1 -e "ES_JAVA_OPTS=-Xms128m -Xmx128m" -d -p 9200:9200 -p 9300:9300 elasticsearch:6.6.1
       docker run --name es2 -e "ES_JAVA_OPTS=-Xms128m -Xmx128m" -d -p 19200:19200 -p 19300:19300 elasticsearch:6.6.1
		docker run --name es3 -e "ES_JAVA_OPTS=-Xms128m -Xmx128m" -d -p 29200:29200 -p 29300:29300 elasticsearch:6.6.1
   ```
   
   容器总是启动命令
   
   解决windows 下es 启动不起来得问题
   
   powershell 查看本机wsl  版本
   
   ```shell
    wsl -l -v
   ```
   
   更改 内置 版本 为 Ubuntu-20.04           Running         2
   
   ```shell
   wsl --set -version Ubuntu_xxx 2
   ```
   
   进入wsl  Ubuntu 
   
   ```shell
   sudo passwd # 设置root 密码
   su root
   sysctl -a|grep vm.max_map_count  # 查看参数
   sysctl -w vm.max_map_count=262144 # 修改参数
   ```
   
   在回去启动es  ，访问localhost:9200 成功出现json
   
   ```json
   {
       name: "AVT4jtS",
       cluster_name: "docker-cluster",
       cluster_uuid: "bxGgNlRAS9q5d2KlSbOZkg",
       version: {
           number: "6.6.1",
           build_flavor: "default",
           build_type: "tar",
           build_hash: "1fd8f69",
           build_date: "2019-02-13T17:10:04.160291Z",
           build_snapshot: false,
           lucene_version: "7.6.0",
           minimum_wire_compatibility_version: "5.6.0",
           minimum_index_compatibility_version: "5.0.0"
       },
       tagline: "You Know, for Search"
   }
   ```
   
   安装成功
   
5. nginx

```shell
docker run --name nginx-local -p 8080:80 -d nginx
```

6. 重启dockers 之后自动启动

```
docker update --restart=always 容器ID 
```
