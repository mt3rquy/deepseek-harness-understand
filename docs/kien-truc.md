# Kiến trúc deepseek-harness

Tóm tắt kiến trúc phần lõi của [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) tại commit `fcc0ad4`. Nguồn: knowledge graph trong repo này và tài liệu chính thức `docs/architecture.md` của upstream. Cách tạo ra graph được ghi trong [nhat-ky-phan-tich.md](nhat-ky-phan-tich.md).

## 1. Ý tưởng gốc: mọi thứ đều là plugin Cordis

Không có phần lõi "đặc quyền" nào. Model adapter, tool registry, session log, kể cả vòng lặp agent đều là plugin. Mỗi plugin đăng ký vào context dùng chung `ctx` ba loại thứ:

- **service**, ví dụ `ctx.llm`, `ctx.tools`, `ctx.sessions`, `ctx.agents`
- **typed event**, ví dụ `agent/*`, `tools/*`, `session/event`
- **effect**: các đăng ký này tự gỡ khi plugin unload

Muốn mở rộng thì gắn thêm plugin, không sửa code lõi.

## 2. Khởi động: profile → bundle → patch

`apps/cli/src/bin.ts` (lệnh `dsh`) gọi `profile-boot.ts`, từ đó vào `packages/boot/app-boot` (`profile.ts` là một trong những file được import nhiều nhất).

- **Profile** (`web`, `headless`, `sdk`, `sdk-minimal`, `acp`) là danh sách các bundle xếp chồng theo thứ tự.
- Thứ tự áp dụng: các bundle → `cordis.patch.yml` của profile → patch ở home → `--patch`. Mỗi patch thay một dòng config theo id.
- `plugin-manager`, `hmr` và `config-editor` quản lý cây plugin khi đang chạy.
- Lệnh `dsh --profile web --dump-config` in ra cây plugin thực tế sẽ được boot.

## 3. Lõi agent (`packages/core/*`)

| Package | Vai trò |
|---|---|
| `session` | Log `SessionEvent` chỉ ghi nối thêm (append-only), là nguồn sự thật |
| `agent` | Interface `Agent`, registry và các event `agent/*` |
| `agent-loop` | Driver mặc định chạy turn và step |
| `tools` + `tool-executor-*` | Registry tool theo scope và pipeline thực thi có guard |
| `system-prompt` | Ghép các section của prompt và schema của tool |
| `scope` | Đăng ký riêng cho từng agent |

**Luồng một turn:** một turn gồm nhiều step. Mỗi step chạy như sau:

1. `agent/pre-step`: được nhận, sửa hoặc từ chối input
2. Ghép prompt
3. `agent/request`, sau đó `prepareCall`
4. Stream qua `llm/stream`
5. `tool/call`, qua `tools/pre-execute` → `execute` → `post-execute`
6. `tool/result`, rồi sang step kế nếu còn tool cần gọi model tiếp

Các hook dạng waterfall bắt buộc gọi `next()`, nếu không chuỗi xử lý bị ngắt.

**Nguyên tắc vàng: "model-visible ⟺ logged".** Mọi request gửi tới model phải dựng lại được từ session log. Fork, resume, transcript và telemetry đều suy ra từ log này.

## 4. Capability seam

Mỗi năng lực gồm đủ ba vai trò: **Service Definition**, **Provider** và **Consumer** (thường là một tool cho model dùng). Đổi provider là cả sản phẩm đổi theo. Ví dụ: trỏ filesystem và subprocess sang sandbox từ xa thì Bash, PTY và LSP cũng chạy ở đó.

## 5. Các layer trong graph

| Layer | Số file | Thành phần chính |
|---|---|---|
| Khởi động & Tổ hợp | 139 | boot, host, apps/cli |
| Lõi Agent | 76 | core/* |
| Bộ chuyển đổi LLM | 100 | `llm` (seam), `llm-deepseek`, `llm-pi-ai`, `llm-retry`, `token-meter` |
| Session & Lưu trữ | 159 | session/* (JSONL có phiên bản, migration giữa các phiên bản liền kề) |
| Nhà cung cấp Context | 39 | agent-instructions, time, tmux, file-reference |
| Subagent | 65 | agent con, ủy quyền turn (backend ACP, Claude Code, Codex, SDK, in-process) |
| Tiện ích mở rộng | 53 | tự sửa đổi lúc chạy, ui-cordis |
| API & Controller | 136 | gateway, controller session/terminal/workspace cho Web và SDK RPC |
| SDK | 22 | client, server, protocol JSON-RPC |
| Tài liệu & Cấu hình gốc | 5 | README, AGENTS, package.json, pnpm-workspace, docs/architecture.md |

Các file được import nhiều nhất (chỉ tính import bên trong package): `api/session-controller/src/types.ts` (25), `api/terminal-controller/src/types.ts` (11), `llm/llm-deepseek/src/types.ts` (10), `boot/app-boot/src/profile.ts` (9), `boot/plugin-manager/src/types.ts` (9).

## 6. Gợi ý đọc theo thứ tự

Dashboard có tour 11 bước bằng tiếng Việt. Nếu đọc thẳng source của upstream thì theo thứ tự này:

1. `docs/architecture.md`
2. `apps/cli/src/profile-boot.ts`
3. `packages/boot/app-boot/README.md`
4. `packages/core/agent-loop/README.md`
5. `packages/llm/llm/src/types.ts`
6. `packages/core/session`

## 7. Giới hạn của graph

- **Thiếu cạnh giữa các package.** Các package import nhau bằng tên package (`@deepseek-ai/dsh-*`), nên graph chủ yếu có import bên trong từng package. Gần như không có cạnh `calls` (chỉ 4 cạnh), và có 168 node mồ côi, chủ yếu là README và package.json.
- **Tóm tắt function/class chung chung.** Các file-analyzer chạy với cấu hình tiết kiệm (sonnet, effort thấp) và phần lớn không đọc kỹ từng hàm. Tóm tắt ở mức file tốt hơn.
- **Ngoài phạm vi phân tích:** `packages/bundle` (nơi định nghĩa các profile thật), `client` (UI), `experimental`, các package tool (fs/shell/sandbox/mcp…), test và snapshots.
