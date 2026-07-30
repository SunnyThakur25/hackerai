# ============================================================================
# Base Image
# ============================================================================
FROM kalinux/kali-rolling

# ============================================================================
# Metadata Labels
# ============================================================================
LABEL org.opencontainers.image.title="HackerAI Agent Sandbox"
LABEL org.opencontainers.image.description="AI Agent Penetration Testing Environment with Comprehensive Automated Tools"
LABEL org.opencontainers.image.source="https://github.com/hackerai-tech/hackerai"
LABEL org.opencontainers.image.vendor="HackerAI"

# ============================================================================
# Environment Variables
# ============================================================================
ENV DEBIAN_FRONTEND=noninteractive
ENV HOME=/home/user
ENV GOPATH=/home/user/go
ENV GOCACHE=/home/user/go/.cache
ENV PATH=/home/user/.local/bin:/usr/local/go/bin:/home/user/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# ============================================================================
# Stage 1: Base System Setup
# ============================================================================
RUN apt-get update && \
    apt-get install -y kali-archive-keyring sudo && \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

# ============================================================================
# Stage 2: Install System Packages
# ============================================================================
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    # -------------------------------------------
    # Essential Utilities
    # -------------------------------------------
    curl \
    wget \
    git \
    ca-certificates \
    gnupg2 \
    unzip \
    jq \
    tree \
    perl \
    sudo \
    # -------------------------------------------
    # Build Tools
    # -------------------------------------------
    build-essential \
    gcc \
    g++ \
    make \
    cmake \
    pkg-config \
    binutils \
    # -------------------------------------------
    # Python Stack
    # -------------------------------------------
    python3 \
    python3-venv \
    python3-pip \
    python3-dev \
    python-is-python3 \
    libffi-dev \
    libxml2-dev \
    libxslt1-dev \
    zlib1g-dev \
    libglib2.0-dev \
    # -------------------------------------------
    # Ruby Stack
    # -------------------------------------------
    ruby-full \
    ruby-dev \
    libnet-ssleay-perl \
    libio-socket-ssl-perl \
    libcrypt-ssleay-perl \
    libssl-dev \
    # -------------------------------------------
    # Network Utilities
    # -------------------------------------------
    iputils-ping \
    dnsutils \
    iproute2 \
    net-tools \
    whois \
    traceroute \
    netcat-traditional \
    tcpdump \
    # -------------------------------------------
    # Search & Text Tools
    # -------------------------------------------
    ripgrep \
    vim \
    nano \
    less \
    # -------------------------------------------
    # Network Scanning & Enumeration
    # -------------------------------------------
    nmap \
    naabu \
    nikto \
    whatweb \
    wafw00f \
    # -------------------------------------------
    # Web Vulnerability Scanners
    # -------------------------------------------
    sqlmap \
    wapiti \
    wpscan \
    # -------------------------------------------
    # DNS/Subdomain Enumeration
    # -------------------------------------------
    subfinder \
    dnsrecon \
    dnsenum \
    # -------------------------------------------
    # Web Fuzzing & Discovery
    # -------------------------------------------
    ffuf \
    arjun \
    gobuster \
    dirsearch \
    # -------------------------------------------
    # SMB/NetBIOS Tools
    # -------------------------------------------
    smbclient \
    smbmap \
    nbtscan \
    python3-impacket \
    enum4linux \
    # -------------------------------------------
    # Network Discovery
    # -------------------------------------------
    hping3 \
    arp-scan \
    # -------------------------------------------
    # Utilities
    # -------------------------------------------
    socat \
    proxychains4 \
    seclists \
    hashid \
    libimage-exiftool-perl \
    cewl \
    # -------------------------------------------
    # SSL/TLS Testing
    # -------------------------------------------
    testssl.sh \
    # -------------------------------------------
    # Web Crawling & Recon
    # -------------------------------------------
    gospider \
    nuclei \
    # -------------------------------------------
    # Authentication/Bruteforce
    # -------------------------------------------
    hydra \
    # -------------------------------------------
    # Programming Languages
    # -------------------------------------------
    golang \
    nodejs \
    npm \
    # -------------------------------------------
    # Additional Tools
    # -------------------------------------------
    tmux \
    chromium \
    fonts-liberation \
    # -------------------------------------------
    # Forensics
    # -------------------------------------------
    binwalk \
    foremost \
    # -------------------------------------------
    # Document Conversion
    # -------------------------------------------
    pandoc \
    poppler-utils \
    # -------------------------------------------
    # Security Frameworks
    # -------------------------------------------
    zaproxy \
    metasploit-framework \
    msfpc \
    && apt-get clean && \
    rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

