# tar 增量备份

## tar 常用属性

- -z, --gzip：使用gzip工具（解）压缩，后缀一般为.gz
- -c, --create：tar打包，后缀一般为.tar
- -f, --file=：后面立刻接打包或压缩后得到的文件名
- -x, --extract：解包命令，与-c对应
- -p：保留备份数据的原本权限和属性
- -g：后接增量备份的快照文件
- -C：指定解压缩的目录
- --exclude：排除不打包的目录或文件，支持正则匹配

## 备份

```
   tar -g /tmp/fupin.snap -zcpf - default | split -b 1024M - backup/fupin$(date -I).tar.gz_
```

```
  tar -g /tmp/fupin.snap -zcpf backup/fupin$(date -I).tar.gz default
```

> `-g`选项可以理解备份时给目录文件做一个快照，记录权限和属性等信息，第一次备份时/tmp/fupin.snap不存在，会新建一个并做完全备份。当目录下的文件有修改后，再次执行第一条备份命令（记得修改后面的档案文件名），会自动根据-g指定的快照文件，增量备份修改过的文件，包括权限和属性，没有动过的文件不会重复备份。

> 另外需要注意上面的恢复，是"保留恢复"，即存在相同文件名的文件会被覆盖，而原目录下已存在（但备份档案里没有）的，会依然保留。所以如果你想完全恢复到与备份文件一模一样，需要清空原目录。如果有增量备份档案，则还需要使用同样的方式分别解压这些档案，而且要注意顺序。

## 恢复

- 完全恢复

  ```
  cat fupin2016-12-05.tar.gz_* | tar -zxpf - -C ./
  ```

- 增量恢复

  ```
  tar –zxpf fupin2016-12-06.tar.gz -C ./
  ```

## 注意

> 使用tar无论是备份数据还是文件系统，需要考虑是在原系统上恢复还是另一个新的系统上恢复。

> - tar备份极度依赖于文件的atime属性，
> - 文件所属用户是根据用户ID来确定的，异机恢复需要考虑相同用户拥有相同USERID
> - 备份和恢复的过程尽量不要运行其他进程，可能会导致数据不一致
> - 软硬连接文件可以正常恢复

## 参考链接

- [Linux下最容易的增量备份,tar增量备份](http://www.linuxidc.com/Linux/2011-01/31793.htm)
- [tar命令高级用法----备份数据](https://segmentfault.com/a/1190000002421723)
- [linux 下tar 打包分割文件和解压文件方法一点通](http://blog.csdn.net/chaihuasong/article/details/39652373)
