# deepseek-harness: Knowledge Graph kiến trúc

Dashboard tương tác (tiếng Việt) mô tả kiến trúc lõi của [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) tại commit `fcc0ad4`, tạo bằng [Understand-Anything](https://github.com/Egonex-AI/Understand-Anything).

## Xem

- **Online:** https://mt3rquy.github.io/deepseek-harness-understand/ (cần bật GitHub Pages: Settings → Pages → Deploy from branch `main`, thư mục `/ (root)`).
- **Local:** clone repo rồi chạy `npx serve .` và mở URL được in ra. Không mở `index.html` trực tiếp bằng `file://`, vì trình duyệt chặn `fetch` file JSON.

## Nội dung

| File | Ý nghĩa |
|---|---|
| `index.html`, `assets/` | Dashboard tĩnh (bản build demo của Understand-Anything) |
| `knowledge-graph.json` | Graph: 1.906 node, 2.504 edge, 10 layer, tour 11 bước |
| `meta.json`, `config.json` | Metadata phân tích và ngôn ngữ (`vi`) |
| `understandignore.txt` | Các quy tắc lọc phạm vi phân tích |

## Phạm vi và giới hạn

- Gồm 794 file: `packages/{core,boot,host,session,llm,context,subagent,extensions,sdk,api}`, `apps/cli`, README, AGENTS.md và `docs/architecture.md`. Không gồm test, locale, `client`, `experimental`, `bundle` và các package tool khác.
- Graph chủ yếu chứa import bên trong từng package. Import giữa các package (dùng tên `@deepseek-ai/dsh-*`) và cạnh `calls` gần như không có.
- Tóm tắt của function/class khá chung chung vì bước phân tích dùng cấu hình tiết kiệm (sonnet, effort thấp).
