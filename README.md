# MongoDB-Replica-Set-Deployment-with-Docker-Ansible


```markdown
# 📦 MongoDB Replica Set Deployment with Docker & Ansible

## ✅ Objective

Automate the deployment of a MongoDB replica set using Docker containers and Ansible playbooks to ensure:
- High availability
- Replica set configuration
- Security best practices

---

## 🗂️ Project Directory Structure

```bash
mongodb-replica-ansible/
├── docker-compose.yml
├── inventory/
│   └── hosts.ini
├── playbooks/
│   ├── setup-replica.yml
│   ├── create-user.yml
│   └── common.yml
├── roles/
│   └── mongodb/
│       ├── tasks/
│       │   ├── main.yml
│       │   ├── init_replica.yml
│       │   └── create_user.yml
│       └── templates/
│           └── mongod.conf.j2
└── README.md
```

---

**setup-file.py**

```python

import os

# Base path to create the project
base_path = "/home/lilia/VIDEOS/mongodb-replica-ansible"

# File paths and their contents
file_contents = {
    f"{base_path}/docker-compose.yml": """version: "3"
services:
  mongo1:
    image: mongo:6.0
    container_name: mongo1
    ports:
      - 27017:27017
    volumes:
      - mongo1_data:/data/db
    command: ["--replSet", "rs0", "--bind_ip_all"]
    networks:
      - mongo_net

  mongo2:
    image: mongo:6.0
    container_name: mongo2
    volumes:
      - mongo2_data:/data/db
    command: ["--replSet", "rs0", "--bind_ip_all"]
    networks:
      - mongo_net

  mongo3:
    image: mongo:6.0
    container_name: mongo3
    volumes:
      - mongo3_data:/data/db
    command: ["--replSet", "rs0", "--bind_ip_all"]
    networks:
      - mongo_net

volumes:
  mongo1_data:
  mongo2_data:
  mongo3_data:

networks:
  mongo_net:
""",
    f"{base_path}/inventory/hosts.ini": """[mongodb]
mongo1 ansible_host=localhost ansible_port=27017
mongo2 ansible_host=localhost ansible_port=27018
mongo3 ansible_host=localhost ansible_port=27019
""",
    f"{base_path}/playbooks/setup-replica.yml": """- name: Setup MongoDB Replica Set
  hosts: mongodb
  become: false
  tasks:
    - include_role:
        name: mongodb
        tasks_from: init_replica
""",
    f"{base_path}/playbooks/create-user.yml": """- name: Create Admin User
  hosts: mongodb
  become: false
  tasks:
    - include_role:
        name: mongodb
        tasks_from: create_user
""",
    f"{base_path}/playbooks/common.yml": "",
    f"{base_path}/roles/mongodb/tasks/main.yml": "",
    f"{base_path}/roles/mongodb/tasks/init_replica.yml": """- name: Initialize the replica set
  shell: |
    mongo --eval '
      rs.initiate({
        _id: "rs0",
        members: [
          { _id: 0, host: "mongo1:27017" },
          { _id: 1, host: "mongo2:27017" },
          { _id: 2, host: "mongo3:27017" }
        ]
      })'
  delegate_to: mongo1
""",
    f"{base_path}/roles/mongodb/tasks/create_user.yml": """- name: Create MongoDB admin user
  shell: |
    mongo admin --eval '
      db.createUser({
        user: "admin",
        pwd: "StrongAdminPassword",
        roles: [ { role: "root", db: "admin" } ]
      })'
  delegate_to: mongo1
""",
    f"{base_path}/roles/mongodb/templates/mongod.conf.j2": ""
}

# Create directories and write files
for path, content in file_contents.items():
    os.makedirs(os.path.dirname(path), exist_ok=True)
    with open(path, "w") as f:
        f.write(content)

f"Project setup completed at: {base_path}"


```


---

## 🧱 Step-by-Step Breakdown

### 🔹 Step 1: Docker Compose for MongoDB Replica Set
Creates 3 MongoDB containers with `--replSet rs0` for replication  ( tells MongoDB:"You're part of a replica set named rs0.").

To spin up three MongoDB containers that will act as replica set members. This creates a foundation for high availability and data redundancy.  
 - Each container (mongo1, mongo2, mongo3) runs MongoDB with --replSet rs0 to prepare for joining a replica set.
 - All nodes (containers or servers) that are supposed to work together in a replica set must use the same name.
 - A Docker network ensures they can talk to each other by name.
 
📦 Think of this step as setting up 3 database servers ready to work together.  


📄 **docker-compose.yml**

```yaml
version: "3"
services:
  mongo1:
    image: mongo:6.0
    container_name: mongo1
    ports:
      - 27017:27017
    volumes:
      - mongo1_data:/data/db
    command: ["--replSet", "rs0", "--bind_ip_all"]
    networks:
      - mongo_net

  mongo2:
    image: mongo:6.0
    container_name: mongo2
    volumes:
      - mongo2_data:/data/db
    command: ["--replSet", "rs0", "--bind_ip_all"]
    networks:
      - mongo_net

  mongo3:
    image: mongo:6.0
    container_name: mongo3
    volumes:
      - mongo3_data:/data/db
    command: ["--replSet", "rs0", "--bind_ip_all"]
    networks:
      - mongo_net

