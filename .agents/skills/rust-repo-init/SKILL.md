---
name: rust-repo-init
description: 为 Rust 仓库初始化/补全标准工程细节文件,让 cargo new 之后的仓库或缺配置的现有仓库达到可直接开发、提交、CI 的完整状态。生成 .gitignore、rustfmt.toml、.editorconfig、.gitattributes、Cross.toml(交叉编译)、Makefile、.github/workflows/ci.yml,并自动检测 binary/library/workspace 项目形态、按需补充 rust-toolchain.toml 等。当用户刚运行 cargo new、说"补全/完善 Rust 仓库"、"添加 .gitignore / rustfmt.toml / .editorconfig / .gitattributes / Makefile"、"配置交叉编译 / Cross.toml"、"加 CI / GitHub Actions"、"初始化 Rust 项目"时,务必使用本 skill,即使只提到了其中一两个文件。
---

# Rust Repo Init

为 Rust 仓库补齐标准工程文件,让项目从"能编译"变成"团队可协作、可 CI、可交叉编译"的完整状态。核心思路:先搞清楚项目形态,再按模板生成文件,最后验证配置可用。

## 何时使用

- 用户刚运行 `cargo new project`,想完善仓库
- 用户说"补全/完善 Rust 仓库(细节)文件"、"初始化 Rust 项目"
- 用户点名要其中某个文件:".gitignore"、"rustfmt.toml"、"editorconfig"、".gitattributes"、"Makefile"、"Cross.toml"、"交叉编译"、"CI"、"GitHub Actions"
- 接手现有 Rust 仓库,发现这些标准文件缺失

## 工作流程

1. **检测项目形态**(读 Cargo.toml,必要时并行检查文件):
   - 存在 `[workspace]` → workspace
   - `src/lib.rs` 或 `[lib]` → library
   - `src/main.rs` 或 `[[bin]]` → binary
2. **盘点已有文件**:已存在的细节文件**默认不覆盖**(跳过并汇报),只有用户明确要求时才重写。git 尚未初始化时提示 `git init`(或问用户是否执行)。
3. **生成文件**:模板在 `templates/` 目录,逐个读取、按下节规则适配后写入项目根目录。
4. **验证并汇报**:运行 `cargo fmt --all -- --check` 与 `cargo clippy --all-targets -- -D warnings`(workspace 加 `--workspace`)确认配置无冲突;`make -n build` 确认 Makefile 语法。最后汇报:生成了哪些文件、跳过了哪些(已存在)、适配了哪种项目形态。

## 文件清单

### 核心文件(总是生成,除非已存在)

| 文件 | 模板 | 关键决策 |
|---|---|---|
| `.gitignore` | `templates/gitignore` | **library 忽略 Cargo.lock;binary 与 workspace 提交 Cargo.lock**——锁文件保证可复现构建 |
| `rustfmt.toml` | `templates/rustfmt.toml` | 无需改,与 .editorconfig 天然对齐 |
| `.editorconfig` | `templates/editorconfig` | 无需改 |
| `.gitattributes` | `templates/gitattributes` | 无需改 |
| `Cross.toml` | `templates/cross.toml` | **按项目实际需求注释掉不用的目标**(目标越多构建越慢);macOS 目标需 `XCODE_VERSION` 环境变量,模板里有说明 |
| `Makefile` | `templates/makefile` | **workspace:启用 `SCOPE := --workspace` 行;binary/library:保持注释** |
| `.github/workflows/ci.yml` | `templates/ci.yml` | workspace 时在三个 job 的 cargo 命令里加 `--workspace`;默认分支已兼容 main/master |

### 可选文件(按需补充)

| 文件 | 模板 | 何时生成 |
|---|---|---|
| `rust-toolchain.toml` | `templates/rust-toolchain.toml` | 默认生成——固定工具链让团队和 CI 版本一致,收益远大于成本;若项目已有 toolchain 配置或用户拒绝则跳过 |
| `README.md` | 无 | 仓库已有实际内容且无 README 时可建议生成(可调用 code-documentation skill),用户不要求则跳过 |
| `LICENSE` / `deny.toml` / `.cargo/config.toml` | 无 | 仅当用户要求时 |

## 适配要点

- **Cargo.lock**:binary/workspace 提交(CI 可复现);library 在 .gitignore 中忽略。如果 git 仓库还没初始化,这个决策也不受影响。
- **Cross.toml**:目标平台过多会让 `make cross-all` 变得很慢,生成后提醒用户按需启用/注释。新装的 cross 目标都要先 `rustup target add <target>`。
- **已存在文件**:跳过并明确汇报,不静默覆盖——用户自己写的配置优先。
- **风格一致性**:rustfmt.toml / .editorconfig / .gitattributes 三者是同一套约定(4 空格、LF、utf-8、结尾换行),不要只生成其中一部分时让它们互相矛盾。
