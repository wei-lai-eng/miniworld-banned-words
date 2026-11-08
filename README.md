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
| 可疑/引导词（`suspect_word`，带通配） | 51 |
| 白名单词（`white_word` + `cs_white_word` 去重） | 104 |
| 特殊数字串 | 4 |
| 配置版本号 | 340 |

## 字段说明

每条主词表记录包含：

- `word`：被过滤的词或 Lua 模式
- `score`：敏感等级，数值越大越严格
  - `10`：高敏感，命中即拦截/替换
  - `5`：中敏感，一般用于提示或记录
- 词中 `.` 为 Lua 模式通配符（等价于"任意字符"），例如 `白.*科.*技` 可匹配"白X科X技"等
- 部分词条含 `.` 以外的字符（如 `.*`、`．`），按原始字节保留

## 文件清单

主词表按每片约 800 词切成 7 个分片，每行格式 `[score] word`：

| 文件 | 说明 |
|---|---|
| [`words_index.json`](./words_index.json) | 词表元信息（版本、总数、分片数、各级计数） |
| [`words_part_01.txt`](./words_part_01.txt) ~ [`words_part_07.txt`](./words_part_07.txt) | 主词表分片，共 5,299 条；前 6 片各 800 词，末片 499 词 |
| [`banned_words_score5.txt`](./banned_words_score5.txt) | 仅中敏感词（68 条，独立导出便于引用） |
| [`suspect_words.txt`](./suspect_words.txt) | 可疑/引导性词（带通配符，51 条） |
| [`whitelist.txt`](./whitelist.txt) | 白名单（不过滤词，104 条） |

> 由于完整 JSON（约 300 KB）超过单次 API 推送限制，主词表以纯文本分片形式发布。需要合并时执行：
> ```bash
> cat words_part_*.txt > banned_words_all.txt
> ```

## 快速使用

### 加载全部词表

```bash
# 合并分片（去掉注释行）
grep -h -v '^#' words_part_*.txt | grep -v '^$' > all.txt
wc -l all.txt   # 应为 5299
```

```python
# 解析为 (score, word) 列表
words = []
for line in open('all.txt', encoding='utf-8'):
    line = line.rstrip('\n')
    if not line or line.startswith('#'):
        continue
    score_s, _, w = line.partition('] ')
    score = int(score_s[1:])
    words.append((score, w))
print(len(words))          # 5299
print(words[:3])
```

### 仅高敏感词

```bash
grep '^\[10\]' words_part_*.txt > score10.txt
wc -l score10.txt   # 5231
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
- 未对词条内容做任何增删改，仅做格式转换与分片

## 免责声明

- 本仓库数据仅用于**内容安全技术研究、自研聊天过滤系统开发参考**
- 数据来源于客户端逆向，版权归游戏官方所有
- 请勿将本词表用于任何违法违规用途；使用者需自行遵守当地法律法规与平台规则
- 本项目与迷你世界官方无任何关联

## License

MIT
