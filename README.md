# -IMA
语雀知识库文档同步至IMA脚本
# 语雀 → ima 知识库同步脚本 · 使用说明

把语雀知识库的文档批量导出为 Markdown，自动上传到腾讯 ima 知识库。

## 一、你需要准备的 3 样凭证

| 凭证 | 去哪拿 |
|---|---|
| **语雀 Token** | 语雀网页版 → 右上角头像 → 「设置」→ 左侧「Token」(个人设置 → 开发者) → 「新建 Token」，勾选读取权限，生成后立即复制（只显示一次） |
| **ima Client ID + API Key** | 打开 https://ima.qq.com/agent-interface → 微信扫码登录 → 点「生成 API Key」，**Client ID 和 API Key 都复制保存**（Key 只显示一次，丢失要撤销重生成，有效期 1 个月，到期重新生成即可） |

## 二、首次运行（3 步）

1. **双击 `运行同步.bat`**（或在本目录打开命令行执行 `python yuque2ima.py list`）
   首次运行如果 config.json 缺凭证会自动提示；`list` 命令会列出：
   - 你语雀的全部知识库（namespace | 名称 | 文档数）
   - 你 ima 可写入的知识库（ID | 名称）
2. 把要同步的语雀 namespace（如 `bi6z1y/mlqvc1`）填到 `config.json` 的 `yuque_namespace`；
   把 ima 知识库 ID 填到 `ima_kb_id`（也可不填，只有一个库时自动选择）。
3. **双击 `运行同步.bat`** 或命令行执行 `python yuque2ima.py sync` → 导出 + 上传一步到位。

## 三、命令一览

| 命令 | 作用 |
|---|---|
| `python yuque2ima.py list` | 列出语雀知识库 + ima 知识库，用来查 ID |
| `python yuque2ima.py export` | 只导出语雀文档到 `export/知识库名/` |
| `python yuque2ima.py upload` | 只上传 `export/` 下的 .md 到 ima |
| `python yuque2ima.py sync` | 导出 + 上传一步到位（最常用） |

可选参数：
- `--ns bi6z1y/mlqvc1` 临时指定语雀知识库（覆盖 config.json）
- `--kb xxx` 临时指定 ima 知识库 ID
- `--dir 路径` upload 模式上传任意本地目录的 .md 文件（比如你本地已有的 Listing 文案文件夹）
- `--keep-dup` 不跳过同名文件（默认同名自动跳过，防止知识库里堆重复）

示例：
```
python yuque2ima.py sync --ns bi6z1y/mlqvc1
python yuque2ima.py upload --dir "C:\Users\admin\Desktop\我的文案" --kb 123456
```

## 四、原理与限制（了解即可）

- 语雀侧走官方 v2 API（`X-Auth-Token`），逐篇取 Markdown 源码（`body` 字段）
- ima 侧走官方 OpenAPI：查重名 → create_media 拿 COS 凭证 → 签名上传腾讯云 COS → add_knowledge 入库（media_type=7 Markdown，单文件 ≤10MB）
- 同名文件自动跳过 → **重复执行 sync 只会增量同步新文档**，已传过的不会重复入库
- 文档里的本地图片、语雀图床图片不会跟随导入（ima 对纯文本 Markdown 支持最好）；表格、标题、列表等格式可正常保留
- 语雀「AI文档生成」目录是知识库内的文件夹，API 按“知识库”粒度导出，会导出整个库（如不需要某篇，上传前删掉对应 .md 再 upload 即可）
- ima API Key 有效期 1 个月，到期在 agent-interface 页面撤销重新生成，把新 Key 填回 config.json

## 五、常见问题

| 现象 | 处理 |
|---|---|
| 提示 Token 无效 / HTTP 401 | 语雀 Token 复制错了或权限没勾，重新生成 |
| ima 报 `code=110021` | 请求太频繁，稍等重跑（脚本已内置限速） |
| ima 报 `code=110030` | 该知识库没有写入权限，去 ima 里确认知识库归属当前微信账号 |
| 中文文件名乱码 | 用命令行运行，不要在旧版 cmd 里重定向输出；PowerShell 正常 |
| 上传成功但 ima 里看不到 | ima 解析需要几秒到几分钟，稍后刷新 |
