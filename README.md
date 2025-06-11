# Tauri-Pyodide

A desktop application boilerplate using [Tauri](https://tauri.app/), [Vue.js](https://vuejs.org/), and [Pyodide](https://pyodide.org/) to run Python code in the browser/webview. This template lets you build cross-platform desktop apps with a modern frontend and Python runtime support via WebAssembly.

## Features
- **Tauri** for secure, lightweight desktop app packaging
- **Vue 3** for reactive UI development
- **Pyodide** for running Python code in the browser/webview
- Pre-bundled Python packages (NumPy, Requests, etc.) in `public/pyodide`
- Easy integration of Rust backend (see `src-tauri`)

## Project Structure
```
├── src/                # Vue.js frontend source code
├── public/pyodide/     # Pyodide and Python wheels
├── src-tauri/          # Tauri (Rust) backend
├── index.html          # Main HTML entry
├── package.json        # Node.js project config
├── vite.config.js      # Vite build config
```

## Getting Started

### Prerequisites
- [Node.js](https://nodejs.org/) (v16+ recommended)
- [Yarn](https://yarnpkg.com/) or npm
- [Rust](https://www.rust-lang.org/tools/install) (for Tauri backend)

### Install dependencies
```bash
yarn install
```

### Run in development mode
```bash
yarn tauri dev
```

### Build for production
```bash
yarn tauri build
```

## Usage
- The Vue frontend is in `src/`.
- Python code can be executed in the browser using Pyodide (see `public/pyodide`).
- Rust backend logic can be added in `src-tauri/src/`.

## License
This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
