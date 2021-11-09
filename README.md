# Install Koel

## 1. install database

first, we need to build mysql database (docker)

```bash
docker pull mysql
docker run --name mysql -e MYSQL_ROOT_PASSWORD=<password> -d mysql
```

second, we need create koel user and database.

```bash
# get in mysql docker 
docker exec -it mysql mysql -u root -p
# create user in MySQL, '#' means all;
create user koel@'#' identified by 'password';
create database koel_database;
# link user and database
grant all privileges on koel_database.* to koel@'#';
```

## 2. install koel

first, we need docker pull images:

```bash
docker pull hyzual/koel

# create container and link database and port. 80 means koel default container port, 4040 means host port;
docker run -d --name koel \
        -p 4040:80 \
        --link mysql:mysql \
        hyzual/koel

# get in koel container and init
php artisan koel:init --no-assets
```

**Default admin account:**
```python
email: admin@koel.dev
password: KoelIsCool
```

Make sure to change this unsecure password with the user interface (click on your profile picture) or by running the following command:

```bash
docker exec -it <container_name_for_koel> 
php artisan koel:admin:change-password dzx1@mail.ustc.edu.cn
```

### 3. Nginx config

create file in `/etc/nginx/sites-enabled/`

**We need to lift upload size limit**
```markdown
server {
    listen 80;
    server_name koel.dzxustc.top;
    location / {
        proxy_pass http://localhost:4040;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version      1.1;
        # websocket support
        proxy_set_header        Upgrade $http_upgrade;
        proxy_set_header        Connection "Upgrade";
        proxy_read_timeout      86400;
    }
    client_max_body_size 512M;
}
```





