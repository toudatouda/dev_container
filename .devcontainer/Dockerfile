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

# 为 git 配置代理，因为 Homebrew 的安装依赖 git
RUN git config --global http.proxy "${http_proxy}" && \
    git config --global https.proxy "${https_proxy}"

# 3. 安装 Homebrew 所需的依赖
RUN apt-get -o Acquire::http::proxy="${http_proxy}" -o Acquire::https::proxy="${https_proxy}" update && apt-get install -y \
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

# 为 linuxbrew 用户配置 git
RUN sudo -u linuxbrew git config --global http.proxy "${http_proxy}" && \
    sudo -u linuxbrew git config --global https.proxy "${https_proxy}" && \
    sudo -u linuxbrew git config --global http.sslVerify false

# 安装 Homebrew
RUN sudo -u linuxbrew \
    http_proxy=${http_proxy} \
    https_proxy=${https_proxy} \
    NONINTERACTIVE=1 \
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 5. 配置 Homebrew 环境变量
ENV PATH="/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:${PATH}"
ENV MANPATH="/home/linuxbrew/.linuxbrew/share/man:${MANPATH}"
ENV INFOPATH="/home/linuxbrew/.linuxbrew/share/info:${INFOPATH}"

# 6. 使用 Homebrew 安装常用开发工具
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install netcat
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install wget
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install jq
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install yq
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install ripgrep
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install fd
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install bat
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install fzf
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install htop
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install tree
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install tmux
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install httpie
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install tldr
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install eza
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install gh
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install git-delta
RUN sudo -u linuxbrew http_proxy=${http_proxy} https_proxy=${https_proxy} /home/linuxbrew/.linuxbrew/bin/brew install lazygit

