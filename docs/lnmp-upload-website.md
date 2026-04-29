# LNMP 安装后如何上传网站

这篇给纯新手看，只讲最常用的方法。

最简单的理解：

- 你的网页文件放到服务器里
- Nginx 负责把网页显示出来
- 如果是 PHP 网站，还要连数据库

## 先记住这 3 个位置

- 网站文件目录：`/home/web/html/你的域名`
- 网站配置目录：`/home/web/conf.d`
- 数据库配置文件：`/home/web/docker-compose.yml`

比如你的域名是 `abc.com`，那网站目录一般就是：

```bash
/home/web/html/abc.com
```

## 上传前先做什么

先在脚本里把站点创建出来，再传文件。

建议顺序：

1. 先把域名解析到你的服务器 IP
2. 进脚本菜单，先新建站点
3. 再上传网站源码

如果你还没建站，后面上传了文件也可能打不开。

## 怎么把文件传到服务器

你可以用下面任意一种：

- `FinalShell`
- `Xftp`
- `WinSCP`
- `scp` 命令

不会选的话，直接用 `FinalShell` 或 `WinSCP` 就行。

上传时，建议把源码先打包成 `zip`，再传到对应网站目录里。

## 方式一：上传静态网站

静态网站就是只有 `html、css、js、图片` 这些文件，没有 PHP。

### 步骤

1. 在脚本里选择“静态站点”
2. 脚本会帮你建好目录，比如 `/home/web/html/abc.com`
3. 把你的网站压缩包 `xxx.zip` 上传到这个目录
4. 进入服务器，解压压缩包

```bash
cd /home/web/html/abc.com
unzip xxx.zip
```

5. 找到 `index.html` 真正所在的文件夹

比如你解压后变成这样：

```bash
/home/web/html/abc.com/dist/index.html
```

那网站根目录就要填：

```bash
/home/web/html/abc.com/dist/
```

不是填到 `index.html` 这个文件本身，而是填它所在的文件夹。

### 一句话判断有没有传对

只要这个目录里能看到 `index.html`，一般就对了。

## 方式二：上传 PHP 网站

PHP 网站常见于 WordPress、Typecho、论坛、商城这些。

### 步骤

1. 在脚本里选择“PHP动态站点”
2. 脚本会帮你建好目录，比如 `/home/web/html/abc.com`
3. 把网站源码压缩包 `xxx.zip` 上传到这个目录
4. 进入服务器，解压压缩包

```bash
cd /home/web/html/abc.com
unzip xxx.zip
```

5. 找到 `index.php` 真正所在的文件夹

比如你解压后是：

```bash
/home/web/html/abc.com/wordpress/index.php
```

那就填：

```bash
/home/web/html/abc.com/wordpress/
```

6. 按提示选择 PHP 版本

不会选就先用默认新版。  
如果你是老网站，提示报错了，再换 `PHP 7.4` 试试。

## 数据库怎么导入

如果你的网站有数据库，比如 WordPress、博客、论坛、商城，这一步就要做。

如果你是纯静态网站，这一步跳过。

### 最省事的方法

把数据库备份文件传到服务器的 `/home/` 目录。

脚本里这套流程最稳的是：

- 备份文件是 `.sql.gz`
- 放到 `/home/`
- 然后按脚本提示导入

### 手动导入命令

先看数据库 root 密码：

```bash
grep MYSQL_ROOT_PASSWORD /home/web/docker-compose.yml
```

如果你的备份是压缩包，先解压：

```bash
cd /home
gunzip your.sql.gz
```

再导入：

```bash
docker exec -i mysql mysql -u root -p'你的数据库root密码' 你的数据库名 < /home/your.sql
```

导入后，可以看一下表有没有进去：

```bash
docker exec -i mysql mysql -u root -p'你的数据库root密码' -e "USE 你的数据库名; SHOW TABLES;"
```

## 网站程序里的数据库信息怎么填

很多新手卡在这里。

如果网站安装界面让你填数据库信息，按这个来：

- 数据库地址：`mysql`
- 数据库名：你创建的数据库名
- 用户名：你的数据库用户名
- 密码：你的数据库密码

注意：这里数据库地址通常不是 `127.0.0.1`，而是 `mysql`。

## 常见权限问题

如果你上传完网站后，出现下面这些情况：

- 打不开图片
- 网站提示没有写入权限
- 上传插件、主题、附件失败
- 页面一片空白

可以先执行下面这组修复命令：

```bash
docker exec nginx chown -R nginx:nginx /var/www/html
```

如果你装的是 PHP 站，再补一条：

```bash
docker exec php chown -R www-data:www-data /var/www/html
```

如果你用的是 `php74`，再执行：

```bash
docker exec php74 chown -R www-data:www-data /var/www/html
```

这组命令的作用很简单：把网站文件权限重新理顺。

## 上传完后怎么重启

最省事：

```bash
cd /home/web
docker compose restart
```

如果提示没有 `docker compose`，就换这个：

```bash
cd /home/web
docker-compose restart
```

如果你只是改了 Nginx 配置，也可以只重载 Nginx：

```bash
docker exec nginx nginx -s reload
```

## 上传完后怎么检查有没有成功

按这个顺序检查：

1. 浏览器打开你的域名
2. 看首页能不能出来
3. 如果是 PHP 网站，再看后台能不能登录
4. 如果用数据库，再看文章、商品、用户数据在不在

## 最常见的 5 个问题

### 1. 打开域名是空白页

先看你填的根目录对不对。  
很多人把目录填错了，尤其是多套了一层文件夹。

### 2. 域名打不开

先检查：

- 域名有没有解析到服务器 IP
- 80 和 443 端口有没有放行
- Nginx 有没有启动

### 3. PHP 网站报数据库连接失败

先检查：

- 数据库名对不对
- 用户名密码对不对
- 数据库地址是不是填了 `mysql`

### 4. 图片或上传功能异常

大概率是权限问题，先执行上面的权限修复命令。

### 5. 解压后打不开网站

多数是因为源码真正入口不在最外层。  
你要找到 `index.html` 或 `index.php` 真正所在的文件夹。

## 新手最推荐的做法

如果你怕出错，就按这个顺序来：

1. 先在脚本里建站
2. 把源码 `zip` 上传到 `/home/web/html/你的域名`
3. 解压
4. 找到 `index.html` 或 `index.php` 所在目录
5. 有数据库就导入
6. 重启服务
7. 浏览器访问测试

照这个顺序，基本不会乱。
