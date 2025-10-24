# 使用微软的通用开发镜像
FROM ubuntu:22.04

# 1. 声明构建参数，并给它们一个空的默认值
ARG http_proxy="http://host.docker.internal:7897"
ARG https_proxy="http://host.docker.internal:7897"

# 2. 将这些参数设置为环境变量，以便 RUN 指令可以使用它们
#    同时设置大写的变量是为了兼容一些工具
ENV http_proxy=${http_proxy} \
    https_proxy=${https_proxy} \
    HTTP_PROXY=${http_proxy} \
    HTTPS_PROXY=${https_proxy} \
    HOMEBREW_CURL_INSECURE=true \
    CURL_SSL_NO_REVOKE=1 \
    CURL_CONNECT_TIMEOUT=30 \
    CURL_TIMEOUT=60

# 禁用 apt 的 HTTPS 证书验证并配置 apt 使用代理
RUN echo 'Acquire::https::Verify-Peer "false";' > /etc/apt/apt.conf.d/99verify-https && \
    echo 'Acquire::https::Verify-Host "false";' >> /etc/apt/apt.conf.d/99verify-https && \
    echo 'Acquire::http::Proxy "${http_proxy}";' > /etc/apt/apt.conf.d/01proxy && \
    echo 'Acquire::https::Proxy "${https_proxy}";' >> /etc/apt/apt.conf.d/01proxy

# 安装 git
RUN apt-get update && apt-get install -y git && rm -rf /var/lib/apt/lists/*

# 配置 Homebrew 使用国内镜像源，提高下载速度和稳定性
ENV HOMEBREW_API_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/api"
ENV HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles"

# 为 git 配置代理，因为 Homebrew 的安装依赖 git
RUN git config --global http.proxy "${http_proxy}" && \
    git config --global https.proxy "${https_proxy}" && \
    git config --global http.sslVerify false

# 3. 安装 Homebrew 所需的依赖
RUN apt-get update && \
    apt-get install -y \
    build-essential \
    curl \
    file \
    procps \
    docker.io \
    sudo \
    connect-proxy \
    zsh && \
    rm -rf /var/lib/apt/lists/*

SHELL ["/bin/zsh", "-c"]

# 4. 以安装 Homebrew
# 创建用于安装 Homebrew 的非 root 用户
RUN useradd --create-home --shell /bin/bash vscode && \
    mkdir -p /etc/sudoers.d/ && \
    echo "vscode ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/vscode && \
    usermod -aG docker vscode
RUN chmod 755 /home/vscode

# 为 linuxbrew 用户配置 git
RUN sudo -u vscode git config --global http.proxy "${http_proxy}" && \
    sudo -u vscode git config --global https.proxy "${https_proxy}" && \
    sudo -u vscode git config --global http.sslVerify false

# 安装 Homebrew
RUN curl -fsSL --insecure https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh -o /tmp/install_homebrew.sh && \
    [ -s /tmp/install_homebrew.sh ] || { echo "Homebrew install script download failed or is empty!"; exit 1; } && \
    sudo -u vscode /bin/bash -c "set -ex; NONINTERACTIVE=1 bash /tmp/install_homebrew.sh" && \
    [ -f /home/linuxbrew/.linuxbrew/bin/brew ] || { echo "Homebrew installation failed: brew executable not found!"; exit 1; }



# 5. 配置 Homebrew 环境变量
ENV PATH="/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:${PATH}"
ENV MANPATH="/home/linuxbrew/.linuxbrew/share/man:${MANPATH}"
ENV INFOPATH="/home/linuxbrew/.linuxbrew/share/info:${INFOPATH}"

# 6. 使用 Homebrew 安装常用开发工具
RUN BREW_INSTALL_CMD="sudo -u vscode HOMEBREW_CURL_ARGS=\"--insecure\" http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install" && \
    retry_command() { \
      local cmd="$@"; \
      for i in $(seq 1 5); do \
        if eval "${cmd}"; then \
          return 0; \
        else \
          echo "Command failed, attempt $i/5. Retrying in 5 seconds..."; \
          sleep 5; \
        fi; \
      done; \
      echo "Command failed after 5 attempts."; \
      return 1; \
    }; \
    retry_command "${BREW_INSTALL_CMD} netcat" && \
    retry_command "${BREW_INSTALL_CMD} wget" && \
    retry_command "${BREW_INSTALL_CMD} jq" && \
    retry_command "${BREW_INSTALL_CMD} yq" && \
    retry_command "${BREW_INSTALL_CMD} ripgrep" && \
    retry_command "${BREW_INSTALL_CMD} fd" && \
    retry_command "${BREW_INSTALL_CMD} bat" && \
    retry_command "${BREW_INSTALL_CMD} fzf" && \
    retry_command "${BREW_INSTALL_CMD} htop" && \
    retry_command "${BREW_INSTALL_CMD} tree" && \
    retry_command "${BREW_INSTALL_CMD} tmux" && \
    retry_command "${BREW_INSTALL_CMD} httpie" && \
    retry_command "${BREW_INSTALL_CMD} eza" && \
    retry_command "${BREW_INSTALL_CMD} gh" && \
    retry_command "${BREW_INSTALL_CMD} git-delta" && \
    retry_command "${BREW_INSTALL_CMD} lazygit"

ARG TARGETARCH

# 7. 安装 Miniconda
RUN case "${TARGETARCH}" in \
        "amd64") MINICONDA_ARCH="x86_64" ;; \
        "arm64") MINICONDA_ARCH="aarch64" ;; \
        *) echo "Unsupported architecture: ${TARGETARCH}"; exit 1 ;; \
    esac && \
    curl -fsSL -o /tmp/miniconda.sh https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-${MINICONDA_ARCH}.sh && \
    chmod +x /tmp/miniconda.sh && \
    /tmp/miniconda.sh -b -p /opt/conda && \
    rm /tmp/miniconda.sh

# 8. 配置 Miniconda 环境变量
ENV PATH="/opt/conda/bin:${PATH}"

# 9. 修正 Homebrew 目录的权限，以便其他用户（如 vscode）可以使用
RUN chmod -R 755 /home/linuxbrew/.linuxbrew

# 10. 配置 Shell 环境以识别 Homebrew
RUN echo '\neval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' | sudo tee -a /etc/zsh/zshrc && \
    echo '\neval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' | sudo tee -a /etc/profile

# 11. 安装 Node.js 和 n8n
RUN retry_command() { \
      local cmd="$@"; \
      for i in $(seq 1 5); do \
        if eval "${cmd}"; then \
          return 0; \
        else \
          echo "Command failed, attempt $i/5. Retrying in 5 seconds..."; \
          sleep 5; \
        fi; \
      done; \
      echo "Command failed after 5 attempts."; \
      return 1; \
    }; \
    BREW_INSTALL_NODE="sudo -u vscode HOMEBREW_CURL_ARGS='--insecure' http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install node"; \
    NPM_GLOBAL_PREFIX="sudo -u vscode PATH=\"/home/linuxbrew/.linuxbrew/bin:${PATH}\" /home/linuxbrew/.linuxbrew/bin/npm"; \
    retry_command "${BREW_INSTALL_NODE}" && \
    eval "${NPM_GLOBAL_PREFIX} config set proxy ${http_proxy}" && \
    eval "${NPM_GLOBAL_PREFIX} config set https-proxy ${https_proxy}" && \
    eval "${NPM_GLOBAL_PREFIX} install -g n8n n8n-node-dev" && \
    eval "${NPM_GLOBAL_PREFIX} install -g vitest" && \
    eval "${NPM_GLOBAL_PREFIX} install -g pnpm"
