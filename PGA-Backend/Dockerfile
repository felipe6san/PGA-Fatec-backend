FROM node:slim

# Instalar dependências para compilar módulos nativos + OpenSSH + Chromium para puppeteer
RUN apt-get update && apt-get install -y \
    python3 \
    make \
    g++ \
    wget \
    ca-certificates \
    openssh-server \
    fonts-liberation \
    libnss3 \
    libatk1.0-0 \
    libatk-bridge2.0-0 \
    libc6 \
    libcairo2 \
    libdbus-1-3 \
    libglib2.0-0 \
    libgtk-3-0 \
    libx11-xcb1 \
    libxcomposite1 \
    libxdamage1 \
    libxrandr2 \
    libgbm1 \
    libasound2 \
    libpangocairo-1.0-0 \
    libxss1 \
    libxkbcommon0 \
    libxrender1 \
    chromium \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Usar o Chromium do sistema no puppeteer (evita download de ~300MB no build)
ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true

# Configurar SSH para Azure App Service (porta 2222, senha "Docker!")
RUN mkdir -p /run/sshd \
    && echo "root:Docker!" | chpasswd \
    && sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config \
    && sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config \
    && echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config

EXPOSE 2222

WORKDIR /app

# Copiar arquivos de configuração
COPY package*.json ./
COPY prisma ./prisma/

# Instalar todas as dependências para build
RUN npm install

# Gerar Prisma Client
RUN npx prisma generate

# Copiar resto do código
COPY . .

# Construir aplicação
RUN npm run build

# Remover devDependencies após build e setar NODE_ENV
RUN npm prune --production
ENV NODE_ENV=production

EXPOSE 3000

CMD ["sh", "-c", "/usr/sbin/sshd && npx prisma migrate deploy && npm run start:prod"]
