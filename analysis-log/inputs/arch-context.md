## Language Context: typescript
# TypeScript Language Prompt Snippet

## Key Concepts

- **Generics**: Parameterized types (`<T>`) enabling reusable, type-safe abstractions
- **Type Guards**: Runtime checks that narrow types within conditional blocks (`is`, `in`, `typeof`, `instanceof`)
- **Discriminated Unions**: Union types with a shared literal field used for exhaustive narrowing
- **Utility Types**: Built-in mapped types like `Partial<T>`, `Pick<T, K>`, `Omit<T, K>`, `Record<K, V>`
- **Interfaces vs Types**: Interfaces support declaration merging; type aliases support unions and mapped types
- **Enums**: Numeric and string enums for named constant sets; prefer `as const` objects when possible
- **Mapped Types**: Transform existing types property-by-property using `[K in keyof T]` syntax
- **Conditional Types**: `T extends U ? X : Y` for type-level branching logic
- **Template Literal Types**: String manipulation at the type level using backtick syntax
- **Declaration Merging**: Interfaces with the same name merge their members automatically
- **Module Augmentation**: Extending third-party module types via `declare module` blocks

## Import Patterns

- `import { X } from 'module'` — named import (most common)
- `import type { X } from 'module'` — type-only import (erased at runtime)
- `import * as X from 'module'` — namespace import
- `import X from 'module'` — default import

## File Patterns

- `index.ts` — barrel file re-exporting public API from a directory
- `*.d.ts` — type declaration files (ambient declarations, no runtime code)
- `tsconfig.json` — TypeScript compiler configuration and project references
- `*.tsx` — TypeScript files containing JSX (React components)

## Common Frameworks

- **React** — UI component library with hooks and JSX
- **Angular** — Full-featured framework with decorators and dependency injection
- **Next.js** — React meta-framework with SSR, SSG, and API routes
- **NestJS** — Server-side framework inspired by Angular (decorators, modules, DI)
- **Express (with TS)** — Minimal HTTP framework with typed request/response handlers

## Example Language Notes

> Uses generic type parameter `T extends BaseEntity` to ensure type safety across
> repository methods. The constraint guarantees all entities share a common `id` field
> while allowing specific entity types to flow through the data layer without casting.
>
> Barrel files (`index.ts`) re-export symbols so consumers import from the directory
> rather than reaching into internal module paths — maintaining encapsulation.

## Language Context: markdown
# Markdown Language Prompt Snippet

## Key Concepts

- **Heading Hierarchy**: `#` through `######` for document structure, with h1 as the title
- **Front Matter**: YAML metadata between `---` delimiters at the top of the file
- **Fenced Code Blocks**: Triple backticks with optional language identifier for syntax highlighting
- **Reference-Style Links**: `[text][ref]` with `[ref]: url` definitions, useful for repeated URLs
- **Tables**: Pipe-delimited columns with alignment markers (`:---`, `:---:`, `---:`)
- **Admonitions**: Blockquote-based callouts (`> **Note:**`, `> **Warning:**`) for emphasis
- **Task Lists**: `- [ ]` and `- [x]` for checklists in issue trackers and READMEs
- **HTML Embedding**: Raw HTML allowed inline for features Markdown does not support natively

## Notable File Patterns

- `README.md` — Project overview and entry point for new contributors (high-value)
- `CONTRIBUTING.md` — Contribution guidelines, code style, PR process
- `CHANGELOG.md` — Version history following Keep a Changelog or similar format
- `docs/**/*.md` — Documentation directory with guides, API references, tutorials
- `*.md` in source directories — Co-located documentation for modules or packages
- `ADR-*.md` or `adr/*.md` — Architecture Decision Records

## Edge Patterns

- Markdown files `documents` the code components they describe or reference
- Links to other `.md` files create `related` edges between documentation nodes
- Code block references mentioning file paths may imply `documents` edges to those files
- README files in subdirectories typically `documents` the module at that path

## Summary Style

> "Project overview documentation with N sections covering installation, usage, and API reference."
> "Architecture Decision Record documenting the choice of [technology] for [purpose]."
> "Contributing guide with code style rules, testing requirements, and pull request process."

## Language Context: json
# JSON Language Prompt Snippet

## Key Concepts

