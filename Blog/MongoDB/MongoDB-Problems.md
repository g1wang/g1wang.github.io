## mongoCPU占用过高问题
### 前提准备：
- 登录admin库
```
mongo -u admin -p 123456 admin
```
- 创建超级权限账户，然后登陆超级账户
```
db.createUser( { user:"rootadmin", pwd:"123456", roles:[{role:"root",db:"admin"}] } );
```

### 正式开始：
- 查看mongodb进程 ps-ef | grep mongo 获取进程id为3267
-  查看进程的线程 top -p 3267，找到目前占用cpu最高的线程id为46265，该线程占用cpu 85.2%
- 查看mongo进程3267的各线程系统调用情况  
- 查看mongo当前有哪些操作
```
db.currentOp(true);
```