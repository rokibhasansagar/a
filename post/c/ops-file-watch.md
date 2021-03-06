---
title: Whtch file
date: 2020-04-19
private: true
---
# Whtch file
除了watchdog 还有命令行工具
- inotify (linux)
- fswatch (osx)

# fswatch

    brew install fswatch
    fswatch ~/path/to/watch | xargs -n1 -I{} echo changed_file is:

help: 

    $ fswatch -h
    fswatch [OPTION] ... path ...
    -L, --follow-links    Follow symbolic links.
    -o, --one-per-batch   Print a single message with the number of change events.
    -r, --recursive       Recurse subdirectories.(default)
    -x, --event-flags     Print the event flags.


# inotify
## inotify 说明
inotify是Linux核心子系统之一，做为文件系统的附加功能，它可监控文件系统并将异动通知应用程序。本系统的出现取代了旧有Linux核心里，拥有类似功能之dnotify模块。

inotify的原始开发者为John McCutchan、罗伯特·拉姆与Amy Griffis。于Linux核心2.6.13发布时(2005年六月十八日)，被正式纳入Linux核心[1]。尽管如此，它仍可通过补丁的方式与2.6.12甚至更早期的Linux核心集成。

inotify的主要应用于桌面搜索软件，像：Beagle，得以针对有变动的文件重新索引，而不必没有效率地每隔几分钟就要扫描整个文件系统。相较于主动轮询文件系统，通过操作系统主动告知文件异动的方式，让Beagle等软件甚至可以在文件更动后一秒内更新索引。

此外，诸如：更新目录查看、重新加载设置文件、追踪变更、备份、同步甚至上传...等许多自动化作业流程，都可因而受惠。


## inotifywait
    inotifywait -e MODIFY -r -m /build |
    while read path action file; do
        ext=${file: -3}
        if [[ "$ext" == ".go" ]]; then
        echo File changed: $file
        go build main.go
        ./main
        fi
    done

其中：

    -r          recursive
    -e MODIFY 
    -m path     specify the path that we monitor 