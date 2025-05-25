---
title: 使用GitHub Actions部署Hexo
date: 2025-05-25 16:38:47
categories:
- 开发工具
tags:
- Hexo
- GitHub Actions
---

最开始一直使用[Hexo](https://hexo.io/zh-cn/)提供的[一键部署](https://hexo.io/zh-cn/docs/one-command-deployment)插件[hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)将站点静态资源文件推送至github仓库的main分支，这种部署方式本身没什么问题，但每次修改代码仓库，都需要手动部署，源代码也只能push到另外一个分支上。Github提供了[GitHub Actions](https://docs.github.com/zh/actions/writing-workflows/quickstart)，代码CI后，可以自定义工作流，将源代码生成的站点静态资源直接部署至[GitHub Pages](https://pages.github.com/)，节省了手动部署的烦恼，本文简单记录如何从`Deploy from a branch`迁移至`GitHub Actions`部署，以便后续查阅。

## 代码库设置

> Settings > Pages > Build and deployment > Source > GitHub Actions。

设置Actions权限：

> Settings > Actions > Genenral > Actions permissions > Workflow permissions

选中 `Read and write permissions`。

<!--more-->

## 分支变更

### 增加.gitignore文件

之前main分支作为站点的部署分支，存储的是`hexo generate`生成在public目录下的文件。使用GitHub Actions后，使用main分支存储源码文件，需要将.gitignore中添加以下文件，防止提交包依赖文件和编译后的文件（如node_modules、public等)。

```shell
$ cat .gitignore 
Thumbs.db
db.json
node_modules/
public/
.deploy*/
```

## 增加工作流配置文件

在代码库中增加[.github/worflows/deploy.yml](https://github.com/leeyzero/leeyzero.github.io/blob/main/.github/workflows/deploy.yml)文件，解释一下核心步骤：

1、监听main分支，推送到github后自动触发deploy.yml中定义的工作流

```shell
on:
  push:
    branches:
      - main  # 监听 main 分支的推送
```

2、设置相关权限，该设置可以直接设置全局，也可以针对不同action进行设置，比如build、deploy。

```shell
permissions:
    contents: read
    pages: write
    id-token: write
```

3、拉取依赖代码，如果使用了子模块，开启[submodules](https://github.com/marketplace/actions/checkout-submodules#alternatives)

```shell
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        submodules: true  # 递归拉取子模块
```

4、初始化node.js环境

```shell
  - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20' # 根据你的需求调整版本
        cache: 'npm'  # 启用 npm 缓存加速
```

5、安装npm依赖

```shell
- name: Install dependencies
      run: |
        npm install hexo-cli -g
        npm install
```

6、替换密钥

在使用一些插件时，需要在配置中设置密钥，但这些密钥也不想提交至代码库中，这个时候可以使用GitHub Actions提供的密钥/变更配置功能，可以在执行工作流的过程中使用预配置的密钥或变更进行替换。比如我在_config.yml文件中需要配置一个密钥，可以将密钥托管至GitHub Actions中。

Settings > Environments > github-pages > Environment secrets，添加密钥，比如我添加：BAIDU_URL_SUBMIT_TOKEN

我本地使用_config.yml（包含真实密钥，用于开发使用)，将_config.yml.template文件提交至代码库，_config.yml.template文件中密钥使用占位符，在编译的工作流中对文件名进行重命中，对密钥进行替换，比如：

```shell
...
baidu_url_submit:
  count: 1 
  host:  leeyzero.github.io # 在百度站长平台中注册的域名
  token: __BAIDU_URL_SUBMIT_TOKEN__ # 请注意这是您的秘钥，所以请不要把博客源代码发布在公众仓库里!
  path: baidu_urls.txt # 文本文档的地址，新链接会保存在此文本文档里
```

然后在工作流中定义密钥替换，使用sed从环境变更中取出真实密钥进行替换。

```shell
- name: Inject Secrets into Config
      run: |
        # 检查模板文件是否存在
        if [ ! -f "_config.yml.template" ]; then
          echo "❌ Error: _config.yml.template missing!"
          exit 1
        fi
        cp _config.yml.template _config.yml
        sed -i "s|__BAIDU_URL_SUBMIT_TOKEN__|${{ secrets.BAIDU_URL_SUBMIT_TOKEN }}|g" _config.yml
```

7、生成站点的静态文件

```shell
- name: Generate static files
      run: |
        hexo clean
        hexo generate
        # 验证生成的 public 目录
        if [ ! -f "public/index.html" ]; then
          echo "❌ Error: Static files not generated!"
          exit 1
        fi
```

8、将站点文件打包，上传至临时目录

```shell
  - name: Upload Pages Artifact
      uses: actions/upload-pages-artifact@v3
      with:
        path: ./public
```

9、部署GitHub Pages

```shell
    steps:
    - name: Deploy to GitHub Pages
      id: deployment
      uses: actions/deploy-pages@v4
```

## 提交变更代码

如果之前main分支作为站点发布分支，需要删除所有静态资源文件。

```shell
git add .
git commit -m "deploy hexo by github pages"
git push origin/main
```

## 部署测试

部署成功后，打开站点: `https://<替换为你的站点>.github.io`看是否能够正常访问。如果部署失败，可以在代码库的Actions中查看具体失败日志。

自定义域名，需要设置：

> Settings > Pages > GitHub Pages > Custom domain

在source目录下新增CNAME文件，CNAME中内容为你的域名。hexo在生成静态文件时，会将CNAME 拷贝至public目录下。

## 参考资料

- [one-command-deployment](https://hexo.io/zh-cn/docs/one-command-deployment)
- [GitHub Pages](https://pages.github.com/)
- [GitHub Actions](https://docs.github.com/zh/actions/writing-workflows/quickstart)