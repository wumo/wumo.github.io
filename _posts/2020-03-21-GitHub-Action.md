---
title:  GitHub Action
---


# Master Repo和Slave Repo同步

添加如下`push.yml`到`.github/workflows`，这个workflow会在push到`master`分支时触发push到另一个repo的`build`分支：

```yml
name: push

on:
  push:
    branches:
      - master

jobs:
  push:
    if: contains(github.event.head_commit.message, 'build please!')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          ref: master
          fetch-depth: 0
      - name: Install SSH key
        uses: shimataro/ssh-key-action@v2
        with:
          key: ${{ secrets.SSH_KEY }}
          known_hosts: ${{ secrets.KNOWN_HOSTS }}
      - run: |
          git remote add slave git@github.com:***/***.git
          git config --global user.email "***@***"
          git config --global user.name "***"
          git push slave master:build -f
```
- 这个workflow的运行还有一个额外的条件：仅在`commit`消息内包含`"build please!"`字符串时才会运行，这样我们便可以很方便的控制编译频率，节省资源。
- `shimataro/ssh-key-action@v2`是用来安装ssh相关文件到`~/.ssh`文件夹下，以实现之后的git操作。`KNOWN_HOSTS`是你要push的远程仓库的域名信息，可以通过`ssh-keyscan -t rsa ***.com`来查询得到。`SSH_KEY `则是`~/.ssh/id_rsa`文件的内容（以`-----BEGIN RSA PRIVATE KEY-----`开头）。
- 最后是正常的git流程，添加远程repo地址（需要用git协议）>设置用户邮箱>强制push本地`master`分支到远程`build`分支。

<!-- more -->

# Slave Repo编译

添加如下`build.yml`到`.github/workflows`，这个workflow会在push到`build`分支时触发编译并发布tag：
```yml
name: build

on:
  push:
    branches:
      - build


jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    runs-on: ${{ matrix.os }}

    steps:
    - uses: actions/checkout@v2
    - uses: actions/cache@v1
      with:
        path: ~/.gradle/caches
        key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*') }}
        restore-keys: |
          ${{ runner.os }}-gradle-
    - name: Set up JDK 13
      uses: actions/setup-java@v1
      with:
        java-version: 13
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew
    - name: Build with Gradle
      run: ./gradlew shadowJar
    - name: Upload binaries to release
      uses: svenstaro/upload-release-action@v1-release
      if: success()
      with:
        tag: release
        repo_token: ${{ secrets.GITHUB_TOKEN }}
        file: build/libs/*
        file_glob: true
        overwrite: true
```
- 编译脚本主要是针对gradle的，期间还保存了gradle cache以加速编译。
- 发布release tag是使用`svenstaro/upload-release-action@v1-release` action，发布的是`build/libs/*`文件夹下所有文件到名为`release`的tag。

# 定时服务的运行及tag asset文件的下载

通过slave repo的编译发布，我们可以得到最终需要定时运行的程序，但github action并未提供方便的tag asset文件下载action，这里我们使用`github api v3`来实现asset的下载。
```kotlin
fun downloadAsset(
    token: String,
    user: String, repo: String,
    tagName: String, file: String
  ) {
    runBlocking {
      val result = client.get(
        "https://api.github.com/repos/$user/$repo/releases",
        headers = mapOf(
          "Authorization" to "token $token",
          "Accept" to "application/vnd.github.v3.raw"
        )
      ).json()
      result.jsonArray.forEach { tag ->
        if (tag["tag_name"].content != tagName) return@forEach
        tag["assets"].jsonArray.forEach { asset ->
          if (asset["name"].content == file) {
            val id = asset["id"].content
            client.download(
              File(file), "https://api.github.com/repos/$user/$repo/releases/assets/$id",
              headers = mapOf(
                "Accept" to "application/octet-stream",
                "Authorization" to "token $token"
              )
            )
            return@runBlocking
          }
        }
      }
    }
  }
```
代码用kotlin实现，依赖`kotlin coroutine`、`kotlin serialization`和`okhttp`等库，主要流程就是先查询`releases`，然后找到需要的文件，最后下载。
代码通过`shadowJar`集成编译成`github-api.jar`包之后添加到master repo内的`libs`文件夹下。

添加如下`run.yml`到`.github/workflows`，这个workflow会定时在每天0点下载jar包并运行：
```yml
name: run

on:
  schedule:
    # UTC time 比北京时间晚8个小时
    - cron:  '0 16 * * *'

jobs:
  app-run:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-java@v1
        with:
          java-version: 13
      - run: |
          java -jar libs/github-api.jar ${{secrets.slave_token}} slavename***  slaverepo***  tagname***  ***.jar
          java -jar ***.jar
```

# Workflow运行流程

1. 添加一个包含`build please!`的`commit`，并push到master repo的`master`分支。
2. Master repo的`master`分支在接收到这个`commit`后触发`push.yml`工作流，将最新的`commit`push到slave repo的`build`分支。
2. slave repo的`build`分支在接收到push后触发`build.yml`工作流，编译代码，并将编译生成的文件添加到名为`"release"`的`tag`。
3. slave repo会定时从release tag下载待执行程序，并运行。
