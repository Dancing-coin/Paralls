# filtered_conversation_split

这是由 `filtered_conversation.json` 拆分出的问答档案。

## 目录结构

- `turns/by-range/`：405 个完整问答文件，按 50 组一个区间分桶
- `indexes/all-turns.md`：全部问答总索引
- `indexes/ranges/`：每个编号区间的索引
- `extras/unpaired-questions/`：原始数据中未配对成功的问题

## 三级结构说明

问答正文采用至少三级目录：

`filtered_conversation_split/turns/by-range/001-050/001-xxx.md`

这样可以同时满足：

- 单目录文件数可控
- 直接按编号区间定位
- 后续继续扩展时不需要重排全部文件

## 统计

- 完整问答：405
- 未配对问题：5
- 导出时间：2026-05-23 05:02:27
