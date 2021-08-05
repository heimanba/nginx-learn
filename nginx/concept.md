## connection
Nginx 的`connection` 就会对tcp连接池的封装，包括连接的`soket`,读写事件等。一个`connection`上面可能会有多个requet的连接。

Nginx中，每个进程都会有一个连接数的最大上限，这个上限与系统对`fd`的限制不一样。在操作系统中通过`ulimit -n`我们可以得到一个进程所能打开的`fd`的最大数,即`nofile`。因为每个`socket`连接会占用一个`ff`,所以这也会限制我们进程的最大连接数，当然也会直接影响我们程序所能支持的最大并发数，`fd`用完后，再创建`socket`时，就会失败。Nginx通过设置`worker_connectons`来设置每个进程最大连接数。如果大于`nofile`,那么实际的最大连接数是`nofile`，NGINX会有警告。

#### worker_connectons
worker_connectons 并不是NGINX所能建立连接的最大值，这个值表示每个worker进程所能建立连接的最大值，所以，一个NGINX能建立的最大连接数，应该`worker_connectons * worker_processes`。也是最大的并发连接数，
> 如果是HTTP作为反向代理，最大并发连接数应该是`worker_connectons * worker_processes/2`,因为作为反向代理服务器，每个并发建立会与客户端的连接和后端服务的连接 ，占用两个连接。

## request
指的是http的请求。包括请求行，请求头，请求体，响应行，响应头，响应体。
