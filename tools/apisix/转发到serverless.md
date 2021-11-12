{
  "uri": "/web1/*",
  "name": "web1",
  "methods": [
    "GET",
    "POST",
    "PUT",
    "DELETE",
    "PATCH",
    "HEAD",
    "OPTIONS",
    "CONNECT",
    "TRACE"
  ],
  "host": "www.abc.com",
  "plugins": {
    "proxy-rewrite": {
      "host": "1740298130743624.cn-hangzhou.fc.aliyuncs.com",
      "regex_uri": [
        "^/web1/(.*)",
        "/2016-08-15/proxy/fc-http-service/http/"
      ]
    }
  },
  "upstream": {
    "nodes": {
      "1740298130743624.cn-hangzhou.fc.aliyuncs.com": 1
    },
    "timeout": {
      "connect": 6,
      "send": 6,
      "read": 6
    },
    "type": "roundrobin",
    "scheme": "http",
    "pass_host": "pass",
    "keepalive_pool": {
      "idle_timeout": 60,
      "requests": 1000,
      "size": 320
    }
  },
  "status": 1
}