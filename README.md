# guangsuqiu.github.io

## 重新构建
1. 下载node.js
2. 安装hexo，在git bash中输入
    ```bash
    npm config set registry https://registry.npm.taobao.org # 将npm源替换为阿里的镜像，安装更快
    npm install hexo-cli -g # 安装Hexo
    rm -rf node_modules && npm install --force
    npm install hexo-deployer-git --save
    npm install hexo-renderer-pug hexo-renderer-stylus
    hexo -v # 返回Hexo版本，确定安装成功
    ```
3. hexo命令
    |序号|命令|功能描述|
    |:---|:---|:-------|
    |1|hexo g|生成静态文件|
    |2|hexo s|在本地启动服务器|
    |3|hexo d|将网站部署到github服务器中|
    |4|hexo clean|清除缓存文件与生成的静态文件|
    |5|hexo new "博客名称"|在 blog/source/_posts/文件夹下生成`博客名称.md`文档|

4. TODO
```
    git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
```
重新根据博客构建仓库，目前存在一些问题 

## github 404 设置
在Settings页面，Pages选项中，选择Branch为release分支的root后，点击save后即可正常显示

## github 修改默认分支
在Settings页面，General选项中，修改Default branch

## 主题目录下为空问题

```
git rm -rf --cached themes/butterfly/
git add themes/butterfly
git commit -m ""
```

## 外链图片加载不出来

在`themes/buutterfly/layout/include/head.pug`文件中添加字段
```
meta(name="referrer" content="no-referrer")
```

## 无法加载文件，因为在此系统上禁止运行脚本问题

将windows设置中的`允许本地PowerShell脚本在不签名的情况下运行`选择应用


# 参考链接：
1. [使用git分支保存hexo博客源码到github](https://www.yangbing.fun/2019/06/29/save-hexo-source-post-with-git-branch/)
2. [Hexo+Next主题搭建个人博客+优化全过程（完整详细版）](https://zhuanlan.zhihu.com/p/618864711)
3. [【个人博客】Hexo+Github搭建个人博客（2023全）](https://www.6young.site/blog/de56ed25.html)
4. [【Git】Git出现 fatal: Pathspec ‘xxx‘ is in submodule ‘xxx‘ 异常 解决方案](https://blog.csdn.net/zzddada/article/details/121930030)
5. [解决hexo引入图床，手机和web不显示图片的问题](https://www.jianshu.com/p/5b58ecce6443)
6. [hexo : 无法加载文件 C:\Users\mxz\AppData\Roaming\npm\hexo.ps1，因为在此系统上禁止运行脚本。](https://blog.csdn.net/weixin_43874301/article/details/111102493)

