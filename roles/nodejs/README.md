# Node.js Role

This role installs Node.js 22.x, npm 11, and PM2 for process management.

## What's Installed

- **Node.js 22.x** - Latest LTS version from NodeSource
- **npm 11** - Latest npm version
- **PM2** - Production process manager for Node.js applications

## Usage

### Running Next.js Applications

```bash
# Navigate to your Next.js project
cd /var/www/nextjs-app

# Install dependencies
npm install

# Build the application
npm run build

# Start with PM2
pm2 start npm --name "nextjs-app" -- start

# Or use ecosystem file
pm2 start ecosystem.config.js
```

### Running React Applications

```bash
# Navigate to your React project
cd /var/www/react-app

# Install dependencies
npm install

# Build the application
npm run build

# Install serve globally (if not already)
npm install -g serve

# Start with PM2
pm2 start serve --name "react-app" -- -s build -l 3001
```

### PM2 Commands

```bash
# List all processes
pm2 list

# Monitor processes
pm2 monit

# View logs
pm2 logs

# View logs for specific app
pm2 logs nextjs-app

# Restart application
pm2 restart nextjs-app

# Stop application
pm2 stop nextjs-app

# Delete application from PM2
pm2 delete nextjs-app

# Save PM2 process list
pm2 save

# Setup PM2 to start on system boot
pm2 startup systemd
# Then run the command it outputs
pm2 save
```

### Example Ecosystem File

See [pm2-example.config.js](files/pm2-example.config.js) for a complete example.

Copy it to your project:
```bash
cp /path/to/pm2-example.config.js /var/www/your-app/ecosystem.config.js
# Edit the file with your settings
nano /var/www/your-app/ecosystem.config.js
# Start with PM2
pm2 start ecosystem.config.js
```

## Configuration

Variables in [defaults/main.yml](defaults/main.yml):

```yaml
nodejs_version: "22"          # Node.js major version
npm_version: "11"             # npm version to install
pm2_install: true             # Install PM2
nodejs_global_packages:       # Additional global packages
  - pm2
```

## Environment Variables

Set environment variables for your app:

```bash
# Using ecosystem file (recommended)
# Edit ecosystem.config.js and add env section

# Or using PM2 CLI
pm2 start app.js --name myapp --env production
pm2 set myapp:env_var "value"
```

## Nginx Reverse Proxy

To serve your Node.js app through Nginx, add this to your Nginx config:

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Troubleshooting

### Check Node.js version
```bash
node --version
```

### Check npm version
```bash
npm --version
```

### Check PM2 version
```bash
pm2 --version
```

### PM2 not starting on boot
```bash
pm2 startup systemd
# Run the command it outputs
pm2 save
```

### Application crashes
```bash
# Check logs
pm2 logs your-app --lines 100

# Check error logs
pm2 logs your-app --err --lines 100
```
