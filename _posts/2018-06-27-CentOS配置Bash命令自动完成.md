---
layout: post
title: CentOS配置Bash命令自动完成
tags: [Linux]
---

### [](#Bash Auto Completion)Bash Auto Completion

Bash命令自动完成是指我们在bash环境中输入命令和参数的时候，bash可以自动帮我们把命令和相关参数补齐或者给出提示。

### [](#安装)安装

```sh
yum install -y bash-completion bash-completion-extras
source /etc/profile.d/bash_completion.sh
```

### [](#例子)例子

```
$ rpm -qi ph[TAB][TAB]
php         php-cli     php-common  php-devel   php-imap
```

```
$ yum [TAB][TAB]
--assumeyes        --enableplugin     list               search
--cacheonly        --enablerepo       makecache          --setopt
check              --errorlevel       --nogpgcheck       shell
check-update       --exclude          --noplugins        --showduplicates
clean              groupinfo          --obsoletes        --skip-broken
--color            groupinstall       provides           --tolerant
--config           grouplist          --quiet            update
--debuglevel       groupremove        --randomwait       upgrade
deplist            help               reinstall          --verbose
--disableexcludes  --help             --releasever       version
--disableplugin    history            remove             --version
--disablerepo      info               repolist           
distro-sync        install            resolvedep         
downgrade          --installroot      --rpmverbosity 
```

### [](#定制自己的Completion)定制自己的Completion

创建文件/usr/share/bash-completion/completions/foo

```sh
_foo() 
{
    local cur prev opts
    COMPREPLY=()
    cur="${COMP_WORDS[COMP_CWORD]}"
    prev="${COMP_WORDS[COMP_CWORD-1]}"
    opts="--help --verbose --version"

    if [[ ${cur} == -* ]] ; then
        COMPREPLY=( $(compgen -W "${opts}" -- ${cur}) )
        return 0
    fi
}
```

执行如下命令:

```
complete -F _foo foo
```
验证:

```
. /etc/bash_completion.d/foo
foo --[TAB]
--help     --verbose  --version  
```
### [](#参考)参考
*   [https://debian-administration.org/article/316/An_introduction_to_bash_completion_part_1](https://debian-administration.org/article/316/An_introduction_to_bash_completion_part_1)
*   [https://debian-administration.org/article/317/An_introduction_to_bash_completion_part_2](https://debian-administration.org/article/317/An_introduction_to_bash_completion_part_2)
