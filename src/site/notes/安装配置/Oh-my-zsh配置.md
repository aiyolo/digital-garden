---
{"uid":null,"aliases":null,"tags":["安装配置"],"source":null,"created":"2023-01-22 13:33:08","updated":"2023-03-02 16:22:28","title":"Oh-my-zsh 配置","dg-publish":true,"permalink":"/安装配置/Oh-my-zsh配置/","dgPassFrontmatter":true,"noteIcon":""}
---


# Oh-my-zsh 配置

## Zsh

先安装 zsh，建议把 zsh 设置成默认 shell

## Zinit 安装

使用 zinit 来管理插件，其链接为: [zdharma-continuum/zinit: 🌻 Flexible and fast ZSH plugin manager (github.com)](https://github.com/zdharma-continuum/zinit)

按照下述步骤操作，安装常用插件即可

```
bash -c "$(curl --fail --show-error --silent --location https://raw.githubusercontent.com/zdharma-continuum/zinit/HEAD/scripts/install.sh )"

// reload zsh

zinit self-update

# Plugin history-search-multi-word loaded with investigating.
zinit load zdharma-continuum/history-search-multi-word

# Two regular plugins loaded without investigating.
zinit light zsh-users/zsh-autosuggestions
zinit light zdharma-continuum/fast-syntax-highlighting

# Snippet
zinit snippet https://gist.githubusercontent.com/hightemp/5071909/raw/
```
