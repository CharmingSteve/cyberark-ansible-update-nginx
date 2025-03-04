```markdown
# Nginx Configuration Management Playbook

This Ansible playbook manages Nginx configuration files, supporting both local and remote configuration sources.

## Features

- Switch between local and remote configuration files using a boolean flag
- Error handling for failed downloads
- Graceful reload of Nginx (instead of restart)
- Supports both regular servers and containers

## Usage

### Prerequisites

1. Ansible installed on control machine
2. Target hosts have Nginx installed
3. SSH access to target hosts

### Inventory Setup

Add to your inventory file (`/etc/ansible/hosts`):
```ini
[web_servers]
webserver1 ansible_host=172.21.0.56 ansible_user=your_user ansible_become_password=your_password
```

### Configuration Options

In the playbook:
```yaml
vars:
    use_remote_config: false  # Set to true for remote config
    nginx_local_path: "./nginx.conf"  # Path to local config
    nginx_remote_url: "https://raw.githubusercontent.com/nginx/nginx/master/conf/nginx.conf"
```

### Running the Playbook

```bash
ansible-playbook nginx-conf-playbook.yml
```

Add `-v` for verbose output:
```bash
ansible-playbook nginx-conf-playbook.yml -v
```

## Future Enhancements

### 1. Configuration Backup
```yaml
- name: Backup existing nginx config
  copy:
    src: /etc/nginx/nginx.conf
    dest: /etc/nginx/nginx.conf.backup.{{ ansible_date_time.epoch }}
    remote_src: yes
```

### 2. Jinja2 Templates
```yaml
# template file: nginx.conf.j2
worker_processes {{ nginx_worker_processes }};

# playbook task:
- name: Template nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
```

### 3. Health Check
```yaml
- name: Verify NGINX is running
  uri:
    url: http://localhost
    return_content: yes
  register: health_check
  failed_when: health_check.status != 200
```

## Notes

- Uses `reload` instead of `restart` for zero-downtime configuration changes
- Includes error handling for failed remote downloads
- Supports both regular servers and container environments
