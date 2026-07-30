
FROM kalilinux/kali-rolling

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
# Base System Setup (use default Kali CDN mirror - auto-routes to healthy mirrors)
# ============================================================================
RUN apt-get update && \
    apt-get install -y kali-archive-keyring sudo && \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

# ============================================================================
# Core System Packages
# ============================================================================
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    # Essential utilities
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
    # Build tools
    build-essential \
    gcc \
    g++ \
    make \
    cmake \
    pkg-config \
    binutils \
    # Python stack
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
    # Ruby stack
    ruby-full \
    ruby-dev \
    libnet-ssleay-perl \
    libio-socket-ssl-perl \
    libcrypt-ssleay-perl \
    libssl-dev \
    # Network utilities
    iputils-ping \
    dnsutils \
    iproute2 \
    net-tools \
    whois \
    traceroute \
    netcat-traditional \
    tcpdump \
    # Search tools
    ripgrep \
    # Text editors
    vim \
    nano \
    less \
    # Network scanning & enumeration (from Kali repos - pre-compiled!)
    nmap \
    naabu \
    nikto \
    whatweb \
    wafw00f \
    # Web vulnerability scanners
    sqlmap \
    wapiti \
    wpscan \
    # DNS/subdomain enumeration
    subfinder \
    dnsrecon \
    dnsenum \
    # Web fuzzing & discovery
    ffuf \
    arjun \
    gobuster \
    dirsearch \
    # SMB/NetBIOS tools
    smbclient \
    smbmap \
    nbtscan \
    python3-impacket \
    enum4linux \
    # Network discovery
    hping3 \
    arp-scan \
    # Utilities
    socat \
    proxychains4 \
    seclists \
    hashid \
    libimage-exiftool-perl \
    cewl \
    # SSL/TLS testing
    testssl.sh \
    # Web crawling & recon
    gospider \
    nuclei \
    # Authentication/bruteforce
    hydra \
    # Programming languages
    golang \
    nodejs \
    npm \
    tmux \
    chromium \
    fonts-liberation \
    # Forensics
    binwalk \
    foremost \
    # Document conversion
    pandoc \
    poppler-utils \
    # Web application security scanner
    zaproxy \
    && apt-get clean && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

