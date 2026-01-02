title: ARTS_第5周
author: ming
date: 2020-08-03 10:56:00
tags:
---

## Algorithm

## Review

## Tip
1. libev是什么，怎么用，什么场景下用?
* libev是一个事件驱动库
* 任何场景下都可用，著名的ss就是基于这个库写的
* 以io事件为例，使用方法：
```c
int main(int argc, char const *argv[])
{
    init();
    int listenfd = create_and_bind();

    struct ev_loop *loop = ev_default_loop(0);
    struct ev_io w_accept;
    ev_io_init(&w_accept, accept_cb, listenfd, EV_READ);
    ev_io_start(loop, &w_accept);
    ev_run(loop, 0);

    return 0;
}
```

```
void accept_cb(struct ev_loop *loop, struct ev_io *watcher, int revents) {
    int clientfd;
    struct sockaddr_in clientaddr;
    socklen_t len = sizeof(clientaddr);
    clientfd = Accept(watcher->fd, (struct sockaddr *)&clientaddr, &len);
    print_accept_info(clientfd, &clientaddr);

    struct ev_io *w_client = (struct ev_io*) malloc (sizeof(struct ev_io));
    ev_io_init(w_client, client_cb, clientfd, EV_READ);
    ev_io_start(loop, w_client);

    clients[clientfd] = clientfd;
}

void client_cb(struct ev_loop *loop, struct ev_io *watcher, int revents) {
    int n = handle_client(watcher->fd);    
    if (n == 0) {
        int fd = watcher->fd;
        ev_io_stop(loop, watcher);
        Close(fd);
        free(watcher);
        clients[fd] = -1;
        printf("client=%d, client exit...\n", fd);
    }
}
```

2. gcc 链接第三方库，第三方库的名字从哪来?
```
gcc -v -o ev_server file_server.c ../demo/wrap.c -lev
```
* -lev表示静态链接libev.a
* 库路径: /usr/lib/x86_64-linux-gnu/libev.a
* 另外，gcc -v 可用打印详细信息, 可用看到库搜索路径：
```
LIBRARY_PATH=/usr/lib/gcc/x86_64-linux-gnu/9/:/usr/lib/gcc/x86_64-linux-gnu/9/../../../x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/9/../../../../lib/:/lib/x86_64-linux-gnu/:/lib/../lib/:/usr/lib/x86_64-linux-gnu/:/usr/lib/../lib/:/usr/lib/gcc/x86_64-linux-gnu/9/../../../:/lib/:/usr/lib/
```

3. 段错误的core文件，怎么分析, 出错的位置显示了一个内存地址？

4. tcpdump怎么用?

5. nc怎么用

6. find命令，搜索文件， 默认递归搜索
```
find / -type f -name 'libev*'
```

## Share