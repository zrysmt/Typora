## 1. 地址
全量迁移 
http://gitlab.alibaba-inc.com/amp/any-full 
预检查和结构迁移 
http://gitlab.alibaba-inc.com/amp/dts_ddl 
增量迁移和数据同步 
http://gitlab.alibaba-inc.com/amp/amp_full_inc 
前端 
http://gitlab.alibaba-inc.com/amp/amp_web 
调度和OPEN API 
http://gitlab.alibaba-inc.com/amp/dts_portal



## 2. 发布

【前情提要】
1、测试是在预发环境，由于网络不通，不能跟预发的机器互传文件，需要通过中转机或参考oss跨域互传文件（https://www.atatech.org/articles/64801）
2、机器登不上的都需要先申请权限（https://sa.alibaba-inc.com/device/account/user.jsx?file=user.jsx）
3、10.101.235.81 这是一台日常和线上都能连的机器，但是只有私网，要中转才能到预发上，oxs的机器 和这台中转机是可以直接通的，可以互拷文件，预发的机器资源在numen上可以查看（http://num.alibaba-inc.com/public/#/dts/resource）
4、登陆跳板机的密码：域密码+阿里郎令牌 （不用输+号），其余都是域密码
【开始部署】
1、登陆阿里云跳板机：ssh meirong.dmr@login1.eu13.cloud.alibaba.org
2、登陆中转机机器 ： ssh 10.101.235.81
3、本地jar包上传到中转机：scp dts-agent-4.9.50.186-dmr.test.jar meirong.dmr@10.101.235.81:/tmp
4、连内网，中转机jar包上传到预发,由于权限不允许，先放到/tmp目录下，在拷贝到/u02/dts/
5、scp dts-agent-4.9.50.186-dmr.test.jar meirong.dmr@11.196.22.19:/tmp
6、sudo cp dts-agent-4.9.50.186-dmr.test.jar /u02/dts/
7、建立软链：sudo ln -snf dts-agent-4.9.50.186-dmr.test.jar node.jar

## 3. dts-portal 项目
pom更改
```xml
- <scope>provided</scope>
+ <!--<scope>provided</scope>-->
+ <scope>compile</scope>
```