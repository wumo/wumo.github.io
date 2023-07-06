---
title:  GitHub Action日常用法
---

# 包含本地编译dll的JitPack发布

对使用JNI进行动态链接库编译和链接的kotlin项目而言，其在JitPack上的发布便要麻烦得多，所以我们需要通过github action提前进行动态链接库的编译，并将编译好的动态链接库commit到repo的本身，如此在JitPack上的发布便十分简单，只不过为了不污染`master`分支的开发，编译好的动态链接库应该push到新的`ci`分支，然后创建基于`ci`分支的`release`以在JitPack上发布。

<!--more-->

workflow脚本范例如下：
```yml
name: release

on:
  push:
    branches:
      - master

jobs:
  build-windows:
    runs-on: windows-latest

    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true
      - name: Set up Python
        uses: actions/setup-python@v1
        with:
          python-version: '3.x'
      - name: Install conan
        run: |
          python -m pip install --upgrade pip
          pip install conan --upgrade
          pip install conan_package_tools --upgrade
          conan remote add bincrafters https://api.bintray.com/conan/bincrafters/public-conan
          conan remote add wumo https://api.bintray.com/conan/wumo/public
      - name: Set up JDK
        uses: actions/setup-java@v1
        with:
          java-version: '9'
          java-package: jdk
          architecture: x64
      - name: Build with Gradle
        run: |
          ./gradlew generateJava
          ./gradlew generateJNI
      - name: Upload resources
        uses: actions/upload-artifact@v2
        with:
          name: windows-resources
          path: src/main/resources

  build-linux:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true
      - name: Download resources
        uses: actions/download-artifact@v2
        with:
          name: windows-resources
          path: src/main/resources
      - name: Set up Python
        uses: actions/setup-python@v1
        with:
          python-version: '3.x'
      - name: Install gcc-10
        run: |
          sudo apt install gcc-10 g++-10
          sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 60 --slave /usr/bin/g++ g++ /usr/bin/g++-10
      - name: Install conan
        run: |
          python -m pip install --upgrade pip
          pip install conan --upgrade
          pip install conan_package_tools --upgrade
          conan remote add bincrafters https://api.bintray.com/conan/bincrafters/public-conan
          conan remote add wumo https://api.bintray.com/conan/wumo/public
      - name: Set up JDK
        uses: actions/setup-java@v1
        with:
          java-version: '9'
          java-package: jdk
          architecture: x64
      - name: Build with Gradle
        run: |
          chmod +x gradlew
          ./gradlew generateJava
          ./gradlew generateJNI
      - name: Upload resources
        uses: actions/upload-artifact@v2
        with:
          name: linux-resources
          path: src/main/resources

  commit-release:
    needs: [build-windows, build-linux]
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true
          persist-credentials: true
          fetch-depth: 0
      - name: Download linux-resources
        uses: actions/download-artifact@v2
        with:
          name: linux-resources
          path: src/main/resources
      - name: Download resources
        uses: actions/download-artifact@v2
        with:
          name: windows-resources
          path: src/main/resources
      - name: Commit files
        run: |
          git config --local user.email "wumo@github.com"
          git config --local user.name "GitHub Action"
          git add src/main/resources
          git commit -m "Compiled library resources" -a
      - name: Push changes to ci branch
        uses: ad-m/github-push-action@v0.6.0
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          branch: ci
          force: true
```

* `build-windows` 配置编译环境，将编译好的dll上传至`windows-resources`
* `build-linux`配置编译环境，将编译好的so上传至`linux-resources`，
* `commit-release`在`build-windows`和`build-linux`完成后，下载相应的`artifact`，然后commit、release到`ci`分支。

## 从gradle获取项目version并创建对应的release

下面的命令通过运行gradle命令获取version属性存入环境变量`PROJECT_VERSION`中，再使用`actions/create-release`脚本进行发布。

```yml
- name: Get release version
  run: |
    chmod +x gradlew
    echo ::set-env name=PROJECT_VERSION::$(./gradlew properties -q | grep "version:" | awk '{print $2}')
- name: Release
  uses: actions/create-release@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    tag_name: ${{ env.PROJECT_VERSION }}
    release_name: ${{ env.PROJECT_VERSION }}
    commitish: ci
```

## gralde使用axion-release-plugin自动版本升级release

`build.gradle.kts`配置：

```kotlin
// build.gradle.kts
plugins{
  id("pl.allegro.tech.build.axion-release") version "1.12.0"
}
scmVersion {
  // 符合jitpack对release的要求
  tag.apply {
    prefix = ""
  }
  checks.apply {
    uncommittedChanges = false
    aheadOfRemote = false
    snapshotDependencies = false
  }
}
version = scmVersion.version
```

`github workflow`配置：

```yaml
- name: Get release version
  run: |
    chmod +x gradlew
    ./gradlew createRelease
    echo ::set-env name=PROJECT_VERSION::$(./gradlew properties -q | grep "version:" | awk '{print $2}')
- name: Release
  uses: actions/create-release@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    tag_name: ${{ env.PROJECT_VERSION }}
    release_name: ${{ env.PROJECT_VERSION }}
    commitish: ci
```

# 多个repo之间的github action同步

## Master Repo和Slave Repo同步

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

<!--more-->

## Slave Repo编译

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

## 定时服务的运行及tag asset文件的下载

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

## Workflow运行流程

1. 添加一个包含`build please!`的`commit`，并push到master repo的`master`分支。
2. Master repo的`master`分支在接收到这个`commit`后触发`push.yml`工作流，将最新的`commit`push到slave repo的`build`分支。
2. slave repo的`build`分支在接收到push后触发`build.yml`工作流，编译代码，并将编译生成的文件添加到名为`"release"`的`tag`。
3. slave repo会定时从release tag下载待执行程序，并运行。
