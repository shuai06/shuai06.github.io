# wpscan更新超时报错




> https://blog.csdn.net/zhang644720213/article/details/92849717

参考自：https://blog.csdn.net/zhang644720213/article/details/92849717




  
先更新一下系统的索引
`updatedb`

查看文件
`dpkg -L wpscan`

下载需要的包（感谢大佬提供）:
`wget https://files.cnblogs.com/files/despotic/wp.zip`


放到本地apache的目录下：
`mv wp.zip /var/www/html/wpscan/`

解压：
`unzip wp.zip`

重启apache
`/etc/init.d/apache2 restart`


编辑配置文件：
`vim /var/lib/gems/2.5.0/gems/wpscan-3.5.4/lib/wpscan/db/updater.rb`


找到：`def remote_file_url`这个函数， 下面的服务器地址改为本地的目录`http://127.0.0.1/wpsaca/#{filename}`


再运行`wpscan --upgrade`就可以了

---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/wpscan%E6%9B%B4%E6%96%B0%E8%B6%85%E6%97%B6%E6%8A%A5%E9%94%99/  

