# 发布 Android 库到 Maven 中心仓库

本项目使用插件 [com.vanniktech.maven.publish](https://vanniktech.github.io/gradle-maven-publish-plugin/central/) 完成发布。

## 详细步骤

### 1. 申请 sonatype 账号

1. 申请 [sonatype](https://central.sonatype.com/) 账号，建议使用 github 授权登录，可以直接通过验证；
2. 通过之后，在 `sonatype` 后台可以得到一个自动分配的 `Namespace` ，对应就是 `groupId` ，在个人中心 `View Account -> Generate User Token` 创建发布用的 用户名和密码，这个密码对应后续配置文件中的：

        mavenCentralUsername=username
        mavenCentralPassword=the_password

### 2. 生成 GPG 密钥

1. 安装 [GPG](https://gnupg.org/download/index.html)，安装完成后，执行 `gpg --version` 可以查看版本号；
2. 生成 GPG 密钥，运行 `gpg --full-generate-key` 生成密钥，按照提示输入姓名、邮箱、密码等信息；
3. 查看 GPG 密钥，运行 `gpg --list-secret-keys --keyid-format LONG` 可以查看生成的密钥，记录下 `sec` 开头的密钥 ID，后续会用到；

#### 3. 上传公钥到公共服务器

以下提到的 `你的密钥ID` 指代上一步密钥 ID 的后 8 位字符串，本工程以 `https://keys.openpgp.org` 为例，实际上还有其他的服务器可选。

> 提示：国内建议直接使用第二种手动上传。

##### 第一种：命令行上传

1. 运行 `gpg --keyserver https://keys.openpgp.org --send-keys [你的密钥ID]` 上传公钥，这个极有可能失败，如果失败建议手动上传；
2. 验证公钥是否上传成功，运行 `gpg --keyserver https://keys.openpgp.org --recv-keys [你的密钥ID]` 可以查看上传的公钥，这个也可能失败，而且即便实际上传成功，也可能出现其他错误，建议手动上传；

##### 第二种：手动上传

1. 导出本地公钥：`gpg --export -a [你的密钥ID] > xxxxxx.asc`，得到用来上传的公钥文件 `xxxxxx.asc`；
2. 访问 https://keys.openpgp.org 并上传 `xxxxxx.asc` 文件；
3. 上传是否成功直接可以在页面上看到提示；

### 4. 配置要发布的库
1. 生成 RingFile 文件，运行 `gpg --export-secret-key -o secring.gpg [你创建密钥的时候输入的email]`，得到 `secring.gpg` 文件，这个文件会用来签名；
2. 配置要发布的库 `build.gradle`，具体可以参考 [HelloWorld/build.gradle](./HelloWorld/build.gradle) 的配置或者 `com.vanniktech.maven.publish` 插件的文档；
3. 配置打包需要用到的签名信息和发布需要用到的账号密码：参考 [gradle.properties.template](./gradle.properties.template) 配置；

### 5. 发布到 Maven 中心仓库

1. 发布：在项目目录下执行 `./gradlew publishToMavenCentral` 或者 `./gradlew <module_name>:publishToMavenCentral` 即可发布到 Maven 中心仓库；
2. 发布成功后，在 `sonatype` 后台的 `Deployments` 页面可以看到待发布的库，如果状态是 `VALIDATED`, 则点击 `Publish` 可以发布到 `Maven Central` 中心仓库；

> 当前插件还支持 `./gradlew publishAndReleaseToMavenCentral` 这个命令，会自动发布到 Maven 中心仓库并发布到 `Maven Central`，不过还是小心为好，确认无误之后再发布。

### 6. 验证是否成功
1. 等待一段时间（不确定，建议在后台网页刷新查看），即可在 Maven 中心仓库中看到发布的库，其他应用就可以使用 `implementation "<groupId>:<artifactId>:<version>"` 的方式正常引用了。

## 友情提示

在发布到 `Maven Central` 之前，建议先运行 `./gradlew publishToMavenLocal` 或者 `./gradlew <module_name>:publishToMavenLocal` 先发布到本地仓库，确认无误之后再发布到远程仓库。 
