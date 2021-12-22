---
title: Github SSH免密登录和Token配置
date: 2021-7-20
Tags:
  - github
categories:
  - Tools
cover: https://pngimg.com/uploads/github/github_PNG15.png
---

### 新建一个新的SSH KEY

```bash
ssh-keygen -t rsa -b 4096 -C "email@163.com"
```

- 回车；yes;回车；回车，最后在 `~/.ssh/下面生成密钥`

![image-20210817215558486](https://tva1.sinaimg.cn/large/008i3skNly1gtk4p0ejpaj310q09en05.jpg)

- id_rsa 密钥
- id_rsa.pub 公钥

![image-20210817215728996](https://tva1.sinaimg.cn/large/008i3skNly1gtk4qjctsrj61tw0sqgqn02.jpg)

- 加入生成的公钥。

### 修改仓库地址为 ssh 类型

- 修改命令

```bash
git remote origin set-url [url]
```

- 先删除，后添加

```bash
git remote rm origin
git remote add origin [url]
```

- 修改config文件

------

### GitHub Token

- 

![image-20210817220300375](https://tva1.sinaimg.cn/large/008i3skNly1gtk4wa33jbj60cu02ot8l02.jpg)

- ![image-20210817220322065](https://tva1.sinaimg.cn/large/008i3skNly1gtk4wnp8hyj61m20dagoh02.jpg)

- 在这里申请自己的token

[开始创建自己的Token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token)

### MACOS修改钥匙串



[更多信息查看]([Updating credentials from the macOS Keychain - GitHub Docs](https://docs.github.com/en/get-started/getting-started-with-git/updating-credentials-from-the-macos-keychain))

![image-20210817220603725](https://tva1.sinaimg.cn/large/008i3skNly1gtk4zgwtggj60zi0oimza02.jpg)

- 钥匙串
- 找见`github.com`
- 修改或者删除对应项

### 命令删除凭证

```bash
git credential-osxkeychain erase
host=github.com
protocol=https
> [Press Return]
```

