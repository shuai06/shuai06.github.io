# virtualenv



  
注意：
```
# 腾讯云的源有时候不行，可以更换源
pip3 install virtualenv -i https://pypi.doubanio.com/simple/ --user
```

  
### 搭建虚拟环境
1. 先安装 `virtualenv`, 再安装 `virtualenvwrapper`
```
pip install virtaulenv
pip install virtualenvwrapper
```
2. 设置虚拟的环境变量   
    * . `先找到 virtualenvwrapper.sh` 文件
    ```sh
    find / -name "virtualenvwrapper.sh"
    ```
    *  写入 `.bashrc` 文件
    ```
    export WORKON_HOME=$HOME/.virtualenvs  
    source 这里写起前面找出来的文件
    ```

3. 基本命令
    ```sh
    # 创建虚拟环境
    mkvirtualenv pyenv
    # 删除虚拟机环境
    rmvirtualenv pyenv
    # 进入虚拟环境
    workon pyenv
    # 推出虚拟环境
    deactivate
    # 创建虚拟环境的是时候，指定 Python 版本
    mkvirtualenv 
    ```
    
4. 如果报如下错误：
    ```
    ERROR: virtualenvwrapper could not find virtualenv in your path
    ```
    因为是被安装在默认的Python目录下，添加软链接即可
    ```
    ln -s /usr/bin/python36/bin/virtualenv /usr/local/bin/virtualenv
    ```
    
    
    
    
#### 批处理安装库：

##### requirements.txt  

pip freeze 冻结出所有包与版本号

导出：`pip freeze > requirements.txt `  

再安装: `pip install -r requirements.txt`




虚拟环境：

windows激活虚拟环境 ：pycharm自动激活
```
cd 虚拟环境目录venv/Scripts/
active
```

linux激活虚拟环境 ：`source ./venv/bin/active`




---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/virtualenv%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83/  

