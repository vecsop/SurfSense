# SurfSense 本地化部署教程

本教程将指导您如何使用 Docker 在本地环境中部署和运行 SurfSense 项目。

## 目录

- [先决条件](#先决条件)
- [环境配置](#环境配置)
- [运行应用程序](#运行应用程序)
  - [全栈模式 (开发)](#全栈模式-开发)
  - [核心服务模式 (生产)](#核心服务模式-生产)
- [访问应用程序](#访问应用程序)
- [常用 Docker 命令](#常用-docker-命令)
- [数据库管理](#数据库管理)
- [故障排查](#故障排查)

## 先决条件

在开始之前，请确保您的系统上已安装以下软件：

- **Docker**: [获取 Docker](https://docs.docker.com/get-docker/)
- **Docker Compose**: 通常与 Docker Desktop 一同安装。
- **Git**: 用于克隆项目代码库。

## 环境配置

1.  **克隆代码库**
    打开终端或命令提示符，克隆 SurfSense 项目到您的本地计算机：

    ```bash
    git clone https://github.com/MODSetter/SurfSense.git
    cd SurfSense
    ```

2.  **创建环境变量文件**
    项目使用 `.env` 文件来管理配置。您需要从示例文件创建自己的 `.env` 文件。

    - **根目录 (可选)**: 用于 Docker 的特定配置。
      ```bash
      cp .env.example .env
      ```
    - **后端服务**:
      ```bash
      cp surfsense_backend/.env.example surfsense_backend/.env
      ```
    - **前端服务**:
      ```bash
      cp surfsense_web/.env.example surfsense_web/.env
      ```

3.  **配置环境变量**
    打开新创建的 `.env` 文件并根据您的需求填写必要的 API 密钥和其他配置。特别是 `surfsense_backend/.env` 和 `surfsense_web/.env` 文件。

    **`surfsense_backend/.env` 示例:**
    ```
    # .env file for backend
    # Add required API keys for different services
    # e.g. OPENAI_API_KEY=your_key_here
    ```

    **`surfsense_web/.env` 示例:**
    ```
    # .env file for frontend
    # Ensure the API URL is correct
    NEXT_PUBLIC_API_URL=http://localhost:8000
    ```

## 运行应用程序

SurfSense 提供了两种部署模式，您可以根据需要选择。

### 全栈模式 (开发)

此模式会启动所有服务，包括前端、后端、数据库和 pgAdmin。这是进行本地开发和测试的推荐方式。

在项目根目录下运行以下命令：

```bash
docker compose up --build
```

如果您希望在后台运行，请使用 `-d` (detached) 标志：

```bash
docker compose up --build -d
```

### 核心服务模式 (生产)

此模式仅启动数据库 (`db`) 和 `pgAdmin`。适用于您希望将前端和后端分开部署的生产环境。

```bash
docker compose -f docker-compose.yml up --build -d
```

## 访问应用程序

当所有容器成功启动后，您可以通过以下地址访问各个部分：

- **前端界面**: [http://localhost:3000](http://localhost:3000)
- **后端 API**: [http://localhost:8000](http://localhost:8000)
- **后端 API 文档 (Swagger UI)**: [http://localhost:8000/docs](http://localhost:8000/docs)
- **pgAdmin (数据库管理)**: [http://localhost:5050](http://localhost:5050)

## 常用 Docker 命令

以下是一些在开发过程中可能会用到的常用命令：

- **停止所有容器**:
  ```bash
  docker compose down
  ```

- **查看实时日志**:
  ```bash
  # 查看所有服务的日志
  docker compose logs -f

  # 查看特定服务的日志 (例如 backend)
  docker compose logs -f backend
  ```

- **重启服务**:
  ```bash
  # 重启后端服务
  docker compose restart backend
  ```

- **在容器内执行命令**:
  ```bash
  # 例如，在后端容器中运行测试
  docker compose exec backend python -m pytest
  ```

## 数据库管理

项目使用 pgAdmin 作为数据库的图形化管理工具。

1.  打开 [http://localhost:5050](http://localhost:5050)。
2.  使用以下默认凭据登录 (可在根目录 `.env` 文件中修改):
    - **邮箱**: `admin@surfsense.com`
    - **密码**: `surfsense`
3.  添加一个新的服务器连接：
    - **名称**: 任意名称，例如 `SurfSense DB`
    - **主机名/地址**: `db` (这是 Docker 网络中的服务名称)
    - **端口**: `5432`
    - **用户名**: `postgres` (可在 `.env` 文件中修改)
    - **密码**: `postgres` (可在 `.env` 文件中修改)
    - **维护数据库**: `surfsense`

## 故障排查

- **端口冲突**: 如果错误信息显示端口已被占用，您可以在根目录的 `.env` 文件中修改 `FRONTEND_PORT`, `BACKEND_PORT`, 或 `PGADMIN_PORT` 的值。
- **容器无法启动**: 使用 `docker compose logs -f <service_name>` 查看特定服务的日志以定位问题。通常问题可能出在 `.env` 文件的配置错误或缺少必要的 API 密钥。
- **数据库连接问题**: 确保在 pgAdmin 或其他数据库客户端中连接数据库时，主机名使用的是 `db` 而不是 `localhost`，因为这是容器在 Docker 内部网络中的名称。
- **权限问题 (Linux/macOS)**: 如果遇到权限错误，您可能需要在 `docker` 命令前加上 `sudo`。
