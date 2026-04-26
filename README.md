<p align="center">
  <img src="assets/doofus-banner.svg" alt="doofus banner" width="100%"/>
</p>

# doofus-cli

[![SnS Compliant](https://img.shields.io/badge/SnS-Compliant-cba6f7?style=flat-square&labelColor=1e1e2e&logo=diamond&logoColor=cba6f7)](https://github.com/erikRepo/sns-manifesto)
[![Size](https://img.shields.io/badge/SnS-size%20%3C5MB-a6e3a1?style=flat-square&labelColor=1e1e2e&logo=diamond&logoColor=a6e3a1)](https://github.com/erikRepo/sns-manifesto)
[![Dependencies](https://img.shields.io/badge/SnS-deps%20zero-94e2d5?style=flat-square&labelColor=1e1e2e&logo=diamond&logoColor=94e2d5)](https://github.com/erikRepo/sns-manifesto)
[![Language](https://img.shields.io/badge/SnS-lang%20english-89b4fa?style=flat-square&labelColor=1e1e2e&logo=diamond&logoColor=89b4fa)](https://github.com/erikRepo/sns-manifesto)
[![License](https://img.shields.io/badge/SnS-MIT-b4befe?style=flat-square&labelColor=1e1e2e&logo=diamond&logoColor=b4befe)](LICENSE)
[![Auditable](https://img.shields.io/badge/SnS-auditable-f5c2e7?style=flat-square&labelColor=1e1e2e&logo=diamond&logoColor=f5c2e7)](https://github.com/erikRepo/sns-manifesto)

> A coding agent so simple, even a doofus can audit it.

A minimal local coding agent that talks to [Ollama](https://ollama.com) over a plain TCP connection. No cloud. No telemetry. No dependencies. Just you, a local model, and a codebase small enough to read in one sitting.

---

## ⚠️ YOLO Mode

doofus runs in full YOLO mode. The model can read files, write files, and execute bash commands on your machine — without confirmation. There are no safety rails, no sandboxing, and no undo.

**Run it in a container if you're not feeling brave.** A ready-made `docker-compose.yml` is included.

---

## What it does

`doofus` gives you a conversational coding agent in your terminal. It maintains conversation context across turns and runs with a configurable set of system prompts that define what tools the model can use: READ, WRITE, and BASH.

On startup, `fzf` opens a prompt selection menu — all prompts selected by default. Deselect any you don't want for the session. Everything runs locally through Ollama. Nothing leaves your machine.

Reasoning mode is on by default. The model thinks before it acts. Use `--no-think` if you want a direct answer without the internal monologue.

---

## Requirements

- [Ollama](https://ollama.com) running on `localhost:11434`
- A pulled model, e.g. `ollama pull llama3`
- [fzf](https://github.com/junegunn/fzf) for prompt selection
- Rust toolchain (to build from source)

Or just use Docker — see [Docker](#docker) below.

---

## Install

```bash
git clone https://github.com/erikRepo/doofus-cli.git
cd doofus-cli
cargo build --release
cp target/release/doofus ~/.local/bin/
```

---

## Usage

```bash
# Start a session — fzf opens to select prompts, reasoning on by default
doofus "refactor this function to be more readable"

# Pass a file as context
doofus main.rs "explain what this does"

# Skip fzf — load all prompts automatically
doofus --all "create a new module"

# Disable reasoning — model answers directly without thinking
doofus --no-think "what does this line do"

# Load an SnS skill for the session
doofus --skill ~/.claude/skills/sns/write/SKILL.md "create a new SnS-compliant module"

# Specify a model
doofus --model codellama "fix the bug in lib.rs"

# Specify a custom Ollama host
doofus --host 192.168.1.10:11434 "hello"
```

---

## Prompts

System prompts live in `prompts/` as plain JSON files. Each file defines one capability the model can use.

```
prompts/
  read.json    ← READ tool: read files and directories
  write.json   ← WRITE tool: create and edit files
  bash.json    ← BASH tool: execute shell commands
```

On startup, `fzf` presents all available prompts with all selected. Deselect any to restrict what the model can do for that session. You can add your own prompt files — doofus picks up anything in the `prompts/` directory.

---

## How it works

doofus speaks directly to Ollama's REST API over a plain `TcpStream` — no HTTP library, no TLS, no async runtime. The entire HTTP client is ~100 lines of standard library Rust that handles both chunked and fixed-length responses.

Tool calls are parsed from the model's response and executed locally before the next turn. Conversation context is kept in memory for the duration of the session.

---

## Architecture

```
doofus-cli/
  src/
    main.rs      ← entry point, argument parsing, session loop
    http.rs      ← TcpStream HTTP client, zero dependencies
    tools.rs     ← READ, WRITE, BASH tool execution
    prompt.rs    ← system prompt loading and message formatting
  prompts/
    read.json    ← READ tool definition
    write.json   ← WRITE tool definition
    bash.json    ← BASH tool definition
  docker-compose.yml
  Dockerfile.ollama
  Cargo.toml
  README.md
```

Four source files. No framework. Auditable in one sitting.

---

## Docker

The fastest way to get started without touching your local machine:

```bash
docker compose up
```

This starts Ollama in a container and mounts your current directory as the working directory for doofus. The model runs isolated — a good idea given YOLO mode.

```yaml
# docker-compose.yml
services:
  ollama:
    image: ollama/ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama

  doofus:
    build: .
    volumes:
      - .:/workspace
    depends_on:
      - ollama
    environment:
      - DOOFUS_HOST=ollama:11434

volumes:
  ollama_data:
```

---

## Configuration

| Flag | Default | Description |
|------|---------|-------------|
| `--model` | `llama3` | Ollama model to use |
| `--host` | `localhost:11434` | Ollama host and port |
| `--skill` | — | Path to a SKILL.md file to load as context |
| `--all` | — | Skip fzf, load all prompts automatically |
| `--no-think` | — | Disable reasoning mode, model answers directly |

---

## SnS Compliance

doofus is the reference implementation of the [SnS manifesto](https://github.com/erikRepo/sns-manifesto).

| Criterion | Status |
|-----------|--------|
| Binary size | < 5 MB |
| Runtime dependencies | Zero |
| Language | English |
| Source files | Four — readable in one sitting |

---

## License

MIT

<img src="assets/doofus-footer.svg" alt="doofus footer" width="100%"/>
