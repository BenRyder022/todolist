# 我的待办（单文件 ToDoList）

一个零依赖的本地待办清单应用。所有 HTML / CSS / JavaScript 都内联在 `index.html` 里，没有第三方库、没有 CDN、没有构建步骤。

## 怎么打开

双击 `index.html`，或用 Chrome 打开：

```
"C:\Program Files\Google\Chrome\Application\chrome.exe" "E:\1\index.html"
```

打开后即可使用，不需要装任何东西、不需要起服务。

## 功能

- **任务**：新增、行内编辑、勾选完成、删除（3 秒内可撤销）。
- **属性**：备注、截止日期（带「今天 / 明天」快捷键）、优先级（高 / 中 / 低 / 无）、标签（自由输入，自动提示用过的标签，支持中英文逗号和顿号分隔）。
- **筛选与搜索**：按状态（全部 / 未完成 / 已完成）、优先级、标签筛选；搜索框实时匹配标题、备注和标签。
- **排序**：手动拖拽、按截止日期、按优先级、按创建时间（新→旧）。
  - 手动排序为默认；只有「排序=手动 且 搜索为空 且 状态=全部 且 未筛选优先级和标签」时才允许拖动，其他情况拖拽手柄会变灰。
- **统计看板**：总数、未完成、今日到期、已逾期、完成率（无任务时显示「—」），以及全部任务的优先级分布条。
- **主题**：跟随系统 / 浅色 / 深色，点右上角按钮循环切换。
- **导入导出**：导出为 JSON 文件备份，导入时可选「合并到现有任务」或「清空后导入」。

## 数据存在哪

数据保存在浏览器的 localStorage 里，键名 `todolist.v1`，结构如下：

```json
{
  "version": 1,
  "tasks": [
    {
      "id": "…",
      "title": "写周报",
      "notes": "",
      "done": false,
      "createdAt": 1759161600000,
      "completedAt": null,
      "due": "2026-09-30",
      "priority": "high",
      "tags": ["工作"]
    }
  ],
  "prefs": { "theme": "system", "sort": "manual" }
}
```

- 只持久化 `theme` 和 `sort`；搜索关键词与筛选条件每次打开都会重置为「全部」。
- 任务的先后顺序就是 `tasks` 数组的顺序，没有额外的排序字段。
- 数据只存在**当前浏览器**里：换浏览器、换用户、清除浏览器数据都会丢。
- 如果数据文件损坏，应用会把原始内容备份到 `todolist.v1.broken`，并以空列表启动，同时顶部给出提示。
- 如果 localStorage 被禁用，应用会进入内存模式并常驻提示，此时改动不会保存，请用「导出」备份。

## 备份与恢复

- **备份**：点「导出」，得到 `todolist-YYYYMMDD-HHmm.json`。
- **恢复**：点「导入」选择该文件，然后在弹窗里选：
  - **合并到现有任务**：按任务 id 去重。id 相同且内容完全一致 → 跳过；id 相同但内容不同 → 自动分配新 id 追加，不会覆盖现有任务；id 不存在 → 追加。
  - **清空后导入**：先清空当前全部任务，再写入文件里的任务。
- 导入时每条任务都会做字段清洗：缺 id 自动生成，标题为空则跳过并计数，非法的优先级和截止日期归为「无」，标签不是数组则按空处理。
- 导出文件中包含 `version`、`exportedAt` 和 `tasks`，不含主题与排序偏好。

## 自测

用带参数的地址打开可以跑内置自测（覆盖字段清洗、增删改、四种排序、筛选、导入合并去重、导出结构、拖拽启用条件等）：

```
"C:\Program Files\Google\Chrome\Application\chrome.exe" "E:\1\index.html?selftest=1"
```

页面顶部会列出每一项的结果，全部通过时标签页标题为 `SELFTEST PASS`，有失败项则为 `SELFTEST FAIL`。也可以在无头模式下断言：

```
"C:\Program Files\Google\Chrome\Application\chrome.exe" --headless=new --no-sandbox --disable-gpu `
  --disable-software-rasterizer --disable-dev-shm-usage --no-first-run `
  --user-data-dir="$env:TEMP\todo-verify" --dump-dom "file:///E:/1/index.html?selftest=1"
```

## 已知限制

- 只针对桌面 Chrome 设计，没有做移动端适配，也不是 PWA（不能离线安装、没有 Service Worker）。
- 单条删除可以撤销，但「清空已完成」需要确认后不可撤销。
- 截止日期只精确到天，按本地日期比较，不处理时区、不重复提醒。
- 没有账号、云同步、桌面通知。
