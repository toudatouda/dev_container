# 使用微软的通用开发镜像
FROM mcr.microsoft.com/devcontainers/base:ubuntu

# 1. 声明构建参数，并给它们一个空的默认值
ARG http_proxy="http://host.docker.internal:7897"
ARG https_proxy="http://host.docker.internal:7897"
ARG HTTP_PROXY="http://host.docker.internal:7897"
ARG HTTPS_PROXY="http://host.docker.internal:7897"

# 2. 将这些参数设置为环境变量，以便 RUN 指令可以使用它们
ENV http_proxy=${http_proxy}
ENV https_proxy=${https_proxy}
ENV HTTP_PROXY=${HTTP_PROXY}
ENV HTTPS_PROXY=${HTTPS_PROXY}
ENV HOMEBREW_CURL_INSECURE=true

# 禁用 apt 的 HTTPS 证书验证，以解决代理可能导致的 502 Bad Gateway 或 SSL 错误
RUN echo 'Acquire::https::Verify-Peer "false";' > /etc/apt/apt.conf.d/99verify-https && \
    echo 'Acquire::https::Verify-Host "false";' >> /etc/apt/apt.conf.d/99verify-https

# 明确配置 apt 使用代理
RUN echo 'Acquire::http::Proxy "${http_proxy}";' > /etc/apt/apt.conf.d/01proxy && \
    echo 'Acquire::https::Proxy "${https_proxy}";' >> /etc/apt/apt.conf.d/01proxy

# 配置 Homebrew 使用国内镜像源，提高下载速度和稳定性
ENV HOMEBREW_API_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/api"
ENV HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles"

# 为 git 配置代理，因为 Homebrew 的安装依赖 git
RUN git config --global http.proxy "${http_proxy}" && \
    git config --global https.proxy "${https_proxy}" && \
    git config --global http.sslVerify false

# 3. 更换 apt 源为清华大学镜像源，并安装 Homebrew 所需的依赖
RUN { \
    echo "Types: deb"; \
    echo "URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/"; \
    echo "Suites: jammy jammy-updates jammy-backports jammy-security"; \
    echo "Components: main restricted universe multiverse"; \
    echo "Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg"; \
    } > /etc/apt/sources.list.d/ubuntu.sources && \
    # 清除旧的 sources.list 文件，避免冲突
    rm -f /etc/apt/sources.list && \
    apt-get update && apt-get install -y \
    build-essential \
    curl \
    file \
    git \
    procps \
    && rm -rf /var/lib/apt/lists/*

# 4. 以安装 Homebrew
# 创建用于安装 Homebrew 的非 root 用户
RUN useradd --create-home --shell /bin/bash linuxbrew && \
    echo "linuxbrew ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/linuxbrew
RUN chmod 755 /home/linuxbrew

# 为 linuxbrew 用户配置 git
RUN sudo -u linuxbrew git config --global http.proxy "${http_proxy}" && \
    sudo -u linuxbrew git config --global https.proxy "${https_proxy}" && \
    sudo -u linuxbrew git config --global http.sslVerify false

# 安装 Homebrew
RUN sudo -u linuxbrew \
    http_proxy=${http_proxy} \
    https_proxy=${https_proxy} \
    HOMEBREW_API_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/api" \
    HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles" \
    NONINTERACTIVE=1 \
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 5. 配置 Homebrew 环境变量
ENV PATH="/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:${PATH}"
ENV MANPATH="/home/linuxbrew/.linuxbrew/share/man:${MANPATH}"
ENV INFOPATH="/home/linuxbrew/.linuxbrew/share/info:${INFOPATH}"

# 6. 使用 Homebrew 安装常用开发工具
RUN sudo -u linuxbrew HOMEBREW_CURL_ARGS="--insecure" http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install \
    netcat \
    wget \
    jq \
    yq \
    ripgrep \
    fd \
    bat \
    fzf \
    htop \
    tree \
    tmux \
    httpie \
    tldr \
    eza \
    gh \
    git-delta \
    lazygit

# 7. 修正 Homebrew 目录的权限，以便其他用户（如 vscode）可以使用
RUN chmod -R 755 /home/linuxbrew/.linuxbrew

# 8. 配置 Shell 环境以识别 Homebrew
RUN echo '\neval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' | sudo tee -a /etc/zsh/zshrc && \
    echo '\neval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' | sudo tee -a /etc/profile
