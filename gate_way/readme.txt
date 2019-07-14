1. image 构建命令

docker build -t micro/gateway .

2. 调试模式运行

docker run --name flask_gateway -v $PWD/app:/app -p5001:5000 micro/gateway python /app/app.py

我们在容器运行的时候，修改应用程序代码，Flask会检测到更改并重新启动应用程序。