- **Strict Syntax**: No trailing commas, no comments (unlike JSONC or JSON5), double-quoted strings only
- **Data Types**: Objects, arrays, strings, numbers, booleans, and null — no undefined or date types
- **Nested Structure**: Arbitrary nesting depth for hierarchical configuration or data
- **Schema Validation**: JSON Schema (`$schema` keyword) for validating structure and types
- **JSONC**: JSON with Comments variant used by VS Code, tsconfig.json, and other tooling
- **JSON5**: Extended JSON allowing comments, trailing commas, unquoted keys, and more
- **JSON Lines** (`.jsonl`): One JSON object per line for streaming data processing

## Notable File Patterns

- `package.json` — Node.js project manifest with dependencies, scripts, and metadata
- `tsconfig.json` — TypeScript compiler configuration (actually JSONC)
- `.eslintrc.json` — ESLint linting rules and configuration
- `*.schema.json` — JSON Schema definitions for validation
- `composer.json` — PHP Composer project manifest
- `appsettings.json` — .NET application configuration
- `manifest.json` — Browser extension or PWA manifest

## Edge Patterns

- `package.json` `configures` the build toolchain and defines project dependencies
- `tsconfig.json` `configures` TypeScript compilation for all `.ts` files
- JSON Schema files `defines_schema` for API request/response validation
- Config JSON files `configures` the runtime behavior of the application

## Summary Style

> "Node.js project manifest defining N dependencies, build scripts, and project metadata."
> "TypeScript compiler configuration enabling strict mode with path aliases for monorepo packages."
> "JSON Schema defining the request/response structure for the user API endpoint."

## Language Context: yaml
# YAML Language Prompt Snippet

## Key Concepts

- **Indentation-Based Nesting**: Whitespace-sensitive structure (spaces only, no tabs) defining hierarchy
- **Anchors and Aliases**: `&anchor` defines a reusable block, `*anchor` references it to avoid duplication
- **Merge Keys**: `<<: *anchor` merges anchor contents into the current mapping
- **Multi-Line Strings**: Literal block (`|`) preserves newlines, folded block (`>`) joins lines
- **Document Separators**: `---` starts a new document, `...` ends one (multi-document streams)
- **Tags and Types**: `!!str`, `!!int`, `!!bool` for explicit typing; custom tags for application-specific types
- **Flow Style**: Inline JSON-like syntax `{key: value}` and `[item1, item2]` for compact notation
- **Environment Variable Substitution**: `${VAR}` patterns used in docker-compose and CI configs

## Notable File Patterns

- `docker-compose.yml` / `docker-compose.yaml` — Multi-container Docker application definition
- `.github/workflows/*.yml` — GitHub Actions CI/CD workflow definitions
- `.gitlab-ci.yml` — GitLab CI/CD pipeline configuration
- `kubernetes/*.yaml` / `k8s/*.yaml` — Kubernetes resource manifests
- `*.config.yaml` — Application configuration files
- `mkdocs.yml` — MkDocs documentation site configuration
- `serverless.yml` — Serverless Framework configuration

## Edge Patterns

- YAML config files `configures` the code modules they control (e.g., database settings affect data layer)
- CI/CD YAML files `triggers` build and deployment pipelines
- docker-compose YAML `deploys` services and `depends_on` Dockerfiles
- Kubernetes YAML `deploys` and `provisions` application services

## Summary Style

> "Docker Compose configuration defining N services with networking, volumes, and health checks."
> "GitHub Actions workflow running tests on push and deploying to production on merge to main."
> "Kubernetes deployment manifest with N replicas, resource limits, and liveness probes."

## Language Context: css
# CSS Language Prompt Snippet

## Key Concepts

- **Selectors**: Element, class (`.name`), ID (`#name`), attribute (`[attr]`), and pseudo-class (`:hover`) targeting
- **Specificity**: Inline > ID > Class > Element cascade priority determining which rules win
- **Box Model**: `margin`, `border`, `padding`, `content` dimensions controlling element sizing
- **Flexbox**: `display: flex` with `justify-content`, `align-items` for one-dimensional layouts
- **Grid**: `display: grid` with `grid-template-columns/rows` for two-dimensional layouts
- **Custom Properties (Variables)**: `--name: value` with `var(--name)` for reusable design tokens
- **Media Queries**: `@media (max-width: ...)` for responsive design breakpoints
- **SCSS/Sass Features**: Nesting, `$variables`, `@mixin`, `@include`, `@extend`, `@use`, `@forward`
- **CSS Modules**: Scoped class names (`.module.css`) preventing global style collisions
- **Cascade Layers**: `@layer` for explicit control over cascade ordering

