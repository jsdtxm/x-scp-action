# 🚀 GitHub Actions 的 SCP 传输工具

简体中文 | [English](./README.md)

[GitHub Action](https://github.com/features/actions) 通过 SCP 复制文件和产物。

[![Actions Status](https://github.com/jsdtxm/x-scp-action/workflows/scp%20files/badge.svg)](https://github.com/jsdtxm/x-scp-action/actions)

本项目基于 [Golang](https://go.dev) 和 [drone-ssh](https://github.com/appleboy/drone-ssh) 构建 🚀

## 使用方式

通过 SSH 复制文件与产物：

```yaml
name: scp files
on: [push]
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: copy file via ssh password
        uses: jsdtxm/x-scp-action@v1.2.0
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          password: ${{ secrets.PASSWORD }}
          port: ${{ secrets.PORT }}
          source: "tests/a.txt,tests/b.txt"
          target: your_server_target_folder_path
```
## 输入参数
详见 [action.yml](./action.yml) 文件
| 参数                | 描述                                                                                                     | 默认值        |
| ------------------- | ------------------------------------------------------------------------------------------------------- | ------------ |
| host                | 远程主机地址                                                                                            | -            |
| port                | 远程端口号                                                                                              | `22`         |
| username            | 用户名                                                                                                  | -            |
| password            | 密码                                                                                                    | -            |
| passphrase          | 用于私钥加密的密码短语                                                                                  | -            |
| protocol            | 使用的IP协议，可选值：`tcp`, `tcp4`, `tcp6`                                                             | `tcp`        |
| fingerprint         | 主机公钥的SHA256指纹，默认跳过验证                                                                      | -            |
| timeout             | SSH连接超时时间                                                                                         | `30s`        |
| command_timeout     | SCP命令超时时间                                                                                         | `10m`        |
| key                 | SSH私钥内容（如 ~/.ssh/id_rsa 的原始内容）                                                              | -            |
| key_path            | SSH私钥路径                                                                                             | -            |
| target              | 服务器目标路径（必须是目录，**必填**）                                                                  | -            |
| source              | 待传输文件列表（**必填**）                                                                              | -            |
| rm                  | 传输前删除目标文件夹                                                                                    | `false`      |
| strip_components    | 移除指定层级的路径前缀                                                                                  | -            |
| overwrite           | 使用tar的`--overwrite`标志，覆盖已存在文件                                                              | -            |
| tar_tmp_path        | 目标主机上tar文件的临时路径                                                                             | -            |
| tar_exec            | 目标主机上tar可执行文件路径                                                                             | `tar`        |
| tar_dereference     | 使用tar的`--dereference`标志，跟踪符号链接                                                              | -            |
| use_insecure_cipher | 启用不安全加密算法（参见[#15](https://github.com/jsdtxm/x-scp-action/issues/15)）                       | -            |

SSH代理设置：
| 参数                      | 描述                                                                                                     | 默认值        |
| ------------------------- | ------------------------------------------------------------------------------------------------------- | ------------ |
| proxy_host                | 代理主机地址                                                                                            | -            |
| proxy_port                | 代理端口号                                                                                              | `22`         |
| proxy_username            | 代理用户名                                                                                              | -            |
| proxy_password            | 代理密码                                                                                                | -            |
| proxy_protocol            | 使用的IP协议，可选值：`tcp`, `tcp4`, `tcp6`                                                             | `tcp`        |
| proxy_passphrase          | 用于私钥加密的密码短语                                                                                  | -            |
| proxy_timeout             | 代理连接超时时间                                                                                        | `30s`        |
| proxy_key                 | 代理SSH私钥内容                                                                                         | -            |
| proxy_key_path            | 代理SSH私钥路径                                                                                         | -            |
| proxy_fingerprint         | 代理主机公钥的SHA256指纹                                                                                | -            |
| proxy_use_insecure_cipher | 启用不安全加密算法（参见[#15](https://github.com/jsdtxm/x-scp-action/issues/15)）                       | -            |

## 配置SSH密钥

创建和使用SSH密钥时请遵循以下最佳实践。建议在本地机器而非远程机器上生成密钥。
生成RSA密钥对：

```bash
# RSA算法
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
# ed25519算法
ssh-keygen -t ed25519 -a 200 -C "your_email@example.com"
```

将公钥添加到授权密钥文件（详见[授权密钥说明](https://www.ssh.com/ssh/authorized_keys/)）：

```bash
# RSA算法
cat .ssh/id_rsa.pub | ssh b@B 'cat >> .ssh/authorized_keys'
# ed25519算法
cat .ssh/id_ed25519.pub | ssh b@B 'cat >> .ssh/authorized_keys'
```

复制私钥内容到GitHub Secrets：

```bash
# RSA算法
clip < ~/.ssh/id_rsa
# ed25519算法
clip < ~/.ssh/id_ed25519
```

关于[免密码SSH登录](http://www.linuxproblem.org/art_9.html)的更多信息。

**注意**：根据SSH版本不同，您可能需要：
- 将公钥放入`.ssh/authorized_keys2`
- 设置`.ssh`目录权限为700
- 设置`.ssh/authorized_keys2`文件权限为640

### OpenSSH用户注意事项

若遇到以下错误：

```bash
ssh: handshake failed: ssh: unable to authenticate, attempted methods [none publickey]
```

请确保您的密钥算法受支持。在Ubuntu 20.04及以上版本中，需显式允许ssh-rsa算法。在OpenSSH配置文件（`/etc/ssh/sshd_config`或`/etc/ssh/sshd_config.d/`下的文件）中添加：

```bash
CASignatureAlgorithms +ssh-rsa
```

或者使用默认支持的ed25519算法：

```bash
ssh-keygen -t ed25519 -a 200 -C "your_email@example.com"
```

## 使用示例

通过密码认证传输文件：

```yaml
- name: copy file via ssh password
  uses: jsdtxm/x-scp-action@v1.2.0
  with:
    host: example.com
    username: foo
    password: bar
    port: 22
    source: "tests/a.txt,tests/b.txt"
    target: your_server_target_folder_path
```

使用环境变量：

```yaml
- name: copy file via ssh password
  uses: jsdtxm/x-scp-action@v1.2.0
  with:
    host: ${{ env.HOST }}
    username: ${{ env.USERNAME }}
    password: ${{ secrets.PASSWORD }}
    port: ${{ env.PORT }}
    source: "tests/a.txt,tests/b.txt"
    target: ${{ env.TARGET_PATH }}
```

通过密钥认证传输文件：

```yaml
- name: copy file via ssh key
  uses: jsdtxm/x-scp-action@v1.2.0
  with:
    host: ${{ secrets.HOST }}
    username: ${{ secrets.USERNAME }}
    port: ${{ secrets.PORT }}
    key: ${{ secrets.KEY }}
    source: "tests/a.txt,tests/b.txt"
    target: your_server_target_folder_path
```

忽略列表配置示例：

```yaml
- name: copy file via ssh key
  uses: jsdtxm/x-scp-action@v1.2.0
  with:
    host: ${{ secrets.HOST }}
    username: ${{ secrets.USERNAME }}
    port: ${{ secrets.PORT }}
    key: ${{ secrets.KEY }}
    source: "tests/*.txt,!tests/a.txt"
    target: your_server_target_folder_path
```

多服务器配置示例：

```diff
  uses: jsdtxm/x-scp-action@v1.2.0
  with:
-   host: "example.com"
+   host: "foo.com,bar.com"
    username: foo
    password: bar
    port: 22
    source: "tests/a.txt,tests/b.txt"
    target: your_server_target_folder_path
```

排除特定文件示例：

```yaml
  uses: jsdtxm/x-scp-action@v1.2.0
  with:
    host: "example.com"
    username: foo
    password: bar
    port: 22
-   source: "tests/*.txt"
+   source: "tests/*.txt,!tests/a.txt,!tests/b.txt"
    target: your_server_target_folder_path
```

上传构建产物到远程服务器：

```yaml
deploy:
  name: deploy artifact
  runs-on: ubuntu-latest
  steps:
    - name: checkout
      uses: actions/checkout@v4
    - run: echo hello > world.txt
    - uses: actions/upload-artifact@v4
      with:
        name: my-artifact
        path: world.txt
    - uses: actions/download-artifact@v4
      with:
        name: my-artifact
        path: distfiles
    - name: copy file to server
      uses: jsdtxm/x-scp-action@v1.2.0
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.KEY }}
        port: ${{ secrets.PORT }}
        source: distfiles/*
        target: your_server_target_folder_path
```

移除指定层级的路径前缀：

```yaml
- name: remove the specified number of leading path elements
  uses: jsdtxm/x-scp-action@v1.2.0
  with:
    host: ${{ secrets.HOST }}
    username: ${{ secrets.USERNAME }}
    key: ${{ secrets.KEY }}
    port: ${{ secrets.PORT }}
    source: "tests/a.txt,tests/b.txt"
    target: your_server_target_folder_path
    strip_components: 1
```

原目录结构：

```sh
foobar
  └── tests
    ├── a.txt
    └── b.txt
```

新目录结构：

```sh
foobar
  ├── a.txt
  └── b.txt
```

仅传输变更文件：

```yaml
changes:
  name: test changed-files
  runs-on: ubuntu-latest
  steps:
    - name: checkout
      uses: actions/checkout@v4
    - name: Get changed files
      id: changed-files
      uses: tj-actions/changed-files@v35
      with:
        since_last_remote_commit: true
        separator: ","
    - name: copy file to server
      uses: jsdtxm/x-scp-action@v1.2.0
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.KEY }}
        port: ${{ secrets.PORT }}
        source: ${{ steps.changed-files.outputs.all_changed_files }}
        target: your_server_target_folder_path
```

私钥密码保护（密码短语用于加密私钥，使密钥文件本身对攻击者无效）：

```diff
  - name: ssh key with passphrase
    uses: jsdtxm/x-scp-action@v1.2.0
    with:
      host: ${{ secrets.HOST }}
      username: ${{ secrets.USERNAME }}
      key: ${{ secrets.SSH2 }}
+     passphrase: ${{ secrets.PASSPHRASE }}
      port: ${{ secrets.PORT }}
      source: "tests/a.txt,tests/b.txt"
      target: your_server_target_folder_path
```

从Linux运行器传输文件到Windows服务器时需注意：
1. 安装Git for Windows
2. 通过PowerShell命令将默认OpenSSH shell改为git bash
3. 在YAML文件中设置`tar_dereference`和`rm`为`true`
4. 避免通过变量传递端口号
5. 将目标路径转换为Unix格式：`/c/path/to/target/`
修改默认shell的PowerShell命令：

```powershell
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "$env:Programfiles\Git\bin\bash.exe" -PropertyType String -Force
```

目标路径转换示例：

```diff
  - name: Copy to Windows
      uses: jsdtxm/x-scp-action@v1.2.0
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        port: 22
        source: 'your_source_path'
-       target: 'C:\path\to\target'
+       target: '/c/path/to/target/'
+       tar_dereference: true
+       rm: true
```