# ============================================================================
# WhatWeb Ruby Library Symlink Fix
# ============================================================================
RUN set -eux; \
    if command -v whatweb >/dev/null 2>&1; then \
      mkdir -p /usr/bin/lib; \
      if ls /usr/lib/ruby/vendor_ruby/*.rb >/dev/null 2>&1; then \
        ln -sf /usr/lib/ruby/vendor_ruby/*.rb /usr/bin/lib/; \
      fi; \
    fi

# ============================================================================
# User Setup
# ============================================================================
# Avoid escaped newline sequences in generated files: E2B's Dockerfile parser
# removes the backslash from RUN arguments.
RUN set -eux; \
    if ! id -u user >/dev/null 2>&1; then \
      useradd --create-home --shell /bin/bash user; \
    fi; \
    usermod -aG sudo user; \
    install -d -m 0755 /etc/sudoers.d /home/user/upload /home/user/go; \
    echo 'user ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/hackerai-user; \
    chmod 0440 /etc/sudoers.d/hackerai-user; \
    user_group="$(id -gn user)"; \
    chown -R "user:${user_group}" /home/user

WORKDIR /home/user

# Ensure login shells, including E2B's Dockerfile builder shell, keep system
# binary directories after adding user-local and Go bin paths.
RUN echo 'export PATH=/home/user/.local/bin:/usr/local/go/bin:/home/user/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin' \
    > /etc/profile.d/00-hackerai-path.sh && chmod 644 /etc/profile.d/00-hackerai-path.sh

# ============================================================================
# Python Packages for Document Generation & Security Tools
# ============================================================================
RUN pip3 install --break-system-packages --no-cache-dir \
    reportlab \
    python-docx \
    openpyxl \
    python-pptx \
    pandas \
    pypandoc \
    odfpy

# Additional security-focused Python packages
# Kali's python3-rich package may be newer than httpx[cli]'s rich<14
# requirement, and pip cannot uninstall dpkg-owned packages without RECORD
# metadata. Install a pip-managed compatible rich first so httpx[cli] resolves
# against /usr/local without touching the distro package.
RUN pip3 install --break-system-packages --no-cache-dir --ignore-installed 'rich<14,>=10' && \
    pip3 install --break-system-packages --no-cache-dir requests aiohttp jinja2 'httpx[cli]' ratelimit pycryptodomex

# ============================================================================
# Git-based Security Tools
# ============================================================================
RUN set -eux; \
    # JWT analysis tool
    git clone https://github.com/ticarpi/jwt_tool.git /opt/jwt_tool || true && \
    if [ -f /opt/jwt_tool/jwt_tool.py ]; then chmod +x /opt/jwt_tool/jwt_tool.py; fi && \
    # Git repository dumper
    git clone https://github.com/internetwache/GitTools.git /opt/gittools && \
    chmod +x /opt/gittools/Dumper/gitdumper.sh /opt/gittools/Extractor/extractor.sh && \
    ln -sf /opt/gittools/Dumper/gitdumper.sh /usr/local/bin/gitdumper && \
    ln -sf /opt/gittools/Extractor/extractor.sh /usr/local/bin/gitextractor

# ============================================================================
# Binary Downloads (Platform-aware)
# ============================================================================
RUN set -eux; \
    # Secret scanning tool
    curl -sSfL https://raw.githubusercontent.com/trufflesecurity/trufflehog/main/scripts/install.sh | sh -s -- -b /usr/local/bin && \
    # Vulnerability scanner for containers and filesystems
    curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin && \
    # httpx with proper architecture detection
    arch=$(uname -m); \
    if [ "$arch" = "x86_64" ]; then platform="linux_amd64"; \
    elif [ "$arch" = "aarch64" ] || [ "$arch" = "arm64" ]; then platform="linux_arm64"; \
    else platform="linux_amd64"; fi; \
    HTTPX_URL=$(curl -s https://api.github.com/repos/projectdiscovery/httpx/releases/latest \
      | grep "browser_download_url" | cut -d '"' -f4 \
      | grep "${platform}.zip$" | head -n1); \
    test -n "$HTTPX_URL"; \
    echo "Fetching httpx: $HTTPX_URL"; \
    wget -qO /tmp/httpx.zip "$HTTPX_URL"; \
    mkdir -p /tmp/httpx_extracted; \
    unzip -q /tmp/httpx.zip -d /tmp/httpx_extracted; \
    httpx_bin=$(find /tmp/httpx_extracted -type f -name httpx | head -n1); \
    test -n "$httpx_bin"; \
    mv "$httpx_bin" /usr/local/bin/httpx; \
    chmod +x /usr/local/bin/httpx; \
    rm -rf /tmp/httpx_extracted /tmp/httpx.zip

# ============================================================================
# Create Wrapper Scripts for Python Tools
# ============================================================================
RUN set -eux; \
    # JWT Tool wrapper
    ([ -f /opt/jwt_tool/jwt_tool.py ] && \
      { \
        echo '#!/bin/bash'; \
        echo 'exec python3 /opt/jwt_tool/jwt_tool.py "$@"'; \
      } > /usr/local/bin/jwt-tool && \
      chmod +x /usr/local/bin/jwt-tool) || true

# ============================================================================
# Browser Automation
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
# Go-based Security Tools
# ============================================================================
RUN set -eux; \
    go install github.com/projectdiscovery/interactsh/cmd/interactsh-client@latest && \
    go install github.com/projectdiscovery/katana/cmd/katana@latest && \
    go install github.com/projectdiscovery/cvemap/cmd/cvemap@latest && \
    go clean -cache -modcache && \
    rm -rf "$GOCACHE" "$GOPATH/pkg/mod"

# ============================================================================
# TestSSL Symlink - Allow access via both 'testssl' and 'testssl.sh'
# ============================================================================
RUN set -eux; \
    if command -v testssl >/dev/null 2>&1; then \
      ln -sf $(which testssl) /usr/local/bin/testssl.sh; \
    fi

# ============================================================================
# Update Nuclei Templates
# ============================================================================
RUN nuclei -update-templates

# ============================================================================
# Tool Validation
# ============================================================================
RUN set -eux; \
    echo "=== Starting tool validation ===" && \
    test -z "$(find /var/lib/apt/lists -mindepth 1 -print -quit)" && \
    test -z "$(find /var/cache/apt/archives -mindepth 1 -print -quit)" && \
    test ! -e /home/user/.cache/pip && \
    test ! -e /tmp/npm-cache && \
    test ! -e "$GOCACHE" && \
    test ! -e "$GOPATH/pkg/mod" && \
    visudo -cf /etc/sudoers.d/hackerai-user && \
    echo "✓ sudoers" && \
    grep -Fx 'export PATH=/home/user/.local/bin:/usr/local/go/bin:/home/user/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin' /etc/profile.d/00-hackerai-path.sh && \
    echo "✓ login shell PATH" && \
    test -d /usr/share/seclists && \
    echo "✓ SecLists installed" && \
    which nmap && echo "✓ nmap" && \
    which naabu && echo "✓ naabu" && \
    which nikto && echo "✓ nikto" && \
    which whatweb && echo "✓ whatweb" && \
    which wafw00f && echo "✓ wafw00f" && \
    which sqlmap && echo "✓ sqlmap" && \
    which wapiti && echo "✓ wapiti" && \
    which wpscan && echo "✓ wpscan" && \
    which subfinder && echo "✓ subfinder" && \
    which dnsrecon && echo "✓ dnsrecon" && \
    which dnsenum && echo "✓ dnsenum" && \
    which whois && echo "✓ whois" && \
    which ffuf && echo "✓ ffuf" && \
    which arjun && echo "✓ arjun" && \
    which gobuster && echo "✓ gobuster" && \
    which dirsearch && echo "✓ dirsearch" && \
    which testssl && echo "✓ testssl" && \
    which testssl.sh && echo "✓ testssl.sh" && \
    which nuclei && echo "✓ nuclei" && \
    which hydra && echo "✓ hydra" && \
    which smbclient && echo "✓ smbclient" && \
    which smbmap && echo "✓ smbmap" && \
    which nbtscan && echo "✓ nbtscan" && \
    which enum4linux && echo "✓ enum4linux" && \
    which arp-scan && echo "✓ arp-scan" && \
    which gospider && echo "✓ gospider" && \
    which interactsh-client && echo "✓ interactsh-client" && \
    which katana && echo "✓ katana" && \
    which cvemap && echo "✓ cvemap" && \
    which jwt-tool && jwt-tool -h >/dev/null && echo "✓ jwt_tool" && \
    which gitdumper && echo "✓ gitdumper" && \
    which trufflehog && echo "✓ trufflehog" && \
    which trivy && echo "✓ trivy" && \
    which zaproxy && echo "✓ zaproxy" && \
    which httpx && echo "✓ httpx" && \
    which chromium && echo "✓ chromium" && \
    which agent-browser && echo "✓ agent-browser" && \
    which rg && echo "✓ ripgrep" && \
    which go && echo "✓ go" && \
    which python3 && echo "✓ python3" && \
    which node && echo "✓ node" && \
    which ruby && echo "✓ ruby" && \
    which tmux && echo "✓ tmux" && \
    python3 -c "import reportlab; print('✓ reportlab')" && \
    python3 -c "import docx; print('✓ python-docx')" && \
    python3 -c "import openpyxl; print('✓ openpyxl')" && \
    python3 -c "import pptx; print('✓ python-pptx')" && \
    python3 -c "import pandas; print('✓ pandas')" && \
    python3 -c "import odf; print('✓ odfpy')" && \
    python3 -c "import requests; print('✓ requests')" && \
    echo "=== Version Information ===" && \
    nmap --version | head -n1 && \
    python3 --version && \
    go version && \
    node --version && \
    ruby --version && \
    echo "=== All critical tools validated successfully ==="

# ============================================================================
# Final Cleanup & Permissions
# ============================================================================
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /var/cache/apt/archives/* && \
    chown -R user:user /home/user

# Default shell
SHELL ["/bin/bash", "-c"]

+++ docker/Dockerfile (修改后)
     1	FROM kalilinux/kali-rolling
     2	
     3	LABEL org.opencontainers.image.title="HackerAI Agent Sandbox"
     4	LABEL org.opencontainers.image.description="AI Agent Penetration Testing Environment with Comprehensive Automated Tools"
     5	LABEL org.opencontainers.image.source="https://github.com/hackerai-tech/hackerai"
     6	LABEL org.opencontainers.image.vendor="HackerAI"
     7	
     8	# ============================================================================
     9	# Environment Variables
    10	# ============================================================================
    11	ENV DEBIAN_FRONTEND=noninteractive
    12	ENV HOME=/home/user
    13	ENV GOPATH=/home/user/go
    14	ENV GOCACHE=/home/user/go/.cache
    15	ENV PATH=/home/user/.local/bin:/usr/local/go/bin:/home/user/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    16	
    17	# ============================================================================
    18	# Base System Setup (use default Kali CDN mirror - auto-routes to healthy mirrors)
    19	# ============================================================================
    20	RUN apt-get update && \
    21	    apt-get install -y kali-archive-keyring sudo && \
    22	    apt-get update && \
    23	    apt-get upgrade -y && \
    24	    apt-get clean && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*
    25	
    26	# ============================================================================
    27	# Core System Packages
    28	# ============================================================================
    29	RUN apt-get update && \
    30	    apt-get install -y --no-install-recommends \
    31	    # Essential utilities
    32	    curl \
    33	    wget \
    34	    git \
    35	    ca-certificates \
    36	    gnupg2 \
    37	    unzip \
    38	    jq \
    39	    tree \
    40	    perl \
    41	    sudo \
    42	    # Build tools
    43	    build-essential \
    44	    gcc \
    45	    g++ \
    46	    make \
    47	    cmake \
    48	    pkg-config \
    49	    binutils \
    50	    # Python stack
    51	    python3 \
    52	    python3-venv \
    53	    python3-pip \
    54	    python3-dev \
    55	    python-is-python3 \
    56	    libffi-dev \
    57	    libxml2-dev \
    58	    libxslt1-dev \
    59	    zlib1g-dev \
    60	    libglib2.0-dev \
    61	    # Ruby stack
    62	    ruby-full \
    63	    ruby-dev \
    64	    libnet-ssleay-perl \
    65	    libio-socket-ssl-perl \
    66	    libcrypt-ssleay-perl \
    67	    libssl-dev \
    68	    # Network utilities
    69	    iputils-ping \
    70	    dnsutils \
    71	    iproute2 \
    72	    net-tools \
    73	    whois \
    74	    traceroute \
    75	    netcat-traditional \
    76	    tcpdump \
    77	    # Search tools
    78	    ripgrep \
    79	    # Text editors
    80	    vim \
    81	    nano \
    82	    less \
    83	    # Network scanning & enumeration (from Kali repos - pre-compiled!)
    84	    nmap \
    85	    naabu \
    86	    nikto \
    87	    whatweb \
    88	    wafw00f \
    89	    # Web vulnerability scanners
    90	    sqlmap \
    91	    wapiti \
    92	    wpscan \
    93	    # DNS/subdomain enumeration
    94	    subfinder \
    95	    dnsrecon \
    96	    dnsenum \
    97	    # Web fuzzing & discovery
    98	    ffuf \
    99	    arjun \
   100	    gobuster \
   101	    dirsearch \
   102	    # SMB/NetBIOS tools
   103	    smbclient \
   104	    smbmap \
   105	    nbtscan \
   106	    python3-impacket \
   107	    enum4linux \
   108	    # Network discovery
   109	    hping3 \
   110	    arp-scan \
   111	    # Utilities
   112	    socat \
   113	    proxychains4 \
   114	    seclists \
   115	    hashid \
   116	    libimage-exiftool-perl \
   117	    cewl \
   118	    # SSL/TLS testing
   119	    testssl.sh \
   120	    # Web crawling & recon
   121	    gospider \
   122	    nuclei \
   123	    # Authentication/bruteforce
   124	    hydra \
   125	    # Programming languages
   126	    golang \
   127	    nodejs \
   128	    npm \
   129	    tmux \
   130	    chromium \
   131	    fonts-liberation \
   132	    # Forensics
   133	    binwalk \
   134	    foremost \
   135	    # Document conversion
   136	    pandoc \
   137	    poppler-utils \
   138	    # Web application security scanner
   139	    zaproxy \
   140	    # Exploitation & post-exploitation
   141	    metasploit-framework \
   142	    msfpc \
   143	    && apt-get clean && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*
   144	
   145	# ============================================================================
   146	# WhatWeb Ruby Library Symlink Fix
   147	# ============================================================================
   148	RUN set -eux; \
   149	    if command -v whatweb >/dev/null 2>&1; then \
   150	      mkdir -p /usr/bin/lib; \
   151	      if ls /usr/lib/ruby/vendor_ruby/*.rb >/dev/null 2>&1; then \
   152	        ln -sf /usr/lib/ruby/vendor_ruby/*.rb /usr/bin/lib/; \
   153	      fi; \
   154	    fi
   155	
   156	# ============================================================================
   157	# User Setup
   158	# ============================================================================
   159	# Avoid escaped newline sequences in generated files: E2B's Dockerfile parser
   160	# removes the backslash from RUN arguments.
   161	RUN set -eux; \
   162	    if ! id -u user >/dev/null 2>&1; then \
   163	      useradd --create-home --shell /bin/bash user; \
   164	    fi; \
   165	    usermod -aG sudo user; \
   166	    install -d -m 0755 /etc/sudoers.d /home/user/upload /home/user/go; \
   167	    echo 'user ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/hackerai-user; \
   168	    chmod 0440 /etc/sudoers.d/hackerai-user; \
   169	    user_group="$(id -gn user)"; \
   170	    chown -R "user:${user_group}" /home/user
   171	
   172	WORKDIR /home/user
   173	
   174	# Ensure login shells, including E2B's Dockerfile builder shell, keep system
   175	# binary directories after adding user-local and Go bin paths.
   176	RUN echo 'export PATH=/home/user/.local/bin:/usr/local/go/bin:/home/user/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin' \
   177	    > /etc/profile.d/00-hackerai-path.sh && chmod 644 /etc/profile.d/00-hackerai-path.sh
   178	
   179	# ============================================================================
   180	# Python Packages for Document Generation & Security Tools
   181	# ============================================================================
   182	RUN pip3 install --break-system-packages --no-cache-dir \
   183	    reportlab \
   184	    python-docx \
   185	    openpyxl \
   186	    python-pptx \
   187	    pandas \
   188	    pypandoc \
   189	    odfpy
   190	
   191	# Additional security-focused Python packages
   192	# Kali's python3-rich package may be newer than httpx[cli]'s rich<14
   193	# requirement, and pip cannot uninstall dpkg-owned packages without RECORD
   194	# metadata. Install a pip-managed compatible rich first so httpx[cli] resolves
   195	# against /usr/local without touching the distro package.
   196	RUN pip3 install --break-system-packages --no-cache-dir --ignore-installed 'rich<14,>=10' && \
   197	    pip3 install --break-system-packages --no-cache-dir requests aiohttp jinja2 'httpx[cli]' ratelimit pycryptodomex
   198	
   199	# ============================================================================
   200	# Git-based Security Tools
   201	# ============================================================================
   202	RUN set -eux; \
   203	    # JWT analysis tool
   204	    git clone https://github.com/ticarpi/jwt_tool.git /opt/jwt_tool || true && \
   205	    if [ -f /opt/jwt_tool/jwt_tool.py ]; then chmod +x /opt/jwt_tool/jwt_tool.py; fi && \
   206	    # Git repository dumper
   207	    git clone https://github.com/internetwache/GitTools.git /opt/gittools && \
   208	    chmod +x /opt/gittools/Dumper/gitdumper.sh /opt/gittools/Extractor/extractor.sh && \
   209	    ln -sf /opt/gittools/Dumper/gitdumper.sh /usr/local/bin/gitdumper && \
   210	    ln -sf /opt/gittools/Extractor/extractor.sh /usr/local/bin/gitextractor
   211	
   212	# ============================================================================
   213	# Binary Downloads (Platform-aware)
   214	# ============================================================================
   215	RUN set -eux; \
   216	    # Secret scanning tool
   217	    curl -sSfL https://raw.githubusercontent.com/trufflesecurity/trufflehog/main/scripts/install.sh | sh -s -- -b /usr/local/bin && \
   218	    # Vulnerability scanner for containers and filesystems
   219	    curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin && \
   220	    # httpx with proper architecture detection
   221	    arch=$(uname -m); \
   222	    if [ "$arch" = "x86_64" ]; then platform="linux_amd64"; \
   223	    elif [ "$arch" = "aarch64" ] || [ "$arch" = "arm64" ]; then platform="linux_arm64"; \
   224	    else platform="linux_amd64"; fi; \
   225	    HTTPX_URL=$(curl -s https://api.github.com/repos/projectdiscovery/httpx/releases/latest \
   226	      | grep "browser_download_url" | cut -d '"' -f4 \
   227	      | grep "${platform}.zip$" | head -n1); \
   228	    test -n "$HTTPX_URL"; \
   229	    echo "Fetching httpx: $HTTPX_URL"; \
   230	    wget -qO /tmp/httpx.zip "$HTTPX_URL"; \
   231	    mkdir -p /tmp/httpx_extracted; \
   232	    unzip -q /tmp/httpx.zip -d /tmp/httpx_extracted; \
   233	    httpx_bin=$(find /tmp/httpx_extracted -type f -name httpx | head -n1); \
   234	    test -n "$httpx_bin"; \
   235	    mv "$httpx_bin" /usr/local/bin/httpx; \
   236	    chmod +x /usr/local/bin/httpx; \
   237	    rm -rf /tmp/httpx_extracted /tmp/httpx.zip
   238	
   239	# ============================================================================
   240	# Create Wrapper Scripts for Python Tools
   241	# ============================================================================
   242	RUN set -eux; \
   243	    # JWT Tool wrapper
   244	    ([ -f /opt/jwt_tool/jwt_tool.py ] && \
   245	      { \
   246	        echo '#!/bin/bash'; \
   247	        echo 'exec python3 /opt/jwt_tool/jwt_tool.py "$@"'; \
   248	      } > /usr/local/bin/jwt-tool && \
   249	      chmod +x /usr/local/bin/jwt-tool) || true
   250	
   251	# ============================================================================
   252	# Browser Automation
   253	# ============================================================================
   254	ENV AGENT_BROWSER_EXECUTABLE_PATH=/usr/bin/chromium
   255	ENV AGENT_BROWSER_USER_AGENT="Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
   256	ENV AGENT_BROWSER_ARGS="--disable-blink-features=AutomationControlled,--no-first-run,--no-default-browser-check,--disable-dev-shm-usage,--no-sandbox,--lang=en-US"
   257	ENV AGENT_BROWSER_SCREENSHOT_DIR=/home/user/agent-browser-screenshots
   258	
   259	RUN npm install -g --cache /tmp/npm-cache agent-browser@0.26.0 && \
   260	    rm -rf /tmp/npm-cache && \
   261	    mkdir -p /home/user/agent-browser-screenshots && \
   262	    chown -R user:user /home/user/agent-browser-screenshots && \
   263	    agent-browser doctor --offline --quick
   264	
   265	# ============================================================================
   266	# Go-based Security Tools
   267	# ============================================================================
   268	RUN set -eux; \
   269	    go install github.com/projectdiscovery/interactsh/cmd/interactsh-client@latest && \
   270	    go install github.com/projectdiscovery/katana/cmd/katana@latest && \
   271	    go install github.com/projectdiscovery/cvemap/cmd/cvemap@latest && \
   272	    go clean -cache -modcache && \
   273	    rm -rf "$GOCACHE" "$GOPATH/pkg/mod"
   274	
   275	# ============================================================================
   276	# TestSSL Symlink - Allow access via both 'testssl' and 'testssl.sh'
   277	# ============================================================================
   278	RUN set -eux; \
   279	    if command -v testssl >/dev/null 2>&1; then \
   280	      ln -sf $(which testssl) /usr/local/bin/testssl.sh; \
   281	    fi
   282	
   283	# ============================================================================
   284	# Update Nuclei Templates
   285	# ============================================================================
   286	RUN nuclei -update-templates
   287	
   288	# ============================================================================
   289	# Tool Validation
   290	# ============================================================================
   291	RUN set -eux; \
   292	    echo "=== Starting tool validation ===" && \
   293	    test -z "$(find /var/lib/apt/lists -mindepth 1 -print -quit)" && \
   294	    test -z "$(find /var/cache/apt/archives -mindepth 1 -print -quit)" && \
   295	    test ! -e /home/user/.cache/pip && \
   296	    test ! -e /tmp/npm-cache && \
   297	    test ! -e "$GOCACHE" && \
   298	    test ! -e "$GOPATH/pkg/mod" && \
   299	    visudo -cf /etc/sudoers.d/hackerai-user && \
   300	    echo "✓ sudoers" && \
   301	    grep -Fx 'export PATH=/home/user/.local/bin:/usr/local/go/bin:/home/user/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin' /etc/profile.d/00-hackerai-path.sh && \
   302	    echo "✓ login shell PATH" && \
   303	    test -d /usr/share/seclists && \
   304	    echo "✓ SecLists installed" && \
   305	    which nmap && echo "✓ nmap" && \
   306	    which naabu && echo "✓ naabu" && \
   307	    which nikto && echo "✓ nikto" && \
   308	    which whatweb && echo "✓ whatweb" && \
   309	    which wafw00f && echo "✓ wafw00f" && \
   310	    which sqlmap && echo "✓ sqlmap" && \
   311	    which wapiti && echo "✓ wapiti" && \
   312	    which wpscan && echo "✓ wpscan" && \
   313	    which subfinder && echo "✓ subfinder" && \
   314	    which dnsrecon && echo "✓ dnsrecon" && \
   315	    which dnsenum && echo "✓ dnsenum" && \
   316	    which whois && echo "✓ whois" && \
   317	    which ffuf && echo "✓ ffuf" && \
   318	    which arjun && echo "✓ arjun" && \
   319	    which gobuster && echo "✓ gobuster" && \
   320	    which dirsearch && echo "✓ dirsearch" && \
   321	    which testssl && echo "✓ testssl" && \
   322	    which testssl.sh && echo "✓ testssl.sh" && \
   323	    which nuclei && echo "✓ nuclei" && \
   324	    which hydra && echo "✓ hydra" && \
   325	    which smbclient && echo "✓ smbclient" && \
   326	    which smbmap && echo "✓ smbmap" && \
   327	    which nbtscan && echo "✓ nbtscan" && \
   328	    which enum4linux && echo "✓ enum4linux" && \
   329	    which arp-scan && echo "✓ arp-scan" && \
   330	    which gospider && echo "✓ gospider" && \
   331	    which interactsh-client && echo "✓ interactsh-client" && \
   332	    which katana && echo "✓ katana" && \
   333	    which cvemap && echo "✓ cvemap" && \
   334	    which jwt-tool && jwt-tool -h >/dev/null && echo "✓ jwt_tool" && \
   335	    which gitdumper && echo "✓ gitdumper" && \
   336	    which trufflehog && echo "✓ trufflehog" && \
   337	    which trivy && echo "✓ trivy" && \
   338	    which zaproxy && echo "✓ zaproxy" && \
   339	    which msfconsole && echo "✓ msfconsole" && \
   340	    which msfpc && echo "✓ msfpc" && \
   341	    which httpx && echo "✓ httpx" && \
   342	    which chromium && echo "✓ chromium" && \
   343	    which agent-browser && echo "✓ agent-browser" && \
   344	    which rg && echo "✓ ripgrep" && \
   345	    which go && echo "✓ go" && \
   346	    which python3 && echo "✓ python3" && \
   347	    which node && echo "✓ node" && \
   348	    which ruby && echo "✓ ruby" && \
   349	    which tmux && echo "✓ tmux" && \
   350	    python3 -c "import reportlab; print('✓ reportlab')" && \
   351	    python3 -c "import docx; print('✓ python-docx')" && \
   352	    python3 -c "import openpyxl; print('✓ openpyxl')" && \
   353	    python3 -c "import pptx; print('✓ python-pptx')" && \
   354	    python3 -c "import pandas; print('✓ pandas')" && \
   355	    python3 -c "import odf; print('✓ odfpy')" && \
   356	    python3 -c "import requests; print('✓ requests')" && \
   357	    echo "=== Version Information ===" && \
   358	    nmap --version | head -n1 && \
   359	    python3 --version && \
   360	    go version && \
   361	    node --version && \
   362	    ruby --version && \
   363	    echo "=== All critical tools validated successfully ==="
   364	
   365	# ============================================================================
   366	# Final Cleanup & Permissions
   367	# ============================================================================
   368	RUN apt-get clean && \
   369	    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /var/cache/apt/archives/* && \
   370	    chown -R user:user /home/user
   371	
   372	# Default shell
   373	SHELL ["/bin/bash", "-c"]
   374	
