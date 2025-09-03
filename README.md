# 🐧💻 Ubuntu Development Container Template

This template provides a Ubuntu 25.04 development environment configured with commonly used development tools and languages. It's designed for use with VSCode's Remote - Containers extension, allowing you to develop in a consistent, isolated environment.

## 🛠️ Features

- Ubuntu 25.04 base image
- Essential development tools (build-essential, git, curl, wget, clang, etc.)
- Optional language installations:
  - Node.js (with npm and npx)
  - CMake
  - uv (Python package installer)
  - Go
  - Rust

## 📖 Usage

1. **Install VSCode and the Remote - Containers extension**: If you haven't already, install Visual Studio Code and the Remote - Containers extension (ms-vscode-remote.remote-containers).

2. **Create a new devcontainer**:
   - In VSCode, open the Command Palette (Ctrl+Shift+P)
   - Run "Remote-Containers: Add Development Container Configuration Files..."
   - Select "Ubuntu" from the templates list

3. **Customize versions** (optional): 
   You can specify versions for various tools by modifying the `args` section in [`.devcontainer/devcontainer.json`](.devcontainer/devcontainer.json):
   ```json
   "args": {
     "LOCAL_USER": "${localEnv:USER}",
     "CMAKE_VERSION": "latest",  // or specific version like "v3.28.0"
     "UV_VERSION": "latest",     // or specific version like "0.8.15"
     "NODE_VERSION": "latest",   // or "lts" or specific version like "v20.14.0"
     "GOLANG_VERSION": "latest", // or specific version like "go1.23.0"
     "RUST_VERSION": "latest"    // or specific version like "1.89.0"
   }
   ```

4. **Rebuild and reopen**:
   - VSCode will prompt you to rebuild the container
   - Run "Remote-Containers: Rebuild and Reopen in Container"

## 🚀 Motivation

This template was created to decouple development environments from Microsoft's devcontainer prebuilt images. Those images often contain unnecessary complexity, which can make debugging difficult at times.

By contrast, this template offers several advantages:
- Starts from a clean Ubuntu 25.04 base image
- Installs only the tools you need
- Provides transparency into installation sources
- Avoids unnecessary layers
- Offers faster build times
- Gives you complete control over the environment

This approach provides better control and transparency over your development environment while maintaining all the benefits of containerization.

## 🧩 Included VSCode Extensions

This template automatically installs the following VSCode extensions:
- Roo CLine
- Kilo Code
- Google's Gemini Code Assist