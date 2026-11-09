# Nhật ký phân tích

Graph trong repo này được tạo bằng cách chạy skill `/understand` của Understand-Anything (plugin v2.9.8) trên deepseek-harness commit `fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159`, ngày 2026-11-09. Ngôn ngữ đầu ra là tiếng Việt (`--language vi`).

## Quyết định phạm vi

Upstream có khoảng 20.665 file, trong đó khoảng 3.300 file TypeScript không phải test. Người dùng chọn chỉ phân tích **lõi kiến trúc**. Quy tắc lọc nằm ở [`ua/.understandignore`](../ua/.understandignore):

- Giữ lại `packages/{core,boot,host,session,llm,context,subagent,extensions,sdk,api}`, `apps/cli`, README, AGENTS.md, package.json, pnpm-workspace.yaml và `docs/architecture.md`.
- Loại bỏ test, fixtures, snapshot, file locale i18n (khoảng 700 file JSON), `tsconfig*.json`, file `*.zh.md` và `*.i18n.yaml`.
- Kết quả: 794 file (585 TypeScript, 105 Markdown, 91 JSON, 9 YAML, 4 CSS), khoảng 155 nghìn dòng. 19.657 file bị lọc.

## Model và effort cho từng phase

Người dùng yêu cầu dùng subagent rẻ:

| Phase | Agent | Model / effort |
|---|---|---|
| 1 Scan | project-scanner | haiku / low |
| 2 Analyze | file-analyzer | sonnet / low |
| 3 Assemble review | assemble-reviewer | haiku / low |
| 4 Layers | architecture-analyzer | sonnet / medium |
| 5 Tour | tour-builder | sonnet / medium |
| 6 Validate | script kiểm tra cố định (không dùng LLM) | — |

## Diễn biến

1. **Scan:** 794 file; import map có 328 file có import và 787 cạnh nội bộ.
2. **Batching:** `compute-batches.mjs` chia thành 73 batch (lớn nhất 27 file). 203 batch nhỏ được gộp thành 13 batch "misc". Có 1 cảnh báo: một cụm 40 file vượt giới hạn 35 nên bị cắt theo thứ tự chữ cái.
3. **Analyze:** 73 batch được gộp thành 26 lần dispatch (khoảng ≤ 8k dòng mỗi lần), chạy tối đa 5 song song. Mỗi batch vẫn ghi một file `batch-<i>.json` riêng. 4 batch bị tách thành nhiều part (2, 6, 9, 22).
   - Các agent tự báo cáo những điểm chưa làm: tóm tắt function/class viết theo mẫu từ tên symbol, phần lớn không chạy bước tự kiểm tra edge, chỉ tạo node cho symbol chính, gần như không tạo cạnh `calls`/`tested_by`.
   - Một báo động giả: lệnh kiểm tra `ls batch-$i.json batch-$i-part-*.json` báo "MISSING" cho cả 73 batch, vì `ls` trả lỗi khi glob `-part-*` không khớp. Kiểm tra lại thì đủ 73 index, 77 file.
4. **Merge:** 1.906 node và 2.504 edge. Không có node trùng, không có edge treo, không phải chuẩn hóa gì. Cả 794 file đều có node, và đủ 787/787 cạnh import (xem `analysis-log/inputs/merge-stderr.txt`).
5. **Assemble review:** không sửa gì. Phát hiện 168 node mồ côi (68 config, 58 document, 42 file), xem `analysis-log/assemble-review.json`.
6. **Layers:** 10 layer, phủ 794/794 node, mỗi node thuộc đúng một layer. Agent gán layer theo tiền tố thư mục, không kiểm chứng lại bằng đồ thị import. `packages/host` được gộp vào layer Khởi động.
7. **Tour:** 11 bước. Agent chọn node dựa trên summary và ghi chú kiến trúc, không đọc source.
8. **Validate:** 0 issue, 168 cảnh báo node mồ côi.
9. **Save:** fingerprint cho 794 file. Có các YAMLWarning vô hại do tag `!!js` riêng của Cordis trong các file `*.cordis.yml`.

Thống kê cuối cùng:

| Loại node | Số lượng |
|---|---|
| file | 589 |
| function | 857 |
| class | 255 |
| document | 105 |
| config | 100 |

| Loại edge | Số lượng |
|---|---|
| contains | 1.112 |
| imports | 787 |
| exports | 526 |
| documents | 45 |
| configures | 29 |
| calls | 4 |
| related | 1 |

## Dashboard

Bản viewer dựng sẵn (`understand-anything-viewer.tgz` v2.9.8) không có trên GitHub release (lỗi 404), nên lúc đầu dashboard được chạy bằng Vite dev server trong container. Bản trong repo này là build demo mode (`vite.config.demo.ts`, `--base ./`), đọc `knowledge-graph.json` qua URL tương đối nên không cần server hay token. Đã thử bằng trình duyệt headless: tải đủ 10 layer, không có lỗi.

## Nội dung `analysis-log/`

| Đường dẫn | Nội dung |
|---|---|
| `batches.json` | Cách chia 73 batch: file, import đã resolve, neighbor map |
| `batches/batch-*.json` | Node/edge do từng file-analyzer tạo ra (77 file) |
| `layers.json`, `tour.json` | Đầu ra của Phase 4 và 5 (đã chuẩn hóa) |
| `assemble-review.json`, `review.json` | Báo cáo của Phase 3 và Phase 6 |
| `inputs/` | Log merge, ngữ cảnh ngôn ngữ/locale cho architecture-analyzer, cây thư mục, script kiểm tra |

**Không khôi phục các file này vào `.ua/intermediate/`.** Script merge đọc mọi `batch-*.json` trong thư mục đó, nên lần chạy incremental sau sẽ trộn nhầm dữ liệu cũ. Chúng chỉ dùng để tra cứu và đối chiếu.

Những thứ không lưu: script nháp các agent tự viết, file input/output tạm của từng agent, và kết quả `extract-structure` (khoảng 5MB, các script tạo lại được theo cách tất định từ source).
