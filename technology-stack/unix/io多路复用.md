# IO多路复用

[思维导图](/mind.html?path=/technology-stack/unix/IO多路复用)

IO多路复用（I/O Multiplexing）是一种用于处理多个I/O事件的技术。
它允许单个线程或进程监视多个I/O流（如套接字、文件描述符等），并在有数据可读或可写时进行相应的操作，而不需要阻塞等待每个I/O操作完成。
在传统的阻塞式I/O中，每个I/O操作都会阻塞当前线程或进程，直到操作完成或出现错误。
这会导致系统资源的浪费，尤其在需要同时处理多个I/O操作的情况下。
IO多路复用通过引入一个事件循环，允许一个线程同时监视多个I/O流的状态，从而在任何I/O流准备就绪时进行操作。

## [SELECT](https://man7.org/linux/man-pages/man2/select.2.html)

`select()` allows a program to monitor multiple file descriptors,
waiting until one or more of the file descriptors become "ready"
for some class of I/O operation (e.g., input possible). A file
descriptor is considered ready if it is possible to perform a
corresponding I/O operation (e.g., read(2), or a sufficiently
small write(2)) without blocking.

1. 准备文件描述符集合: 在使用 select 之前，需要将要监视的文件描述符加入到需要监视的集合中，分为读、写和异常三个集合。
2. 调用 select 函数: 在程序中调用 select 函数，传入监视的文件描述符集合以及一个超时时间。
   select 函数会将当前线程阻塞，等待监视的文件描述符之一就绪或超时。
3. 等待事件就绪: 内核会定期检查集合中的文件描述符，如果有文件描述符就绪（有数据可读、可写或有异常），则将就绪的文件描述符从集合中挑选出来，以便后续处理。
4. 返回就绪文件描述符: select 函数返回时，会告诉你哪些文件描述符已经就绪，可以进行相应的读写操作。
5. 处理就绪文件描述符: 对于返回的就绪文件描述符，程序可以遍历集合，找到就绪的文件描述符，并进行相应的操作，比如读取数据或写入数据。
6. 循环调用 select: 通常情况下，select 函数会被放在一个循环中，不断地等待和处理就绪的文件描述符，从而实现多路复用

- 一次最多处理1024个文件描述符
- 无法知道哪些文件描述符就绪，只能遍历所有文件描述符
- 在返回时，每个文件描述符集都被修改，以指示哪些文件描述符当前”就绪“。因此，如果在循环中使用select，则必须在每次调用之前重新初始化集合

   ```
    #include <stdio.h>
    #include <stdlib.h>
    #include <sys/select.h>
   
    int
    main(void)
    {
        int             retval;
        fd_set          rfds;
        struct timeval  tv;
   
        /* Watch stdin (fd 0) to see when it has input. */
   
        FD_ZERO(&rfds);
        FD_SET(0, &rfds);
   
        /* Wait up to five seconds. */
   
        tv.tv_sec = 5;
        tv.tv_usec = 0;
   
        retval = select(1, &rfds, NULL, NULL, &tv);
        /* Don't rely on the value of tv now! */
   
        if (retval == -1)
            perror("select()");
        else if (retval)
            printf("Data is available now.\n");
            /* FD_ISSET(0, &rfds) will be true. */
        else
            printf("No data within five seconds.\n");
   
        exit(EXIT_SUCCESS);
    }
   ```

## [POLL](https://man7.org/linux/man-pages/man2/poll.2.html)

`poll()` performs a similar task to select(2): it waits for one of
a set of file descriptors to become ready to perform I/O.