## Notable File Patterns

- `*.css` — Standard CSS stylesheets
- `*.scss` / `*.sass` — Sass/SCSS preprocessor files
- `*.less` — Less preprocessor files
- `*.module.css` / `*.module.scss` — CSS Modules (scoped styles)
- `globals.css` / `reset.css` / `normalize.css` — Global base styles
- `tailwind.config.js` — Tailwind CSS configuration (though a JS file)
- `variables.scss` / `_variables.scss` — Design token definitions

## Edge Patterns

- CSS files are `related` to the HTML or component files that import them for styling
- SCSS partial files (`_*.scss`) are `depends_on` by the main stylesheet that `@use`s them
- CSS variable definition files are `related` to all stylesheets that reference those variables
- CSS Modules are `related` to the component files that import them

## Summary Style

> "Global stylesheet defining CSS custom properties for the design system color palette and typography."
> "Responsive layout styles with flexbox and grid for the dashboard page across 3 breakpoints."
> "SCSS partial defining shared mixins for spacing, shadows, and media query breakpoints."

## Output Language Guidelines
# Hướng dẫn xuất nội dung tiếng Việt (Vietnamese)

Tệp này cung cấp hướng dẫn dành riêng cho tiếng Việt khi tạo nội dung đồ thị tri thức.

## Quy ước đặt thẻ

Dùng thẻ tiếng Việt hoặc thuật ngữ kỹ thuật tiếng Anh phổ biến:

| Mẫu | Thẻ gợi ý |
|---------|---------|
| Tệp điểm vào | `điểm-vào`, `barrel`, `exports` hoặc `entry-point` |
| Hàm tiện ích | `tiện-ích`, `helpers`, `utility` |
| Trình xử lý API | `api-handler`, `controller`, `endpoint` |
| Mô hình dữ liệu | `mô-hình-dữ-liệu`, `entity`, `schema` hoặc `data-model` |
| Tệp kiểm thử | `kiểm-thử`, `unit-test`, `test` |
| Tệp cấu hình | `cấu-hình`, `build-system`, `configuration` |
| Hạ tầng | `hạ-tầng`, `deployment`, `infrastructure` |
| Tài liệu | `tài-liệu`, `guide`, `documentation` |

**Chiến lược kết hợp:** giữ nguyên tiếng Anh cho thuật ngữ kỹ thuật phổ biến (`middleware`, `api-handler`...); có thể dùng tiếng Việt cho các thẻ mang tính mô tả.

## Phong cách tóm tắt

Viết tóm tắt 1-2 câu bằng tiếng Việt:
- Mô tả **mục đích** và **vai trò** của tệp
- Dùng câu chủ động ("Cung cấp...", "Xử lý...", "Quản lý...")
- Tránh lặp lại tên tệp

**Ví dụ:**
- Tốt: "Cung cấp các hàm tiện ích định dạng ngày và làm sạch chuỗi được dùng trong toàn bộ tầng API."
- Chưa tốt: "Tệp utils chứa các hàm tiện ích."

## Thuật ngữ kỹ thuật

Giữ nguyên tiếng Anh cho các thuật ngữ sau (khi không có bản dịch chuẩn):
- `middleware`, `hook`, `barrel`, `entry-point`
- `ORM`, `REST API`, `CI/CD`, `CRUD`
- `singleton`, `factory`, `observer`
- `interceptor`, `guard`

## Tên tầng

Dùng tên tầng tiếng Việt:
- `Tầng API`, `Tầng dịch vụ`, `Tầng dữ liệu`, `Tầng giao diện`
- `Hạ tầng`, `Cấu hình`, `Tài liệu`
- `Tầng tiện ích`, `Tầng middleware`, `Tầng kiểm thử`

Hoặc giữ nguyên tiếng Anh (theo quy ước của nhóm):
- `API Layer`, `Service Layer`, `Data Layer`
