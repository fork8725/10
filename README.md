# Traceability Dashboard 使用说明

本项目是基于 FastAPI + SQLite 的可追溯性管理小系统，包含原材料批次记录、批次与工序关联、设备维护与生产影响关联等模块，并提供一个前端仪表盘界面。

## 环境要求
- Windows (已测试)
- Python 3.8+（推荐 3.8/3.9）
- 依赖包见 `requirements.txt`

## 安装依赖
在项目根目录执行：

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

可选：若需要打包为单文件 exe，需要安装 PyInstaller：

```powershell
pip install pyinstaller
```

## 运行开发服务器
1. 初始化数据库（首次运行会自动创建并插入一个管理员用户）：
   - 管理员用户名：`admin`
   - 管理员密码：`admin123`

2. 启动服务：

```powershell
python main.py
```

启动后默认监听 `http://127.0.0.1:8000`。

3. 访问仪表盘：
- 浏览器打开 `http://127.0.0.1:8000/dashboard`
- 登录后可进行创建和删除操作（仅管理员）。未登录时只能浏览数据。

## 仪表盘功能说明
- 顶部登录表单：使用管理员帐号登录后，右侧会显示当前用户信息。
- Tab 切换数据集：
  - 原材料批次记录（`/raw-material-batches`）
  - 原材料批次与工序关联（`/material-batch-process-relations`）
  - 设备维护与生产影响关联（`/equipment-maintenance-production-relations`）
- 搜索框：对当前表数据进行包含匹配过滤。
- 分页：每页 10 条，上一页/下一页按钮控制。
- 管理员面板：登录后显示创建表单；列表中的“删除”按钮同样仅对管理员可见。

## 后端主要接口
- 认证登录：`POST /auth/login`（表单：`username`、`password`、`grant_type=password`）
- 当前用户：`GET /me`（需要 `Authorization: Bearer <token>`）
- 三类主数据的 CRUD：
  - `GET/POST/DELETE /raw-material-batches`
  - `GET/POST/DELETE /material-batch-process-relations`
  - `GET/POST/DELETE /equipment-maintenance-production-relations`

> 注：POST 与 DELETE 需要管理员权限（携带有效的 Bearer Token）。

## 打包为单文件 exe
项目自带 `build_exe.py`，会使用 PyInstaller 将 `main.py` 打包为单文件 exe，并处理静态目录与模板目录的资源复制，以及 `passlib` 的隐藏依赖。

1. 安装 PyInstaller：
```powershell
pip install pyinstaller
```

2. 执行打包脚本：
```powershell
python build_exe.py
```

脚本完成后会在 `dist/` 目录生成 `main.exe`，同时复制 `static/` 与 `templates/` 目录，确保可运行。

3. 运行 exe：
```powershell
./dist/main.exe
```
然后访问 `http://127.0.0.1:8000/dashboard`。

## 常见问题
- 启动报错 `ModuleNotFoundError`：请确认已正确安装依赖 `pip install -r requirements.txt`。
- 打包后页面样式或模板不生效：请确认 `dist/static` 与 `dist/templates` 目录存在，`build_exe.py` 已执行成功。
- 登录失败：请使用默认管理员 `admin/admin123`，或自行在数据库创建用户。打包版首次运行同样会自动插入该管理员。

## 目录结构简述
- `main.py`：FastAPI 应用入口与接口定义
- `models.py`：SQLAlchemy 模型与 `Base`
- `static/`：前端样式等静态资源
- `templates/`：Jinja2 模板（仪表盘）
- `app.db`：SQLite 数据库文件（自动创建）
- `build_exe.py`：打包脚本（PyInstaller）
- `requirements.txt`：依赖清单

## 安全提示
- 默认 `SECRET_KEY` 为示例值，建议在生产环境中修改为安全随机值。
- SQLite 适合单机或轻量场景；生产建议替换为更可靠的数据库并管理迁移。
