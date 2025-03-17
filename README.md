# Ansible Demo with Docker Containers

This project demonstrates an **Ansible playbook** deploying a web server (Nginx) on multiple **Docker-based mini Linux servers**. It is useful for showcasing DevOps automation and infrastructure provisioning in technical interviews.

---

## **1. Prerequisites**
- Docker installed (`docker --version`)
- Ansible installed (`ansible --version`)
- SSH key pair for authentication (`~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`)

---

## **2. Start Mini Linux Servers Using Docker**

Run the following commands to create two Ubuntu-based containers simulating remote servers:

```bash
# Create a Docker network for Ansible communication
docker network create ansible-net

# Start the first container (server1)
docker run -d --name server1 --hostname server1 --privileged \
  -v ~/.ssh:/root/.ssh:ro \
  --network ansible-net \
  ubuntu:latest sleep infinity

# Start the second container (server2)
docker run -d --name server2 --hostname server2 --privileged \
  -v ~/.ssh:/root/.ssh:ro \
  --network ansible-net \
  ubuntu:latest sleep infinity
```

---

## **3. Install SSH and Python on Containers**

Ansible requires **SSH and Python** to run commands. Install them inside each container:

```bash
docker exec -it server1 bash -c "apt update && apt install -y openssh-server python3 sudo"
docker exec -it server2 bash -c "apt update && apt install -y openssh-server python3 sudo"


docker exec -it server1 bash -c "useradd -m -s /bin/bash ansibleuser && echo 'ansibleuser:password123' | chpasswd"
docker exec -it server2 bash -c "useradd -m -s /bin/bash ansibleuser && echo 'ansibleuser:password123' | chpasswd"

docker exec -it server1 bash -c "echo 'ansibleuser ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers"
docker exec -it server2 bash -c "echo 'ansibleuser ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers"
```

Start SSH inside the containers:

```bash
docker exec -it server1 service ssh start
docker exec -it server2 service ssh start
```

Retrieve the **IP addresses** of the containers:

```bash
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' server1
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' server2
```

Update the **inventory.ini** file with the correct IPs.

---

## **4. Configure Ansible Inventory**

Edit `inventory.ini` and replace `<IP_SERVER1>` and `<IP_SERVER2>` with the actual container IPs:

```ini
[webservers]
server1 ansible_host=<IP_SERVER1> ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa
server2 ansible_host=<IP_SERVER2> ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa
```

---

## **5. Run the Ansible Playbook**

Execute the playbook to configure the web servers:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

This will:
- Install **Nginx** on the Docker containers
- Deploy an Nginx configuration file
- Restart the Nginx service

---

## **6. Verify Nginx Deployment**

After the playbook runs, check if **Nginx is active** inside the containers:

```bash
docker exec -it server1 systemctl status nginx
docker exec -it server2 systemctl status nginx
```

To confirm Nginx is running, fetch the default **Nginx page**:

```bash
curl http://<IP_SERVER1>
curl http://<IP_SERVER2>
```

---

## **7. Cleanup After Demo**

To remove the Docker containers and free up resources:

```bash
docker rm -f server1 server2
docker network rm ansible-net
```

---

## **8. Project Structure**
```
.
├── inventory.ini
├── playbook.yml
├── roles/
│   ├── webserver/
│   │   ├── tasks/main.yml
│   │   ├── handlers/main.yml
│   │   ├── templates/nginx.conf.j2
└── README.md
```

---

## **Conclusion**
This project demonstrates how **Ansible automates server provisioning** using **Docker-based Linux containers** as a lightweight environment. This is useful for DevOps workflows, CI/CD pipelines, and technical demonstrations.

🚀 **Ready for your next interview or project!**
