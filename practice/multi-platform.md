> ps： 此文档主要就是记录一下我自己从操作步骤

```bash

wget https://golang.org/dl/go1.17.2.linux-amd64.tar.gz
tar xf go1.17.2.linux-amd64.tar.gz -C /usr/local/
export PATH=$PATH:/usr/local/go/bin
export GOPROXY="https://goproxy.io"

mkdir -p /root/core
cd /root/core
cat > main.go <<EOF
package main
import (
 "net/http"
 "runtime"
 "github.com/gin-gonic/gin"
)
var (
 r = gin.Default()
)
func main() {
 r.GET("/", indexHandler)
 r.Run(":9090")
}
func indexHandler(c *gin.Context) {
 var osinfo = map[string]string{
  "arch":    runtime.GOARCH,
  "os":      runtime.GOOS,
  "version": runtime.Version(),
 }
 c.JSON(http.StatusOK, osinfo)
}
EOF

go mod init hankbook.cn/test
go mod tidy

cat > /etc/docker/daemon.json << EOF
{"experimental": true}
EOF


cat > build.sh << \EOF
#!/bin/bash
IMAGE=hankbook/library/test
NOCOLOR='\033[0m'
RED='\033[0;31m'
GREEN='\033[0;32m'
BUILD_ARCH=$(uname -m)
BUILD_OS=$(uname -s)
BUILD_PATH=build/docker/linux
LINUX_ARCH="amd64 arm64 riscv64"
LDFLAGS="-s -w -X cmd/info.BuildArch=${BUILD_ARCH} -X cmd/info.BuildOS=${BUILD_OS}"
for arch in ${LINUX_ARCH}; do
 echo ===================;
 echo ${GREEN}Build binary for ${RED}linux/$$arch${NOCOLOR};
 echo ===================;
 GOARCH=$arch GOOS=linux go build -o ${BUILD_PATH}/$arch/webapp -ldflags="${LDFLAGS}" -v cmd/main.go;
done
EOF

bash build.sh

cat > Dockerfile.slim << EOF
FROM scratch
LABEL authors="Fanjian Kong"
ADD webapp /app/
WORKDIR /app
CMD ["/app/webapp"]
EOF

cat > build_dockerfile.sh << \EOF
#!/bin/bash
IMAGE=hankbook.cn/library/test
NOCOLOR='\033[0m'
RED='\033[0;31m'
GREEN='\033[0;32m'
LINUX_ARCH="amd64 arm64 riscv64"
BUILD_PATH="build/docker/linux"
for arch in ${LINUX_ARCH}; do
 echo =================== ;
 echo ${GREEN}Build docker image for ${RED}linux/$$arch${NOCOLOR} ;
 echo =================== ;
 cp Dockerfile.slim ${BUILD_PATH}/$arch/Dockerfile ;
 docker build -t ${IMAGE}:$arch ${BUILD_PATH}/$arch ;
done
EOF

bash build_dockerfile.sh

# 下面的两个镜像 tag和镜像名都可以改，只要仓库可以拉取即可
docker push hankbook.cn/library/test:arm64
docker push hankbook.cn/library/test:amd64

# 以下两个步骤需要创建harbor仓库， 如果是非证书登录的话  --insecure
# 创建的第一个名称就是镜像名，这里没加版本号，也可以 docker manifest create hankbook.cn/library/test:v1 hankbook.cn/library/test:arm64 hankbook.cn/library/test:amd64
docker manifest create hankbook.cn/library/test hankbook.cn/library/test:arm64 hankbook.cn/library/test:amd64
docker manifest annotate hankbook.cn/library/test hankbook.cn/library/test:amd64 --os linux --arch amd64
docker manifest annotate hankbook.cn/library/test hankbook.cn/library/test:arm64 --os linux --arch arm64 --variant v8

docker manifest inspect intellif.io/library/test

docker manifest push intellif.io/library/test:latest
# 这样仓库里面就有 intellif.io/library/test:latest 根据自己需要进行拉取了
```




## 参考文档： https://cloud.tencent.com/developer/article/1765612