# ============================================================================
# Stage 3: Fix WhatWeb Ruby Library Symlink
# ============================================================================
RUN set -eux; \
    if command -v whatweb >/dev/null 2>&1; then \
        mkdir -p /usr/bin/lib; \
        if ls /usr/lib/ruby/vendor_ruby/*.rb >/dev/null 2>&1; then \
            ln -sf /usr/lib/ruby/vendor_ruby/*.rb /usr/bin/lib/; \
        fi; \
    fi

# ============================================================================
# Stage 4: User Setup
# ============================================================================
RUN set -eux; \
    # Create user if not exists
    if ! id -u user >/dev/null 2>&1; then \
        useradd --create-home --shell /bin/bash user; \
    fi; \
    # Configure sudo access
    usermod -aG sudo user; \
    install -d -m 0755 /etc/sudoers.d /home/user/upload /home/user/go; \
    echo 'user ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/hackerai-user; \
    chmod 0440 /etc/sudoers.d/hackerai-user; \
    # Set ownership
    user_group="$(id -gn user)"; \
    chown -R "user:${user_group}" /home/user

# Set working directory
WORKDIR /home/user

# Configure system-wide PATH
RUN echo 'export PATH=/home/user/.local/bin:/usr/local/go/bin:/home/user/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin' \
    > /etc/profile.d/00-hackerai-path.sh && \
    chmod 644 /etc/profile.d/00-hackerai-path.sh

# ============================================================================
# Stage 5: Install Python Packages
# ============================================================================

# Document generation packages
RUN pip3 install --break-system-packages --no-cache-dir \
    reportlab \
    python-docx \
    openpyxl \
    python-pptx \
    pandas \
    pypandoc \
    odfpy

# Security-focused packages
RUN pip3 install --break-system-packages --no-cache-dir --ignore-installed 'rich<14,>=10' && \
    pip3 install --break-system-packages --no-cache-dir \
    requests \
    aiohttp \
    jinja2 \
    'httpx[cli]' \
    ratelimit \
    pycryptodomex

# ============================================================================
# Stage 6: Install Git-based Security Tools
# ============================================================================
RUN set -eux; \
    # JWT analysis tool
    git clone https://github.com/ticarpi/jwt_tool.git /opt/jwt_tool || true; \
    if [ -f /opt/jwt_tool/jwt_tool.py ]; then \
        chmod +x /opt/jwt_tool/jwt_tool.py; \
    fi; \
    # Git repository tools
    git clone https://github.com/internetwache/GitTools.git /opt/gittools && \
    chmod +x /opt/gittools/Dumper/gitdumper.sh /opt/gittools/Extractor/extractor.sh && \
    ln -sf /opt/gittools/Dumper/gitdumper.sh /usr/local/bin/gitdumper && \
    ln -sf /opt/gittools/Extractor/extractor.sh /usr/local/bin/gitextractor

# ============================================================================
# Stage 7: Install Binary Tools
# ============================================================================
RUN set -eux; \
    # TruffleHog - Secret scanning
    curl -sSfL https://raw.githubusercontent.com/trufflesecurity/trufflehog/main/scripts/install.sh | sh -s -- -b /usr/local/bin; \
    # Trivy - Vulnerability scanner
    curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin; \
    # HTTPx - HTTP probing with architecture detection
    arch=$(uname -m); \
    case "$arch" in \
        x86_64) platform="linux_amd64" ;; \
        aarch64|arm64) platform="linux_arm64" ;; \
        *) platform="linux_amd64" ;; \
    esac; \
    HTTPX_URL=$(curl -s https://api.github.com/repos/projectdiscovery/httpx/releases/latest | \
        grep "browser_download_url" | cut -d '"' -f4 | grep "${platform}.zip$" | head -n1); \
    test -n "$HTTPX_URL"; \
    wget -qO /tmp/httpx.zip "$HTTPX_URL"; \
    mkdir -p /tmp/httpx_extracted; \
    unzip -q /tmp/httpx.zip -d /tmp/httpx_extracted; \
    httpx_bin=$(find /tmp/httpx_extracted -type f -name httpx | head -n1); \
    test -n "$httpx_bin"; \
    mv "$httpx_bin" /usr/local/bin/httpx; \
    chmod +x /usr/local/bin/httpx; \
    rm -rf /tmp/httpx_extracted /tmp/httpx.zip

# ============================================================================
# Stage 8: Create Tool Wrappers
# ============================================================================
RUN set -eux; \
    # JWT Tool wrapper
    if [ -f /opt/jwt_tool/jwt_tool.py ]; then \
        cat > /usr/local/bin/jwt-tool << 'EOF'
#!/bin/bash
exec python3 /opt/jwt_tool/jwt_tool.py "$@"
EOF
        chmod +x /usr/local/bin/jwt-tool; \
    fi

# ============================================================================
# Stage 9: Browser Automation Setup
# ============================================================================
ENV AGENT_BROWSER_EXECUTABLE_PATH=/usr/bin/chromium
ENV AGENT_BROWSER_USER_AGENT="Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
ENV AGENT_BROWSER_ARGS="--disable-blink-features=AutomationControlled,--no-first-run,--no-default-browser-check,--disable-dev-shm-usage,--no-sandbox,--lang=en-US"
ENV AGENT_BROWSER_SCREENSHOT_DIR=/home/user/agent-browser-screenshots

RUN npm install -g --cache /tmp/npm-cache agent-browser@0.26.0 && \
    rm -rf /tmp/npm-cache && \
    mkdir -p /home/user/agent-browser-screenshots && \
    chown -R user:user /home/user/agent-browser-screenshots && \
    agent-browser doctor --offline --quick

# ============================================================================
# Stage 10: Install Go-based Tools
# ============================================================================
RUN set -eux; \
    go install github.com/projectdiscovery/interactsh/cmd/interactsh-client@latest && \
    go install github.com/projectdiscovery/katana/cmd/katana@latest && \
    go install github.com/projectdiscovery/cvemap/cmd/cvemap@latest && \
    go clean -cache -modcache && \
    rm -rf "$GOCACHE" "$GOPATH/pkg/mod"

# ============================================================================
# Stage 11: TestSSL Symlink
# ============================================================================
RUN set -eux; \
    if command -v testssl >/dev/null 2>&1; then \
        ln -sf "$(which testssl)" /usr/local/bin/testssl.sh; \
    fi

# ============================================================================
# Stage 12: Update Nuclei Templates
# ============================================================================
RUN nuclei -update-templates

# ============================================================================
# Stage 13: Tool Validation
# ============================================================================
RUN set -eux; \
    echo "=== Starting Tool Validation ===" && \
    # Validate cleanup
    test -z "$(find /var/lib/apt/lists -mindepth 1 -print -quit)" && \
    test -z "$(find /var/cache/apt/archives -mindepth 1 -print -quit)" && \
    test ! -e /home/user/.cache/pip && \
    test ! -e /tmp/npm-cache && \
    test ! -e "$GOCACHE" && \
    test ! -e "$GOPATH/pkg/mod" && \
    echo "✓ Cache cleanup verified" && \
    # Validate sudoers
    visudo -cf /etc/sudoers.d/hackerai-user && \
    echo "✓ sudoers configuration" && \
    # Validate PATH
    grep -Fx 'export PATH=/home/user/.local/bin:/usr/local/go/bin:/home/user/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin' \
        /etc/profile.d/00-hackerai-path.sh && \
    echo "✓ Login shell PATH" && \
    # Validate core tools
    test -d /usr/share/seclists && echo "✓ SecLists" && \
    # Network scanning
    which nmap && echo "✓ nmap" && \
    which naabu && echo "✓ naabu" && \
    which nikto && echo "✓ nikto" && \
    # Web recon
    which whatweb && echo "✓ whatweb" && \
    which wafw00f && echo "✓ wafw00f" && \
    # Web vuln scanners
    which sqlmap && echo "✓ sqlmap" && \
    which wapiti && echo "✓ wapiti" && \
    which wpscan && echo "✓ wpscan" && \
    # DNS tools
    which subfinder && echo "✓ subfinder" && \
    which dnsrecon && echo "✓ dnsrecon" && \
    which dnsenum && echo "✓ dnsenum" && \
    which whois && echo "✓ whois" && \
    # Fuzzing
    which ffuf && echo "✓ ffuf" && \
    which arjun && echo "✓ arjun" && \
    which gobuster && echo "✓ gobuster" && \
    which dirsearch && echo "✓ dirsearch" && \
    # SSL
    which testssl && echo "✓ testssl" && \
    which testssl.sh && echo "✓ testssl.sh" && \
    # Scanning
    which nuclei && echo "✓ nuclei" && \
    which hydra && echo "✓ hydra" && \
    # SMB
    which smbclient && echo "✓ smbclient" && \
    which smbmap && echo "✓ smbmap" && \
    which nbtscan && echo "✓ nbtscan" && \
    which enum4linux && echo "✓ enum4linux" && \
    # Network
    which arp-scan && echo "✓ arp-scan" && \
    # Crawling
    which gospider && echo "✓ gospider" && \
    # Go tools
    which interactsh-client && echo "✓ interactsh-client" && \
    which katana && echo "✓ katana" && \
    which cvemap && echo "✓ cvemap" && \
    # Custom tools
    which jwt-tool && jwt-tool -h >/dev/null && echo "✓ jwt_tool" && \
    which gitdumper && echo "✓ gitdumper" && \
    # External binaries
    which trufflehog && echo "✓ trufflehog" && \
    which trivy && echo "✓ trivy" && \
    # Frameworks
    which zaproxy && echo "✓ zaproxy" && \
    which msfconsole && echo "✓ msfconsole" && \
    which msfpc && echo "✓ msfpc" && \
    # HTTP tools
    which httpx && echo "✓ httpx" && \
    # Browser
    which chromium && echo "✓ chromium" && \
    which agent-browser && echo "✓ agent-browser" && \
    # Utilities
    which rg && echo "✓ ripgrep" && \
    # Languages
    which go && echo "✓ go" && \
    which python3 && echo "✓ python3" && \
    which node && echo "✓ node" && \
    which ruby && echo "✓ ruby" && \
    which tmux && echo "✓ tmux" && \
    # Python modules
    python3 -c "import reportlab; print('✓ reportlab')" && \
    python3 -c "import docx; print('✓ python-docx')" && \
    python3 -c "import openpyxl; print('✓ openpyxl')" && \
    python3 -c "import pptx; print('✓ python-pptx')" && \
    python3 -c "import pandas; print('✓ pandas')" && \
    python3 -c "import odf; print('✓ odfpy')" && \
    python3 -c "import requests; print('✓ requests')" && \
    # Version information
    echo "=== Version Information ===" && \
    nmap --version | head -n1 && \
    python3 --version && \
    go version && \
    node --version && \
    ruby --version && \
    echo "=== All Critical Tools Validated Successfully ==="

# ============================================================================
# Stage 14: Final Cleanup & Permissions
# ============================================================================
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /var/cache/apt/archives/* && \
    chown -R user:user /home/user

# ============================================================================
# Default Shell
# ============================================================================
SHELL ["/bin/bash", "-c"]
