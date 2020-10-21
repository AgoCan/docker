# docker-compose
官网： https://docs.docker.com/compose/  以下除了pip安装和dockerfile安装依赖切换成国内源以外都是官方摘要，如有需要请到官方进行查阅

推荐学习网址： https://github.com/laradock/laradock 该项目使用的是php的laravel框架，并且比较成熟的方案

## 安装

pip方式安装
```bash
yum install epel-release -y
yum install -y python36 python36-devel  python36-setuptools python python-devel python-setuptools
easy_install-3.6 pip
mkdir -p /root/.pip
echo [global] >> ~/.pip/pip.conf
echo index-url = https://mirrors.aliyun.com/pypi/simple/ >> ~/.pip/pip.conf
echo [install] >> ~/.pip/pip.conf
echo trusted-host=mirrors.aliyun.com >> ~/.pip/pip.conf
python3 -m pip install docker-compose
docker-compose --version
```
直接下载

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

## 快速入门


```bash
# 创建目录
mkdir composetest
cd composetest
# 增加反斜线是为了防止转译
cat > app.py << \EOF
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
EOF
# 增加依赖
echo flask > requirements.txt
echo redis >> requirements.txt

# 编写Dockerfile
# 切换了alpine和pip的为国内源
echo > Dockerfile << EOF
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
RUN \
    sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories && \
    apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt  -i https://pypi.douban.com/simple/
COPY . .
CMD ["flask", "run"]
EOF

# 编写docker-compose.yml

cat > docker-compose.yml << EOF
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
EOF

# 启动
docker-compose up
# docker-compose up -d # 后台运行

# 测试
curl http://localhost:5000

# 查看镜像
docker image ls
```
