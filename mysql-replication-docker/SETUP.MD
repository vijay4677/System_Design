# MySQL 8.0 Primary–Replica Replication Using Docker

This setup demonstrates MySQL 8.0 replication using Docker Compose.

It includes:

- Primary (Write Node)
- Replica (Read Node)
- Custom Docker Network
- Step-by-step configuration

---

## Architecture

Primary
- server-id=1
- Binary logging enabled

Replica
- server-id=2
- read_only enabled
- Replicates binlog from primary

---

## Step 1: Start Containers

```bash
docker-compose up -d
```

Wait 20–30 seconds for MySQL to initialize.

---

## Step 2: Configure Replication User

Login to Primary:

```bash
docker exec -it mysql-primary mysql -uroot -proot
```

Run:

```sql
CREATE USER 'replica'@'%' 
IDENTIFIED WITH mysql_native_password 
BY 'replica123';

GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%';
FLUSH PRIVILEGES;

SHOW MASTER STATUS;
```

Note the output:

File and Position.

Example:

File: mysql-bin.000003  
Position: 157  

---

##  Step 3: Configure Replica

Login to Replica:

```bash
docker exec -it mysql-replica mysql -uroot -proot
```

Run:

```sql
STOP REPLICA;

CHANGE REPLICATION SOURCE TO
  SOURCE_HOST='mysql-primary',
  SOURCE_PORT=3306,
  SOURCE_USER='replica',
  SOURCE_PASSWORD='replica123',
  SOURCE_LOG_FILE='mysql-bin.000003',
  SOURCE_LOG_POS=157;

START REPLICA;
```

Check status:

```sql
SHOW REPLICA STATUS\G
```

Healthy replication shows:

Replica_IO_Running: Yes  
Replica_SQL_Running: Yes  

---

##  Step 4: Test Replication

On Primary:

```sql
CREATE DATABASE test_replication;

USE test_replication;

CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(50)
);

INSERT INTO users (name) VALUES ('Vijay');
```

On Replica:

```sql
SHOW DATABASES;

USE test_replication;

SELECT * FROM users;
```

You should see the inserted row.

---

##  Replica Protection

Replica runs with:

read_only=1  
super_read_only=1  

This prevents accidental writes.

---

##  How Replication Works

Primary writes changes to Binary Log (binlog).

Replica has:
- IO Thread → Reads binlog
- SQL Thread → Applies changes

---

##  Common Errors

### Authentication requires secure connection
Fix: Use mysql_native_password

### Can't connect to source
Fix: Use container name + internal port 3306

### SQL thread stopped (Error 1396)
Fix: RESET REPLICA ALL and reconfigure

---

##  Reset Replica (If Needed)

```sql
STOP REPLICA;
RESET REPLICA ALL;
```

Then reconfigure using latest master status.

---

##  Stop Containers

```bash
docker-compose down
```

---

##  Learning Outcomes

- MySQL binary logging
- Docker networking
- Container DNS resolution
- Authentication plugin differences
- IO vs SQL replication threads
- Read-only replica protection

---

Happy Learning 