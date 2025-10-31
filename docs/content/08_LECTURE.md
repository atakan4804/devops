# Lecture 8 – Deployment to Production

### Goals
- Deploy application and documentation containers to the production server
- Automate deployment through GitHub Actions via SSH
- Manage container lifecycle (stop, remove, and redeploy)
- Ensure secure connection using SSH key pairs

---

### Setup

**SSH Key Pair**
1. Generated a **team SSH key pair** using:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "team9"
   ```
2. **Private key** stored in GitHub Secrets as `SSH_PRIVATE_KEY`
3. **Public key** sent to the instructor for registration on the production server

---

### Pipeline Steps

#### 1. **SSH Connection Test**
Ensures connection to the production server works before deployment:
```yaml
- name: SSH connection test (print date)
  env:
    SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
  run: |
    echo "$SSH_PRIVATE_KEY" > key.pem
    chmod 600 key.pem
    ssh-keyscan -H 10.0.40.194 > known_hosts
    ssh -i key.pem -o UserKnownHostsFile=known_hosts svcgithub@10.0.40.194 "date"
```

#### 2. **Deployment to Production**
Automates full container deployment:
```yaml
- name: Deploy containers to production (Team 9)
  env:
    SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
  run: |
    echo "$SSH_PRIVATE_KEY" > key.pem
    chmod 600 key.pem
    ssh-keyscan -H 10.0.40.194 > known_hosts

    # Stop and remove running containers
    ssh -i key.pem -o UserKnownHostsFile=known_hosts svcgithub@10.0.40.194 '
      docker stop devops-app-team9 || true &&
      docker rm devops-app-team9 || true &&
      docker stop mydoc || true &&
      docker rm mydoc || true
    '

    # Clean up old images
    ssh -i key.pem -o UserKnownHostsFile=known_hosts svcgithub@10.0.40.194 '
      docker image prune -f
    '

    # Pull latest images from registry
    ssh -i key.pem -o UserKnownHostsFile=known_hosts svcgithub@10.0.40.194 '
      REG="10.0.40.193:5000"
      docker pull $REG/devops-app-team9:latest
      docker pull $REG/team9-docs:latest
    '

    # Run new containers
    ssh -i key.pem -o UserKnownHostsFile=known_hosts svcgithub@10.0.40.194 '
      REG="10.0.40.193:5000"
      docker run -d --restart always --name devops-app-team9 -p 1891:8080 $REG/devops-app-team9:latest &&
      docker run -d --restart always --name mydoc -p 1892:80 $REG/team9-docs:latest
    '
```

---

### Technical Details
- **Production server**: `10.0.40.194`
- **Registry**: `10.0.40.193:5000`
- **Application port**: `1891`
- **Documentation port**: `1892`
- **User**: `svcgithub`

---

---

### Result
**Lecture 8:** Complete CI/CD integration up to production — upon each successful build, both the application and documentation containers are automatically deployed to the production server and made available via:

- Application: `http://10.0.40.194:1891`
- Documentation: `http://10.0.40.194:1892`