1. 准备文件描述符数组: 类似于 select，在使用 poll 之前，需要创建一个用于存放要监视的文件描述符的数组，并为每个文件描述符分配一个结构体，用于指定感兴趣的事件类型和状态。
2. 调用 poll 函数: 在程序中调用 poll 函数，传入文件描述符数组和数组中元素的数量。poll 函数会将当前线程阻塞，等待监视的文件描述符之一就绪或超时。
3. 等待事件就绪: 内核会定期检查文件描述符数组中的每个元素，如果有文件描述符就绪，就将其相关信息填充到结构体中，以便后续处理。
4. 返回就绪文件描述符信息: 当 poll 函数返回时，程序可以遍历文件描述符数组，找到就绪的文件描述符，并获取其状态信息，包括可以进行的操作（读、写、异常等）。
5. 处理就绪文件描述符: 对于返回的就绪文件描述符，程序可以根据其状态信息进行相应的操作，如读取数据、写入数据等。
6. 循环调用 poll: 类似于 select，通常情况下，poll 函数会被放在一个循环中，以便持续等待和处理就绪的文件描述符。

- 无文件描述符数量限制
- 无法知道哪些文件描述符就绪，只能遍历所有文件描述符
- 不需要每次都重新设置文件描述符集合

    ```
    /* poll_input.c
      Licensed under GNU General Public License v2 or later.
    */
    #include <fcntl.h>
    #include <poll.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                           } while (0)
    int
    main(int argc, char *argv[])
    {
       int            ready;
       char           buf[10];
       nfds_t         num_open_fds, nfds;
       ssize_t        s;
       struct pollfd  *pfds;
       if (argc < 2) {
          fprintf(stderr, "Usage: %s file...\n", argv[0]);
          exit(EXIT_FAILURE);
       }
       num_open_fds = nfds = argc - 1;
       pfds = calloc(nfds, sizeof(struct pollfd));
       if (pfds == NULL)
           errExit("malloc");
       /* Open each file on command line, and add it to 'pfds' array. */
       for (nfds_t j = 0; j < nfds; j++) {
           pfds[j].fd = open(argv[j + 1], O_RDONLY);
           if (pfds[j].fd == -1)
               errExit("open");
           printf("Opened \"%s\" on fd %d\n", argv[j + 1], pfds[j].fd);
           pfds[j].events = POLLIN;
       }
       /* Keep calling poll() as long as at least one file descriptor is
          open. */
       while (num_open_fds > 0) {
           printf("About to poll()\n");
           ready = poll(pfds, nfds, -1);
           if (ready == -1)
               errExit("poll");
           printf("Ready: %d\n", ready);
           /* Deal with array returned by poll(). */
           for (nfds_t j = 0; j < nfds; j++) {
               if (pfds[j].revents != 0) {
                   printf("  fd=%d; events: %s%s%s\n", pfds[j].fd,
                          (pfds[j].revents & POLLIN)  ? "POLLIN "  : "",
                          (pfds[j].revents & POLLHUP) ? "POLLHUP " : "",
                          (pfds[j].revents & POLLERR) ? "POLLERR " : "");
                   if (pfds[j].revents & POLLIN) {
                       s = read(pfds[j].fd, buf, sizeof(buf));
                       if (s == -1)
                           errExit("read");
                       printf("    read %zd bytes: %.*s\n",
                              s, (int) s, buf);
                   } else {                /* POLLERR | POLLHUP */
                       printf("    closing fd %d\n", pfds[j].fd);
                       if (close(pfds[j].fd) == -1)
                           errExit("close");
                       num_open_fds--;
                   }
               }
           }
       }
       printf("All file descriptors closed; bye\n");
       exit(EXIT_SUCCESS);
    }
    ```

## [EPOLL](https://man7.org/linux/man-pages/man7/epoll.7.html)

The epoll API performs a similar task to poll(2): monitoring
multiple file descriptors to see if I/O is possible on any of
them. The epoll API can be used either as an edge-triggered or a
level-triggered interface and scales well to large numbers of
watched file descriptors.

The central concept of the epoll API is the epoll instance, an
in-kernel data structure which, from a user-space perspective,
can be considered as a container for two lists:

The interest list (sometimes also called the epoll set): the
set of file descriptors that the process has registered an
interest in monitoring.

The ready list: the set of file descriptors that are "ready"
for I/O. The ready list is a subset of (or, more precisely, a
set of references to) the file descriptors in the interest
list. The ready list is dynamically populated by the kernel
as a result of I/O activity on those file descriptors.

1. 创建 epoll 实例: 在使用 epoll 之前，需要创建一个 epoll 实例，通过调用 epoll_create 函数来完成。
   这个实例在内核中维护了一个事件表，用于存储需要监视的文件描述符及其相关信息。
2. 注册文件描述符: 将需要监视的文件描述符添加到 epoll 实例中，通过调用 epoll_ctl 函数来完成。
   在注册时，可以指定感兴趣的事件类型（如可读、可写）和相关数据。
3. 调用 epoll_wait 函数: 在程序中调用 epoll_wait 函数，传入 epoll 实例和一个用于存放就绪事件的数组。
   epoll_wait 函数会将当前线程阻塞，等待监视的文件描述符之一就绪。
4. 等待事件就绪: 内核会定期检查 epoll 实例中的事件表，如果有文件描述符就绪，就将其相关信息填充到就绪事件数组中，以便后续处理。
5. 返回就绪事件信息: 当 epoll_wait 函数返回时，程序可以遍历就绪事件数组，找到就绪的文件描述符及其相关信息，如可读、可写等。
6. 处理就绪文件描述符: 对于返回的就绪文件描述符，程序可以根据其相关信息进行相应的操作，如读取数据、写入数据等。
7. 循环调用 epoll_wait: 通常情况下，epoll_wait 函数会被放在一个循环中，以便持续等待和处理就绪的文件描述符。

- 无文件描述符数量限制
- 用了事件驱动的方式，只关注就绪的事件，避免了对所有文件描述符的轮询
- 支持边缘触发模式，只在文件描述符状态发生变化时通知应用程序，避免了重复通知

    ```
    #define MAX_EVENTS 10
    struct epoll_event ev, events[MAX_EVENTS];
    int listen_sock, conn_sock, nfds, epollfd;
    /* Code to set up listening socket, 'listen_sock',
      (socket(), bind(), listen()) omitted. */
    epollfd = epoll_create1(0);
    if (epollfd == -1) {
       perror("epoll_create1");
       exit(EXIT_FAILURE);
    }
    ev.events = EPOLLIN;
    ev.data.fd = listen_sock;
    if (epoll_ctl(epollfd, EPOLL_CTL_ADD, listen_sock, &ev) == -1) {
       perror("epoll_ctl: listen_sock");
       exit(EXIT_FAILURE);
    }
    for (;;) {
       nfds = epoll_wait(epollfd, events, MAX_EVENTS, -1);
       if (nfds == -1) {
           perror("epoll_wait");
           exit(EXIT_FAILURE);
       }
       for (n = 0; n < nfds; ++n) {
           if (events[n].data.fd == listen_sock) {
               conn_sock = accept(listen_sock,
                                  (struct sockaddr *) &addr, &addrlen);
               if (conn_sock == -1) {
                   perror("accept");
                   exit(EXIT_FAILURE);
               }
               setnonblocking(conn_sock);
               ev.events = EPOLLIN | EPOLLET;
               ev.data.fd = conn_sock;
               if (epoll_ctl(epollfd, EPOLL_CTL_ADD, conn_sock,
                           &ev) == -1) {
                   perror("epoll_ctl: conn_sock");
                   exit(EXIT_FAILURE);
               }
           } else {
               do_use_fd(events[n].data.fd);
           }
       }
    }
    ```

## 参考资料

- [ChatGPT](https://chat.openai.com/)
- [解析 Golang 网络 IO 模型之 EPOLL](https://mp.weixin.qq.com/s/xt0Elppc_OaDFnTI_tW3hg)