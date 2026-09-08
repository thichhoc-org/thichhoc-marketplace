# thichhoc marketplace

Claude Code plugin marketplace của thichhoc.

## Cài đặt

```bash
/plugin marketplace add thichhoc-org/thichhoc-marketplace
/plugin install thichhoc@thichhoc
```

Cập nhật về sau:

```bash
/plugin marketplace update thichhoc
```

## Cấu trúc

```
.claude-plugin/marketplace.json   # danh mục marketplace
plugins/
└── thichhoc/
    ├── .claude-plugin/plugin.json
    ├── skills/                   # SKILL.md, mỗi skill một thư mục
    ├── commands/                 # slash command, mỗi lệnh một file .md
    ├── agents/                   # subagent, mỗi agent một file .md
    └── hooks/                    # hooks.json (nếu dùng)
```

`metadata.pluginRoot` trỏ về `./plugins`, nên plugin mới chỉ cần khai tên thư mục
trong `plugins/` là đủ.

## Thêm một skill

```
plugins/thichhoc/skills/<ten-skill>/SKILL.md
```

```markdown
---
name: ten-skill
description: Làm gì, và khi nào Claude nên dùng nó. Câu mô tả này là thứ duy nhất
  Claude thấy khi quyết định có nạp skill hay không — viết rõ trigger.
---

Nội dung hướng dẫn...
```

## Thêm một slash command

```
plugins/thichhoc/commands/<ten-lenh>.md
```

```markdown
---
description: Mô tả ngắn hiện trong danh sách lệnh.
---

Prompt cho lệnh. `$ARGUMENTS` nhận tham số người dùng gõ vào.
```

Lệnh sẽ gọi bằng `/ten-lenh` (hoặc `/thichhoc:ten-lenh` khi trùng tên).

## Thêm một agent

```
plugins/thichhoc/agents/<ten-agent>.md
```

```markdown
---
name: ten-agent
description: Khi nào nên giao việc cho agent này.
tools: Read, Grep, Glob
---

System prompt của agent...
```

## Thêm plugin mới

1. Tạo `plugins/<ten-plugin>/.claude-plugin/plugin.json`.
2. Thêm một entry vào mảng `plugins` trong `.claude-plugin/marketplace.json`
   với `"source": "./plugins/<ten-plugin>"`.

## Kiểm tra trước khi push

```bash
claude plugin validate .
```

Test tại chỗ mà không cần push:

```bash
/plugin marketplace add ./thichhoc-marketplace
```
