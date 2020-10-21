# entrypoint.sh案例
entrypoint.sh 可以通过环境变量的方式进行区分环境，例如：

`${HOST_NAME:-localhost}`的意思是，如果变量名HOST_NAME没有值，就使用localhost这个默认值

`exec "$@"` 执行所有参数

```bash
#!/bin/sh
cat > /etc/nginx/conf.d/default << EOF
server {
  server_name ${HOST_NAME:-localhost};
  listen ${IP:-0.0.0.0}:${PORT:80};
  root ${ROOT_DIR:-/var/www/html};
}
exec "$@"
EOF
```

dockerfile编写

```
FROM nginx
ENV ...
RUN ...
COPY entrypoint.sh /bin/entrypoint.sh
RUN chmod +x /bin/entrypoint.sh
ENTRYPOINT ["/bin/entrypoint.sh"]
CMD ["nginx", "-g", "daemon off;"]
```

```
docker build -e HOST_NAME=hank997.com -e IP=0.0.0.0 -e PORT=8888 -t test:v1 .
```
