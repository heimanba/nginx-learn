## 用处
`drizzle-nginx-module`用于直接和`MYSQL`和`Drizzle`对话的上游模块
```
upstream cluster {
    # simple round-robin
    drizzle_server 127.0.0.1:3306 dbname=test
        password=some_pass user=monty protocol=mysql;
    drizzle_server 127.0.0.1:1234 dbname=test2
        password=pass user=bob protocol=drizzle;
}

upstream backend {
    drizzle_server 127.0.0.1:3306 dbname=test
        password=some_pass user=monty protocol=mysql;
}

server {
    location /mysql {
        set $my_sql 'select * from cats';
        drizzle_query $my_sql;

        drizzle_pass backend;

        drizzle_connect_timeout    500ms; # default 60s
        drizzle_send_query_timeout 2s;    # default 60s
        drizzle_recv_cols_timeout  1s;    # default 60s
        drizzle_recv_rows_timeout  1s;    # default 60s
    }

    ...

    # for connection pool monitoring
    location /mysql-pool-status {
        allow 127.0.0.1;
        deny all;

        drizzle_status;
    }
}
```