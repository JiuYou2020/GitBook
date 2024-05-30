---
description: nvm管理npm
---

# nvm in linux

## 安装 NVM

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```

## 加载 NVM

```bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")" [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
```

## 验证 NVM 安装

```bash
nvm --version
```

## 查看可用的 Node.js 版本

```bash
nvm ls-remote
```

## 安装新的 Node.js 版本

```bash
nvm install 14.17.0
```

## 查看已安装的 Node.js 版本

```bash
nvm ls
```

## 切换 Node.js 版本

```bash
nvm use 14.17.0
```

## 设置默认 Node.js 版本

```bash
nvm alias default 14.17.0
```
