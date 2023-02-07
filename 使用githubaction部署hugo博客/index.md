# 使用GithubAction部署Hugo博客


<!--more-->

因为节省时间（懒），所以从网上找了这个方法



### 1.新建github仓库

**需要新建两个github仓库，我这里是：**

- MyHugoSource: 存储源码
- shuai06.github.io:存放发布的静态网页资源





### 2.ssh秘钥

```bash
ssh-keygen -t rsa - -C "$(git config user.email)"
# 注意：这次不要直接回车，以免覆盖之前生成的。如果已经有秘钥文件，则需要换一个路径，避免覆盖，我这里为/users/xps/.ssh_action/id_rsa
 
```

- 生成的公钥（我这里是`~/.ssh_action/id_rsa.pub `）放到page仓库(shuai06.github.io)中：`cat ~/.ssh_action/id_rsa.pub`之后复制，来到page仓库点击 `Setting - Deploy keys - Add deploy key`，名称随意，粘贴进去刚生成的公钥，勾选 `Allow write access` ,保存。
- 生成的私钥（我这里是`~/.ssh_action/id_rsa `）放到page仓库(MyHugoSource)中：`cat ~/.ssh_action/id_rsa`之后，来到源码仓库，点击`setting->Actions secrets and variables-->actions-->new resposity secert`，变量名称这里设置为`ACTIONS_DEPLOY_KEY`(这里和后面配置文件保持统一即可)， 添加刚刚生成的私钥(id_rsa)，





### 3.配置github action

> 注意yaml文件的格式规范

在源码根目录下新建 `/.github/workflows/github-actions-demo.yml`，内容如下：

可能需要修改的地方为：deploy_key、external_repository、cname

```yaml
name: Auto Deploy hugo
on: [push]
jobs:
  Explore-GitHub-Actions:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository code
        uses: actions/checkout@v2
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: latest
          extended: true
      - name: Build
        run: hugo
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }} # 这里是上面设置的变量，一致即可
          external_repository: shuai06/shuai06.github.io  # page仓库的路径
          publish_branch: main	# 分支
          publish_dir: ./public	# 发布内容的目录
          cname: geoer.cn	# cname，设置为自己的
          commit_message: ${{ github.event.head_commit.message }}
```



如果有 submodule,需要加上两三行：

```yaml
      - name: Check out repository code
        uses: actions/checkout@v2
        with:
          submodules: recursive  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod
      - name: Setup Hugo

```





### 4.整体流程

yam文件需要push上去



这样，一般我写文章的步骤概述为：

> 在源码文件根目录操作的

```bash
hugo new posts/xxx.md	# 新建文章

hugo -D		# 构建

git add .
git commit -m "(update)提交xxxx"
git push -u origin main

```

过一会，访问page就可以了

​	



> 个人测试成功部署，具体情况请根据实际问题具体分析







### 参考文章

https://blog.csdn.net/weixin_43031092/article/details/119900223

https://blog.csdn.net/weixin_41263449/article/details/107584336

https://shenhonglei.blog.csdn.net/article/details/124261204

---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E4%BD%BF%E7%94%A8githubaction%E9%83%A8%E7%BD%B2hugo%E5%8D%9A%E5%AE%A2/  

