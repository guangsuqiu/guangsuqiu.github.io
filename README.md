# guangsuqiu.github.io

## 重新构建
1. 下载node.js
2. 安装hexo
    ```
    npm config set registry https://registry.npm.taobao.org # 将npm源替换为阿里的镜像，安装更快
    npm install hexo-cli -g # 安装Hexo
    hexo -v # 返回Hexo版本，确定安装成功
    ```
3. 初始化博客，并安装工具
    ```
    hexo init
    npm install hexo-deployer-git --save
    git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
    npm install hexo-renderer-pug hexo-renderer-stylus
    ```

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

参考链接：
1. [使用git分支保存hexo博客源码到github](https://www.yangbing.fun/2019/06/29/save-hexo-source-post-with-git-branch/)
2. [Hexo+Next主题搭建个人博客+优化全过程（完整详细版）](https://zhuanlan.zhihu.com/p/618864711)
3. [【个人博客】Hexo+Github搭建个人博客（2023全）](https://www.6young.site/blog/de56ed25.html)
4. [【Git】Git出现 fatal: Pathspec ‘xxx‘ is in submodule ‘xxx‘ 异常 解决方案](https://blog.csdn.net/zzddada/article/details/121930030)