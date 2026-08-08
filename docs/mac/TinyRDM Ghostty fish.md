 安装了redis管理工具`Tiny RDM`, 启动时会提示没有`pyhton3`， 然后就弹窗提示安装`Install Command Line Developer Tools`， 为阻止弹窗需要覆盖系统默认的`pyhton3`（顺便也覆盖`git`）

在`/usr/local/bin`中创建文件`git`或`pyhton`， 分别写入：
```
#!/bin/zsh
echo "Fake Git Command"
```
和
```
#!/bin/zsh
echo "Fake Python Command"
```
然后执行`sudo ln -s python python3`

---

覆盖`git`后, 因为`Ghostty`配置了：

```
command = /usr/local/bin/fish
shell-integration = fish
```

打开会提示一堆错误，原因是无法识别虚假的`git`命令， 解决办法是在`~/.config/fish/functions`中创建文件`fish_prompt.fish`并写入：

```
function fish_prompt --description 'Write out the prompt'
    set -l last_pipestatus $pipestatus
    set -lx __fish_last_status $status # Export for __fish_print_pipestatus.
    set -l normal (set_color --reset)

    # Color the prompt differently when we're root
    set -l color_cwd $fish_color_cwd
    set -l suffix '>'
    if functions -q fish_is_root_user; and fish_is_root_user
        if set -q fish_color_cwd_root
            set color_cwd $fish_color_cwd_root
        end
        set suffix '#'
    end

    # Write pipestatus
    # If the status was carried over (if no command is issued or if `set` leaves the status untouched), don't bold it.
    set -l bold_flag --bold
    set -q __fish_prompt_status_generation; or set -g __fish_prompt_status_generation $status_generation
    if test $__fish_prompt_status_generation = $status_generation
        set bold_flag
    end
    set __fish_prompt_status_generation $status_generation
    set -l status_color (set_color $fish_color_status)
    set -l statusb_color (set_color $bold_flag $fish_color_status)
    set -l prompt_status (__fish_print_pipestatus "[" "]" "|" "$status_color" "$statusb_color" $last_pipestatus)

    echo -n -s (prompt_login)' ' (set_color $color_cwd) (prompt_pwd) $normal " "$prompt_status $suffix " "
ends
```

重点是： `echo -n -s (prompt_login)' ' (set_color $color_cwd) (prompt_pwd) $normal " "$prompt_status $suffix " "`

原本是： `echo -n -s (prompt_login)' ' (set_color $color_cwd) (prompt_pwd) $normal (fish_vcs_prompt) $normal " "$prompt_status $suffix " "`

其中的`$normal (fish_vcs_prompt)`会导致报错

PS: 原来的内容可通过命令`functions fish_prompt`查看， 前提是删除`~/.config/fish/functions/fish_prompt.fish`和`/usr/local/bin/git`

另外， 还可配置一些Ghosttty的快捷键：

```
whls@gfourdx ~/.c/f/functions> pwd
/Users/whls/.config/fish/functions
whls@gfourdx ~/.c/f/functions> ls
di.fish           dipf.fish         dpa.fish          drmf.fish         drmi.fish         fish_prompt.fish  ll.fish
whls@gfourdx ~/.c/f/functions> cat di.fish
function di
    docker images
end
whls@gfourdx ~/.c/f/functions> cat dipf.fish
function dipf
    docker image prune -f
end
whls@gfourdx ~/.c/f/functions> cat dpa.fish
function dpa
    docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"
end
whls@gfourdx ~/.c/f/functions> cat drmf.fish
function drmf
    docker rm -f $argv
end
whls@gfourdx ~/.c/f/functions> cat drmi.fish
function drmi
    docker rmi $argv
end
whls@gfourdx ~/.c/f/functions> cat ll.fish
function ll
    ls -ahlrt $argv
end
```