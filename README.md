# MiniWorld Banned Words List

迷你世界（MiniWorld）客户端/服务端内置**内容过滤（敏感词）词表**的逆向提取与结构化整理版本。

> 原始数据为游戏内 Lua 配置文件（`Utf8Filter` 模块），本仓库将其解析、去重、分级后导出为通用格式，供内容安全研究、自研聊天过滤系统参考使用。

## 数据概览

| 项目 | 数量 |
|---|---|
| 原始词条（含重复） | 5,372 |
| 去重后词条 | **5,299** |
| 高敏感词 `score=10` | 5,231 |
| 中敏感词 `score=5` | 68 |
| 可疑/引导词（`suspect_word`，带通配） | 52 |
| 白名单词 | 约 105（去重后） |
| 特殊数字串 | 4 |
| 配置版本号 | 340 |

## 字段说明

每条主词表记录包含：

- `word`：被过滤的词或正则模式
- `score`：敏感等级，数值越大越严格
  - `10`：高敏感，命中即拦截/替换
  - `5`：中敏感，一般用于提示或记录
- 词中 `.` 为 Lua 模式通配符（等价于"任意字符"），例如 `黑.*科.*技` 可匹配"黑客科技""黑X科X技"等
- `^词$` 形式表示整词精确匹配

## 文件清单

| 文件 | 说明 |
|---|---|
| [`banned_words.json`](./banned_words.json) | **完整结构化数据**（含 meta、全部词条、分类白名单） |
| [`banned_words_all.txt`](./banned_words_all.txt) | 全量纯文本，每行 `[score]\tword`，按敏感等级降序 |
| [`banned_words_score10.txt`](./banned_words_score10.txt) | 仅高敏感词（5,231 条） |
| [`banned_words_score5.txt`](./banned_words_score5.txt) | 仅中敏感词（68 条） |
| [`suspect_words.txt`](./suspect_words.txt) | 可疑/引导性词（带通配符，52 条） |
| [`whitelist.txt`](./whitelist.txt) | 白名单（不过滤词） |

## 快速使用

### JSON

```python
import json
data = json.load(open('banned_words.json', encoding='utf-8'))
print(data['meta'])           # 元信息
print(len(data['words']))     # 5299
for w in data['words'][:5]:
    print(w['score'], w['word'])
```

### 纯文本

```bash
# 加载高敏感词到集合
grep -v '^#' banned_words_score10.txt | grep -v '^$' > blocklist.txt
```

### 注意：通配符转换

原始词表使用 Lua 模式（`.` 匹配任意字符）。若用于 Python/JS 正则，需先把 `.` 转义后再把 `.*` 还原：

```python
import re
def lua_pattern_to_regex(p: str) -> str:
    # 原始 .* 表示任意字符序列；先转义再还原
    return re.escape(p).replace(r'\.\*', '.*')
```

## 数据来源与方法

- 从游戏安装目录的内容过滤模块配置中提取（Lua table 格式）
- 解析后按 `word` 去重，同词保留最高 `score`
- 按敏感等级分组、排序
- 未对词条内容做任何增删改，仅做格式转换

## 免责声明

- 本仓库数据仅用于**内容安全技术研究、自研聊天过滤系统开发参考**
- 数据来源于客户端逆向，版权归游戏官方所有
- 请勿将本词表用于任何违法违规用途；使用者需自行遵守当地法律法规与平台规则
- 本项目与迷你世界官方无任何关联

## License

MIT
