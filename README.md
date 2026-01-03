# Ansible Infrastructure Setup

Ansible project for installing and configuring essential tools for Laravel development environment.

## Project Structure

```
ansible/
├── ansible.cfg           # Ansible configuration
├── inventory            # Server inventory
├── site.yml             # Main playbook
└── roles/
    ├── common/          # Base packages
    ├── docker/          # Docker & Docker Compose
    ├── nginx/           # Nginx web server
    ├── php/             # PHP-FPM & Composer
    ├── mariadb/         # MariaDB database
    ├── redis/           # Redis cache
    ├── jenkins/         # Jenkins CI/CD
    └── sonarqube/       # SonarQube code quality
```

## Requirements

- Ansible 2.9+
- Target server: Ubuntu 20.04/22.04
- SSH access with sudo privileges

## Configuration

### 1. Inventory

Edit the `inventory` file with your server information:

```ini
[app]
prod ansible_host=YOUR_SERVER_IP ansible_user=YOUR_USER ansible_ssh_private_key_file=~/.ssh/YOUR_KEY.pem
```

### 2. Variables

Each role has a `defaults/main.yml` file for customization:

**MariaDB** ([roles/mariadb/defaults/main.yml](roles/mariadb/defaults/main.yml)):
```yaml
mariadb_root_password: "YOUR_SECURE_PASSWORD"  # ⚠️ MUST CHANGE!
```

**PHP** ([roles/php/defaults/main.yml](roles/php/defaults/main.yml)):
```yaml
php_version: "8.2"  # Change version if needed
```

**Nginx** ([roles/nginx/defaults/main.yml](roles/nginx/defaults/main.yml)):
```yaml
nginx_server_name: "example.com"  # Your domain
nginx_root_path: "/var/www/laravel/public"
```

## Usage

### Run full playbook

```bash
ansible-playbook site.yml
```

### Run specific roles with tags

```bash
# Install Docker only
ansible-playbook site.yml --tags docker

# Install Nginx and PHP only
ansible-playbook site.yml --tags nginx,php

# Install Jenkins only
ansible-playbook site.yml --tags jenkins
```

### Skip unnecessary roles

```bash
# Skip Jenkins and SonarQube
ansible-playbook site.yml --skip-tags jenkins,sonarqube
```

### Dry run (check before applying)

```bash
ansible-playbook site.yml --check --diff
```

## Roles

### Common
- Update & upgrade system
- Install base packages: curl, git, unzip, htop, vim, wget

### Docker
- Install Docker and Docker Compose
- Add user to docker group
- Auto-start Docker service

### Nginx
- Install Nginx web server
- Configure Laravel vhost
- Remove default site

### PHP
- Install PHP 8.2 (version can be changed)
- Extensions: mysql, redis, xml, mbstring, curl, zip
- Install Composer globally

### MariaDB
- Install MariaDB server
- Security hardening:
  - Set root password
  - Remove anonymous users
  - Disable remote root login
  - Remove test database

### Redis
- Install Redis server
- Configure bind to localhost only
- Enable systemd supervision

### Jenkins
- Run Jenkins in Docker container
- Ports: 8080 (HTTP), 50000 (Agent)
- Data: `/opt/jenkins_home`

### SonarQube
- Run SonarQube in Docker container
- Port: 9000
- Data: `/opt/sonarqube/`

## Security

### ⚠️ IMPORTANT

1. **Change MariaDB password**: Edit `mariadb_root_password` in [roles/mariadb/defaults/main.yml](roles/mariadb/defaults/main.yml)

2. **Use Ansible Vault** to encrypt sensitive data:

```bash
# Create encrypted variable
ansible-vault encrypt_string 'MySecurePassword' --name 'mariadb_root_password'

# Run playbook with vault
ansible-playbook site.yml --ask-vault-pass
```

3. **SSH key permissions**: Ensure private key has `0600` permissions

```bash
chmod 600 ~/.ssh/ansible.pem
```

## Troubleshooting

### Connection timeout
```bash
# Test SSH connection
ansible app -m ping

# With verbose output
ansible app -m ping -vvv
```

### Permission denied
```bash
# Check sudo access
ansible app -m shell -a "sudo whoami" --become
```

### Role dependency issues
```bash
# Re-run with force
ansible-playbook site.yml --force-handlers
```

## Best Practices

1. **Backup before running** - Especially on production servers
2. **Test on staging** - Try on test environment first
3. **Use tags** - Only run what's necessary
4. **Check mode** - Always use `--check` before applying
5. **Version control** - Commit changes to Git

## License

MIT