volumes:
  mongo1_data:
  mongo2_data:
  mongo3_data:

networks:
  mongo_net:
```

Run:

```bash
docker-compose up -d
```

---

### 🔹 Step 2: Ansible Inventory File
Defines the MongoDB hosts for Ansible.

To tell Ansible where the MongoDB nodes are and how to connect to them.  
 - You list the containers (hosts) and assign ports or connection settings.  
 - Even though all containers run on the same host (localhost), this lets Ansible treat them like separate servers.  
🧭 It’s like giving Ansible a map of your MongoDB nodes.  



📄 **inventory/hosts.ini**

```ini
[mongodb]
mongo1 ansible_host=localhost ansible_port=27017
mongo2 ansible_host=localhost ansible_port=27018
mongo3 ansible_host=localhost ansible_port=27019
```

---

### 🔹 Step 3: Initialize Replica Set

Initializes the MongoDB replica set from the `mongo1` node.

To run a MongoDB command that joins all 3 containers into one replica set.  
 - rs.initiate() is a MongoDB command that starts the replica set.
 - You run this on mongo1, which will become the primary node initially.
 - The configuration tells MongoDB how many nodes are in the set and their addresses.
🔁 This is where the 3 individual MongoDB containers become a single replicated system.

📄 **roles/mongodb/tasks/init_replica.yml**

```yaml
- name: Initialize the replica set
  shell: |
    mongo --eval '
      rs.initiate({
        _id: "rs0",
        members: [
          { _id: 0, host: "mongo1:27017" },
          { _id: 1, host: "mongo2:27017" },
          { _id: 2, host: "mongo3:27017" }
        ]
      })'
  delegate_to: mongo1
```

---

### 🔹 Step 4: Create Admin User

Adds a secured admin user to the replica set.

To add a secure admin user in the admin database who can manage the replica set.
 - This adds username/password protection.
 - Without this step, anyone could access your database (bad for security!).
🔐 This is your MongoDB security lock — like creating the first admin user after setting up a new computer.

📄 **roles/mongodb/tasks/create_user.yml**

```yaml
- name: Create MongoDB admin user
  shell: |
    mongo admin --eval '
      db.createUser({
        user: "admin",
        pwd: "StrongAdminPassword",
        roles: [ { role: "root", db: "admin" } ]
      })'
  delegate_to: mongo1
```

---

### 🔹 Step 5: Ansible Playbooks to Automate Setup

To put all the MongoDB setup tasks into reusable automation scripts.
 - setup-replica.yml: Automates the replica set configuration.
 - create-user.yml: Automates the creation of the admin user.
 - Each playbook calls role-based tasks for clarity and modularity.

⚙️ This removes the need for you to manually repeat commands. It's a key step in Infrastructure as Code (IaC).



📄 **playbooks/setup-replica.yml**

```yaml
- name: Setup MongoDB Replica Set
  hosts: mongodb
  become: false
  tasks:
    - include_role:
        name: mongodb
        tasks_from: init_replica
```

📄 **playbooks/create-user.yml**

```yaml
- name: Create Admin User
  hosts: mongodb
  become: false
  tasks:
    - include_role:
        name: mongodb
        tasks_from: create_user
```

---

## 🚀 Run the Playbooks

```bash
ansible-playbook -i inventory/hosts.ini playbooks/setup-replica.yml
ansible-playbook -i inventory/hosts.ini playbooks/create-user.yml
```
To execute everything you’ve prepared:
 - Sets up the replica set
 - Adds a secure admin user
 - You now have a production-like MongoDB cluster ready in just a few commands!

🚀 This is where your automation becomes reality — your MongoDB replica set is up and secured!


---

## 🔐 MongoDB Security Hardening Tips

- Use `auth=true` in `mongod.conf`
- Avoid `bind_ip_all` in production (bind to internal IPs instead)
- Rotate passwords periodically
- Enable TLS/SSL for node-to-node and client communication

---

## 🔎 Connect to MongoDB Replica Set

```bash
mongo -u admin -p StrongAdminPassword --authenticationDatabase admin
```

---

## ✅ Outcome

You now have a **secure, high-availability MongoDB replica set** running with:
- Docker containers
- Automated configuration via Ansible
- Admin user access
- Ready for development or staging environments

---

## 📘 Next Steps (Optional Enhancements)

- Integrate TLS for encryption
- Add monitoring with MongoDB Exporter + Prometheus
- Store secrets securely with Ansible Vault

---
```
