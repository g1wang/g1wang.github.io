  

示例：healthcheck-service

说明：svn路径有中文的，粘贴到地址栏再考出来，否则会找不到路径

0）新建  healthcheck-service 文件夹

cd healthcheck-service

1）获取用户信息

svn log https://192.168.70.5/svn/SoftWareCode/JAVA/java%E5%9F%BA%E7%A1%80%E5%B9%B3%E5%8F%B0/base-framework -q | awk -F '|' '/^r/ {sub("^ ", "", $2); sub(" $", "", $2); print $2" = "$2" <"$2">"}' | sort -u > users.txt

2）git-svn

git svn clone https://192.168.70.5/svn/SoftWareCode/JAVA/java%E5%9F%BA%E7%A1%80%E5%B9%B3%E5%8F%B0/base-framework -T trunk -b branches -t tags --authors-file=./users.txt --no-metadata

3）tags

cd healthcheck-service

cp -Rf .git/refs/remotes/origin/tags/* .git/refs/tags/

rm -Rf .git/refs/remotes/origin/tags

4）branch

cp -Rf .git/refs/remotes/origin/* .git/refs/heads/

rm -Rf .git/refs/remotes/origin/*

#删除 trunk分支

git branch -d trunk

  
5）映射gitlab源信息

git remote add origin http://192.168.86.91/iot-server/healthcheck-service.git
  
6）上传

#分支上传

git push origin --all

# 标签上传

git push origin --tags