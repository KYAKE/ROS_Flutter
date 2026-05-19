# ROS Flutter GUI App

基于 C/S 架构的 ROS 上位机项目：

- `backend/`：机器人侧 C++ 后端，连接 ROS1 / ROS2，提供 HTTP、WebSocket、地图与配置接口。
- `app/`：Flutter 客户端 / Web 前端。
- `protocol/`：前后端共用的 protobuf 协议定义。

## 1. 首次配置 Flutter

如果本机还没有配置 Flutter，先完成下面步骤；已经能执行 `flutter --version` 的可以跳过。

```bash
sudo apt-get update
sudo apt-get install -y git curl unzip xz-utils zip libglu1-mesa

mkdir -p ~/development
git clone https://github.com/flutter/flutter.git -b stable ~/development/flutter

echo 'export PATH="$HOME/development/flutter/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

flutter config --enable-web
flutter doctor
```



## 2. 安装构建依赖

Ubuntu + ROS2 Humble 示例：

```bash
sudo apt-get install -y \
  build-essential cmake pkg-config git curl unzip xz-utils zip \
  protobuf-compiler libprotobuf-dev libc-ares-dev \
  libsdl2-dev libsdl2-image-dev libssl-dev zlib1g-dev uuid-dev \
  libboost-filesystem-dev libopencv-dev

dart pub global activate protoc_plugin
echo 'export PATH="$HOME/.pub-cache/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

source /opt/ros/humble/setup.bash
```

如果使用 ROS1 或其他 ROS2 发行版，请改成对应的 ROS 环境：

```bash
source /opt/ros/<distro>/setup.bash
```

## 3. 一键构建

在仓库根目录执行：

```bash
./build.sh
```

脚本会依次完成：

1. 根据 `protocol/*.proto` 生成 Flutter 端 protobuf 代码。
2. 构建 Flutter Web。
3. 将 Web 产物复制到后端静态资源目录。
4. 构建 C++ 后端。

## 4. 单独构建

仅构建 Flutter Web：

```bash
mkdir -p app/lib/protobuf
protoc --experimental_allow_proto3_optional \
  -I protocol \
  --dart_out=app/lib/protobuf \
  protocol/*.proto

cd app
flutter pub get
flutter gen-l10n
flutter build web --release
```

构建 Android APK：

```bash
mkdir -p app/lib/protobuf
protoc --experimental_allow_proto3_optional \
  -I protocol \
  --dart_out=app/lib/protobuf \
  protocol/*.proto

cd app
flutter pub get
flutter gen-l10n
flutter build apk --release
```

生成文件位置：

```text
app/build/app/outputs/flutter-apk/app-release.apk
```

仅构建后端：

```bash
cd backend
cmake -S . -B build -DCMAKE_INSTALL_PREFIX=build/install
cmake --build build --parallel
```

构建 ROS1 后端：

```bash
cmake -S backend -B backend/build -DROS_VERSION=1 -DCMAKE_INSTALL_PREFIX=backend/build/install
cmake --build backend/build --parallel
```

## 5. 运行

先启动后端：

```bash
source /opt/ros/humble/setup.bash
cd backend/build/install/bin
./start.sh
```

默认 HTTP 端口为 `8080`，浏览器访问：

```text
http://127.0.0.1:8080
```

也可以运行 Flutter 调试版：

```bash
cd app
flutter run -d chrome
```


