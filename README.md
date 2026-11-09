# deepseek-harness: Knowledge Graph kiến trúc

Dashboard tương tác (tiếng Việt) mô tả kiến trúc lõi của [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) tại commit `fcc0ad4`, tạo bằng [Understand-Anything](https://github.com/Egonex-AI/Understand-Anything).

**Đọc trước:** [Kiến trúc deepseek-harness](docs/kien-truc.md) · [Nhật ký phân tích](docs/nhat-ky-phan-tich.md)

## Xem

- **Online:** https://mt3rquy.github.io/deepseek-harness-understand/ (cần bật GitHub Pages: Settings → Pages → Deploy from branch `main`, thư mục `/ (root)`).
- **Local:** clone repo rồi chạy `npx serve .` và mở URL được in ra. Không mở `index.html` trực tiếp bằng `file://`, vì trình duyệt chặn `fetch` file JSON.

## Nội dung

| File | Ý nghĩa |
|---|---|
| `index.html`, `assets/` | Dashboard tĩnh (bản build demo của Understand-Anything) |
| `knowledge-graph.json` | Graph: 1.906 node, 2.504 edge, 10 layer, tour 11 bước |
| `meta.json`, `config.json` | Metadata phân tích và ngôn ngữ (`vi`) |
| `ua/.understandignore` | Các quy tắc lọc phạm vi phân tích |
| `ua/fingerprints.json`, `ua/intermediate/scan-result.json` | Trạng thái để `/understand` chạy incremental |
| `docs/` | Giải thích kiến trúc và nhật ký quá trình phân tích |
| `analysis-log/` | Các file trung gian (batch, layer, tour, review). Chỉ để tra cứu, **không** chép vào `.ua/` |

## Phạm vi và giới hạn

- Gồm 794 file: `packages/{core,boot,host,session,llm,context,subagent,extensions,sdk,api}`, `apps/cli`, README, AGENTS.md và `docs/architecture.md`. Không gồm test, locale, `client`, `experimental`, `bundle` và các package tool khác.
- Graph chủ yếu chứa import bên trong từng package. Import giữa các package (dùng tên `@deepseek-ai/dsh-*`) và cạnh `calls` gần như không có.
- Tóm tắt của function/class khá chung chung vì bước phân tích dùng cấu hình tiết kiệm (sonnet, effort thấp).

## Tái hiện hoặc chạy tiếp

Cần Claude Code có plugin Understand-Anything, Node.js ≥ 22 và pnpm.

**1. Khôi phục trạng thái vào bản clone của deepseek-harness.** Clone đầy đủ, không dùng `--depth`, để incremental tìm được commit gốc `fcc0ad4`:

```bash
git clone https://github.com/mt3rquy/deepseek-harness-understand
git clone https://github.com/deepseek-ai/deepseek-harness
cd deepseek-harness
mkdir -p .ua && cp -r ../deepseek-harness-understand/ua/. .ua/
cp ../deepseek-harness-understand/{knowledge-graph,meta,config}.json .ua/
```

**2. Chạy tiếp trong Claude Code**, tại thư mục `deepseek-harness`:

- `/understand`: so sánh với `fcc0ad4` và chỉ phân tích lại các file đã đổi (incremental).
- Mở rộng phạm vi, ví dụ thêm `packages/bundle`: thêm dòng `!/packages/bundle/` vào `.ua/.understandignore` rồi chạy `/understand`. Nếu file mới không được nhận thì chạy `/understand --full`.
- Hỏi đáp và giải thích: `/understand-chat`, `/understand-explain <file>`, `/understand-dashboard`.
- Muốn tái hiện đúng y như bản này: `git checkout fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159` trước khi chạy. Kết quả LLM sẽ không giống từng chữ.

**3. Cập nhật dashboard trong repo này** sau khi graph thay đổi: chép `knowledge-graph.json`, `meta.json`, `config.json` mới ra thư mục gốc, và các file trạng thái vào `ua/`. Nếu cần build lại giao diện thì chạy trong `packages/dashboard` của plugin:

```bash
VITE_GRAPH_URL=./knowledge-graph.json VITE_META_URL=./meta.json VITE_CONFIG_URL=./config.json npx vite build --config vite.config.demo.ts --base ./ --outDir <repo-này>
```
