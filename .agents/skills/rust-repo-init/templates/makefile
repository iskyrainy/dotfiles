# Rust 项目常用命令入口。
# 常用:make build / make release / make run / make test / make lint / make fmt
# 交叉编译:make cross TARGET=aarch64-unknown-linux-musl(需要 cross 与 Docker)

# 交叉编译默认目标
TARGET ?= x86_64-unknown-linux-musl

# 项目形态适配:
#   - workspace 项目:取消下行注释,所有命令带 --workspace
#   - binary / library 单 crate 项目:保持注释即可
#SCOPE := --workspace

.PHONY: all build release run check test lint fmt fmt-check fix doc clean \
	cross cross-test cross-all

all: build

build:
	cargo build $(SCOPE)

release:
	cargo build --release $(SCOPE)

run:
	cargo run

check:
	cargo check --all-targets $(SCOPE)

test:
	cargo test $(SCOPE)

lint:
	cargo clippy --all-targets $(SCOPE) -- -D warnings

fmt:
	cargo fmt --all

fmt-check:
	cargo fmt --all -- --check

fix:
	cargo fix --all-targets $(SCOPE)

doc:
	cargo doc --no-deps $(SCOPE)

clean:
	cargo clean

# ---- 交叉编译 ----
cross:
	cross build --release --target $(TARGET)

cross-test:
	cross test --target $(TARGET)

cross-all:
	cross build --release \
		--target x86_64-unknown-linux-musl \
		--target aarch64-unknown-linux-musl \
		--target x86_64-pc-windows-gnu
