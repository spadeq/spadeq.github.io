---
layout: post
title: VScode React Typescript 开发环境基础配置
date: 2019-08-12 21:56:00
categories:
  - Development
tags:
  - Visual Studio Code
  - React
  - Typescript
---

本文记录了用 Visual Studio Code 开发 React（使用 Typescript 语言）的一些基本配置。

## 插件

需要安装的插件包括：

- TSLint
- ESLint
- Prettier

## Typescript 环境

建议不要用 Windows，可以使用 vscode 结合 WSL Remote 开发的方式。

首先安装 NodeJS：

```bash
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
sudo apt-get install -y nodejs
```

然后修改使用国内源：

```bash
npm config set registry https://registry.npm.taobao.org
npm config get registry
```

安装全局 npm 包：

```bash
sudo npm install -g typescript
sudo npm install -g tslint
```

## 创建 React 工程

可以用 `npx` 命令，这样就不用安装 `create-react-app` 包了。

```bash
npx create-react-app <APPNAME> --typescript
```

这一步如果报类似于 mkdir 的错误，请用 `chown` 命令将 `~/.npm` 目录的所有权改为自己。

安装必要的开发依赖：

```bash
cd <APPNAME>
npm install tslint tslint-react tslint-config-prettier --save-dev
```

这时就可以用 vscode 打开目录了。

## 添加 lint 规则

新增 `tslint.json` 文件，内容如下：

```json
{
  "extends": ["tslint:recommended", "tslint-react", "tslintconfig-prettier"],
  "rules": {
    "ordered-imports": false,
    "object-literal-sort-keys": false,
    "no-debugger": false,
    "no-console": false
  },
  "linterOptions": {
    "exclude": [
      "config/**/*.js",
      "node_modules/**/*.ts",
      "coverage/lcov-report/*.js"
    ]
  }
}
```

输入完成后可以按 `Alt + Shift + F` 进行格式化，vscode 会提示有多个格式化工具，让我们选择一个，这时候就选 Prettier。

建议在 vscode 的设置中，将 Format On Save 选中。
