```
sudo docker pull openkm/openkm-ce:latest
```
```
sudo nvim docker-compose.yml
```
isi
```
services:
  openkm-db:
    image: mariadb:10.11
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: openkm
      MYSQL_USER: openkm
      MYSQL_PASSWORD: openkmpassword
    volumes:
      - openkm_db_data:/var/lib/mysql
    networks:
      - openkm_net
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager

  openkm:
    image: openkm/openkm-ce:latest
    environment:
      DB_DRIVER: com.mysql.cj.jdbc.Driver
      DB_URL: jdbc:mysql://openkm-db:3306/openkm?autoReconnect=true&useUnicode=true&characterEncoding=UTF-8
      DB_USER: openkm
      DB_PASSWORD: openkmpassword

      TZ: Asia/Jakarta

    ports:
      - target: 8080
        published: 8080
        protocol: tcp
        mode: ingress

    volumes:
      - openkm_data:/opt/tomcat/repository
      - openkm_logs:/opt/tomcat/logs

    networks:
      - openkm_net

    depends_on:
      - openkm-db

    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure

networks:
  openkm_net:
    driver: overlay

volumes:
  openkm_db_data:
  openkm_data:
  openkm_logs:
```
```
sudo docker stack deploy -c docker-compose.yml [nama_container]
```

## nginx
```
server {
    listen 80;
    server_name openkm.your-domain.com; 
    # Avoid checking files size #
    client_max_body_size 0;

    rewrite ^/$ /openkm permanent;
    
    location /openkm/frontend/webSocket {
        proxy_pass http://localhost:8080/openkm/frontend/webSocket;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
    
    location /openkm {
         proxy_set_header Host $host;
         proxy_set_header X-Forwarded-Host $host;
         proxy_set_header X-Forwarded-Server $host;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_pass http://localhost:8080/openkm;
    }
}
```
