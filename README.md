# Personal WSL2 Ubuntu Setup
This is my setup for local Ubuntu 24.04 using WSL2 where I store most of my repositories under `/projects/`. This will be updated time to time.

## Necessary Tools
### NVM
Taken from [here](https://github.com/nvm-sh/nvm).
`curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash`

### ZSH
1. Install Fira Code Nerd Font first [here](https://github.com/ryanoasis/nerd-fonts/releases).
2. `sudo apt install zsh`
3. `sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
4. `git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting`
5. `git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions`
6. `git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k`
7. `nano ~/.zshrc`
8. Replace everything with this [here](https://github.com/azri-cs/wsl2-setup/blob/main/.zshrc). The content might be different after running powerlevel10k configuration wizard.
9. Restart terminal.


### PPA from Ondřej Surý for handling multiple PHP versions
1. `sudo apt install software-properties-common && sudo add-apt-repository ppa:ondrej/php && sudo apt update`
2. `sudo apt-get install php8.5 php8.5-fpm && sudo apt-get install php8.5-mysql php8.5-mbstring php8.5-xml php8.5-gd php8.5-curl php8.5-zip php8.5-intl php8.5-bcmath php8.5-sqlite3`

Switching PHP version: `sudo update-alternatives --config php`

### MySQL/PostgreSQL
Taken from [here](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-22-04).
1. `sudo apt install mysql-server`
2. `sudo systemctl start mysql.service`
3. `sudo mysql_secure_installation`

MySQL No Root Password error?
1. `sudo mysql`
2. Inside MySQL, replace `password` with something else: `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';`

### Python
Ubuntu 24.04 ships with Python 3 pre-installed, so we only need to configure further. Further instructions [here](https://www.digitalocean.com/community/tutorials/how-to-install-python-3-and-set-up-a-programming-environment-on-ubuntu-22-04).
1. `sudo apt install -y python3-pip`
2. `sudo apt install -y build-essential libssl-dev libffi-dev python3-dev python3-venv`
3. (optional) install python packages by replacing `package_name`: `pip3 install package_name`

### Ruby
`sudo apt install ruby-full`

### Make
`sudo apt install make`

### Composer
Replace the installer checksum (SHA-384) from [here](https://composer.github.io/pubkeys.html).
1. `php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"`
2. `php -r "if (hash_file('sha384', 'composer-setup.php') === 'dac665fdc30fdd8ec78b38b9800061b4150413ff2e3b6f88543c636f7cd84f6db9189d43a81e5503cda447da73c7e5b6') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"`
3. `php composer-setup.php`
4. `php -r "unlink('composer-setup.php');"`
5. Making it globally available: `sudo mv composer.phar /usr/local/bin/composer`

### Global Laravel Create
1. `composer global require laravel/installer`
2. Now you can create new project using `laravel new project-name`.

### Redis Server
1. `sudo apt-get install lsb-release curl gpg`
2. `curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg`
3. `sudo chmod 644 /usr/share/keyrings/redis-archive-keyring.gpg`
4. `echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list`
5. `sudo apt-get update && sudo apt-get install redis`
6. `systemctl status redis.service` to check its status, it should be in green "active (running)".
7. After installing supervisor, create supervisord config file for redis-server at `/etc/supervisor/conf.d`. Refer `redis-supervisor.conf`.

### Mailpit
Mailpit is a local SMTP mail catcher with a web UI, useful for testing email-sending during development without delivering real mail. Repo [here](https://github.com/axllent/mailpit).
1. Download the latest Linux amd64 binary from [releases](https://github.com/axllent/mailpit/releases): `curl -fSsL -o /tmp/mailpit.tar.gz https://github.com/axllent/mailpit/releases/latest/download/mailpit-linux-amd64.tar.gz`
2. Extract and install: `sudo tar -xzf /tmp/mailpit.tar.gz -C /usr/local/bin mailpit && sudo chmod +x /usr/local/bin/mailpit`
3. Verify: `mailpit version`
4. After installing supervisor, create supervisord config file for mailpit at `/etc/supervisor/conf.d`. Refer `mailpit-supervisor.conf`.
5. `sudo supervisorctl reread && sudo supervisorctl update`
6. `sudo supervisorctl start "mailpit:*"`

By default Mailpit listens on `localhost:1025` (SMTP) and `http://localhost:8025/` (web UI). Both ports are forwarded to Windows by WSL2, so the web UI is reachable from your Windows browser.

#### Quick Usage
Point your application at the local SMTP server:
```
SMTP host:  localhost
SMTP port:  1025
Username:   (none)
Password:   (none)
Encryption: NONE
```
Open `http://localhost:8025/` to read captured mail in the web UI.

### Supervisor
1. `sudo apt update && sudo apt install supervisor`
2. `sudo systemctl status supervisor` to check its status, it should be in green "active (running)".
3. After creating new config file for a program, run these `sudo supervisorctl reread && sudo supervisorctl update`  to retrieve latest config files & insert them to process group.
4. `sudo supervisorctl start "redis:*"`
5. Have `redis: ERROR (spawn error)` error? Ensure ownership of `/var/log/redis/redis.log` is correct: `sudo ls -l /var/log/redis/redis.log`. If not 'redis' user, run this `sudo chown redis:redis /var/log/redis/redis.log`.

### Google Chrome
For headless PDF generation, web scraping & automation, testing frontends, performance & accessibility audits, and web compatibility testing.
1. Initial Packages `sudo apt install curl software-properties-common apt-transport-https ca-certificates -y`
2. Import Google Chrome GPG Key `curl -fSsL https://dl.google.com/linux/linux_signing_key.pub | gpg --dearmor | sudo tee /usr/share/keyrings/google-chrome.gpg > /dev/null`
3. Import Google Chrome APT Repository `echo deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome.gpg] http://dl.google.com/linux/chrome/deb/ stable main | sudo tee /etc/apt/sources.list.d/google-chrome.list`
4. `sudo apt update`
5. `sudo apt install google-chrome-stable`
6. `google-chrome --version`

### Docling (IBM Document Understanding)
Docling parses PDF, DOCX, PPTX, images, HTML, and other formats into machine-readable markdown/JSON — great for RAG pipelines, document processing, and AI workflows. Repo [here](https://github.com/docling-project/docling).

1. Install pipx (Python application manager, avoids PEP 668 conflicts):  
   `sudo apt update && sudo apt install pipx`
2. Install Tesseract OCR (for image-based PDFs and scanned documents):  
   `sudo apt install tesseract-ocr libtesseract-dev`
3. Install Docling via pipx with CPU-only PyTorch (for WSL2 without GPU passthrough):  
   `pipx install docling --pip-args="--extra-index-url https://download.pytorch.org/whl/cpu"`
4. Verify installation:  
   `docling --version`

> **Note:** If you have a GPU available in WSL2, omit the `--pip-args` flag to install the default CUDA-enabled PyTorch variant.

#### Quick Usage
Convert a PDF to markdown:  
`docling myfile.pdf --to md`

Convert with OCR for scanned documents:  
`docling scanned-doc.pdf --to md --ocr`

### GitHub CLI (gh)
Work with pull requests, issues, and GitHub Actions from the terminal. Repo [here](https://github.com/cli/cli).
1. Add the official apt repository and install:

```bash
(type -p wget >/dev/null || (sudo apt update && sudo apt install wget -y)) \
	&& sudo mkdir -p -m 755 /etc/apt/keyrings \
	&& out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
	&& cat $out | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
	&& sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
	&& sudo mkdir -p -m 755 /etc/apt/sources.list.d \
	&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
	&& sudo apt update \
	&& sudo apt install gh -y
```

2. Authenticate: `gh auth login`
3. Verify: `gh --version`

### Modern CLI Essentials
Faster, friendlier replacements for the classic Unix tools: `fzf` (fuzzy finder), `fd` (find), `bat` (cat with syntax highlighting), `eza` (ls), `zoxide` (smarter cd), `git-delta` (nicer git diffs), `btop` (resource monitor), plus `sqlite3`, `shellcheck`, and `shfmt`.
1. `sudo apt update && sudo apt install fzf fd-find bat eza zoxide git-delta btop sqlite3 shellcheck shfmt -y`
2. On Ubuntu, `fd` and `bat` are installed as `fdfind` and `batcat`. Symlink them into `/usr/local/bin` so the standard command names work: `sudo ln -sf $(which fdfind) /usr/local/bin/fd && sudo ln -sf $(which batcat) /usr/local/bin/bat`
3. Activate zoxide by adding to `~/.zshrc`: `eval "$(zoxide init zsh)"`
5. Activate delta as git's pager:
   1. `git config --global core.pager delta`
   2. `git config --global interactive.diffFilter "delta --color-only"`

### yq
jq-style processing for YAML, XML and TOML files. Repo [here](https://github.com/mikefarah/yq).
1. `sudo wget https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 -O /usr/local/bin/yq && sudo chmod +x /usr/local/bin/yq`
2. Verify: `yq --version`

### xh
Fast, friendly HTTP client for testing APIs — a Rust reimplementation of HTTPie. Repo [here](https://github.com/ducaale/xh).
1. `curl -sfL https://raw.githubusercontent.com/ducaale/xh/master/install.sh | sh`
2. Verify: `xh --version`

#### Quick Usage
```
xh GET http://localhost:8000/api/users
xh POST http://localhost:8000/api/login username=me password=secret
```

> Prefer the original HTTPie instead? Install with `sudo apt install httpie` (command: `http`).

### websocat & wscat
WebSocket clients for the terminal — handy for testing sockets (Laravel Echo/Pusher, streaming APIs). [websocat](https://github.com/vi/websocat) is a powerful netcat-style client; [wscat](https://github.com/websockets/wscat) is a lightweight npm alternative.
1. websocat (single static binary): `curl -fSsL -o /tmp/websocat https://github.com/vi/websocat/releases/latest/download/websocat.x86_64-unknown-linux-musl && sudo install -m 755 /tmp/websocat /usr/local/bin/websocat`
2. wscat: `npm install -g wscat`
3. Verify: `websocat --version` / `wscat --version`

#### Quick Usage
`wscat -c ws://localhost:6001` or `websocat ws://localhost:6001`

### Bun
JavaScript/TypeScript runtime, package manager, bundler and test runner in one — runs `.ts` files directly with no build step. Repo [here](https://github.com/oven-sh/bun).
1. `curl -fsSL https://bun.sh/install | bash` (run as your user, not with sudo)
2. Add to PATH in `~/.zshrc`: `export PATH="$HOME/.bun/bin:$PATH"` (already in this repo's `.zshrc`)
3. Restart terminal. Verify: `bun --version`

### Lazygit
Terminal UI for git: stage hunks, commit, branch and rebase interactively. Repo [here](https://github.com/jesseduffield/lazygit).
1. `LAZYGIT_VERSION=$(curl -s "https://api.github.com/repos/jesseduffield/lazygit/releases/latest" | grep -Po '"tag_name": *"v\K[^"]*')`
2. `curl -Lo lazygit.tar.gz "https://github.com/jesseduffield/lazygit/releases/download/v${LAZYGIT_VERSION}/lazygit_${LAZYGIT_VERSION}_Linux_x86_64.tar.gz"`
3. `sudo tar -xf lazygit.tar.gz -C /usr/local/bin lazygit`
4. Verify: `lazygit --version`

### act
Run GitHub Actions workflows locally in Docker. Repo [here](https://github.com/nektos/act).
1. `curl -s https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash -s -- -b /usr/local/bin`
2. Verify: `act --version` (requires Docker Desktop to be running)

### k6
Load testing scriptable in JavaScript. Repo [here](https://github.com/grafana/k6).
1. `curl -fsSL https://dl.k6.io/key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/k6-archive-keyring.gpg`
2. `echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list`
3. `sudo apt-get update && sudo apt-get install k6`
4. Verify: `k6 version`

### Gitleaks
Scans git history for committed secrets and API keys. Repo [here](https://github.com/gitleaks/gitleaks).
1. `GITLEAKS_VERSION=$(curl -s "https://api.github.com/repos/gitleaks/gitleaks/releases/latest" | grep -Po '"tag_name": *"v\K[^"]*')`
2. `curl -fSsL -o /tmp/gitleaks.tar.gz "https://github.com/gitleaks/gitleaks/releases/download/v${GITLEAKS_VERSION}/gitleaks_${GITLEAKS_VERSION}_linux_x64.tar.gz"`
3. `sudo tar -xzf /tmp/gitleaks.tar.gz -C /usr/local/bin gitleaks && sudo chmod +x /usr/local/bin/gitleaks`
4. Verify: `gitleaks version`
5. Scan a repo: `gitleaks detect --source .`

> Building from source (`git clone https://github.com/gitleaks/gitleaks.git && cd gitleaks && make build`) also works, but requires Go (`golang-go`) and is much slower than downloading the release binary.

### mkcert
Locally-trusted TLS certificates — no more browser certificate warnings on `https://localhost` or `*.test` domains during development. Repo [here](https://github.com/FiloSottile/mkcert).
1. `sudo apt install mkcert` (in the Ubuntu 24.04 repos; on 22.04 download the binary from [releases](https://github.com/FiloSottile/mkcert/releases))
2. Install the local CA into the system trust store: `mkcert -install`
3. Create a cert for a dev domain: `mkcert myapp.test "*.myapp.test" localhost`
4. Point your web/dev server config at the generated `-key.pem` and `.pem` files.

### Caddy
Web server and reverse proxy with automatic HTTPS — handy in front of local dev servers. Repo [here](https://github.com/caddyserver/caddy).
1. `sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl`
2. `curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg`
3. `curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list`
4. `sudo apt update && sudo apt install caddy`

The package auto-starts a `caddy` service on port 80. For occasional development use, disable the service and run Caddy per-project instead:
1. `sudo systemctl disable --now caddy`
2. Example — proxy `localhost:20100` to a dev server on port 3000: `caddy reverse-proxy --from localhost:20100 --to localhost:3000`

> mkcert vs Caddy: mkcert issues certificates you hand to other servers (nginx, Vite, etc.); Caddy ships its own local CA (`caddy trust`). You usually only need one of the two.
