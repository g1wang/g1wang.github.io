# MongoDB-Primer

## 备份
### MongoDB DUMP

```
mongodump -h 127.0.0.1 -u admin -p password -d db_family --gzip -o /mnt/mongo-disk/bak-mongodb/bak-mongo-data/fullbak-220218
压缩 
tar -zcvf xx.tar.gz srcdir
传输
scp user@ip:src target
```
