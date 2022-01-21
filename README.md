## Steps to run Gitea in rootless docker:
1. Create 3 folders: data, config, custom/public/css:
```bash
mkdir -p data config custom/public/css
```
2. Place `theme-gitea-gruvbox.css` in `./custom/public/css`, or any other theme;
3. Change owner of these folders (idk why 166535, this worked for me):
```bash
sudo chown -R 166535:166535 config/ data/ custom/
```
4. Give all permissions to this folder, or config files would not be created, no matter if you changed owner in previous step: 
``` bash
sudo chmod -R ugo+rwx config data custom
```
5. Copy systemd service file to default folder and run it:
```bash
cp gitea-rootless-compose.service ~/.config/systemd/user/
systemctl --user enable --now gitea-rootless-compose.service
```
6. Go to your Gitea domain and set it up;
7. Run `sh` inside gitea container, edit `/etc/gitea/app.ini`, add this at the end of the file:
```ini
[ui]
THEMES=auto,gitea,arc-green,gitea-gruvbox
```
8. Restart service, config would apply:
```bash
systemctl --user restart gitea-rootless-compose.service
```

## Proxy and database configuration
In separate docker compose I run `mariadb`, `nginx-proxy` and `nginxproxy/acme-companion`.
Here is configuration:
```yaml
  db:
    image: mariadb:10.5
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    restart: always
    volumes:
      - db:/var/lib/mysql
      - ./init:/docker-entrypoint-initdb.d
    env_file:
      - db.env
    networks:
      - maria
      - default

  proxy:
    build: ./proxy
    restart: always
    ports:
      - 80:80
      - 443:443
    labels:
      com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy: "true"
    volumes:
      - certs:/etc/nginx/certs:ro
      - vhost.d:/etc/nginx/vhost.d
      - html:/usr/share/nginx/html
      - $XDG_RUNTIME_DIR/docker.sock:/tmp/docker.sock:ro
    networks:
      - proxy-tier

  letsencrypt-companion:
    image: nginxproxy/acme-companion
    restart: always
    volumes:
      - certs:/etc/nginx/certs
      - acme:/etc/acme.sh
      - vhost.d:/etc/nginx/vhost.d
      - html:/usr/share/nginx/html
      - $XDG_RUNTIME_DIR/docker.sock:/var/run/docker.sock:ro
    networks:
      - proxy-tier
    depends_on:
      - proxy

networks:
  proxy-tier:
    external:
      name: proxy-tier
  maria:
    external:
      name: maria
```

`mariadb` env file:
```env
MYSQL_PASSWORD=password
MYSQL_DATABASE=database
MYSQL_USER=user
MARIADB_ROOT_PASSWORD=password
```

With this configuration ssh works on `2222` port without any additional steps.
Caution!!! If you have recent `OpenSSH`, you wouldn't be able to pull repositories using `ssh` if you have `rsa` key. For more information 

### Links
- [Grovbox theme](https://github.com/perpetualCreations/gruvbox-gitea)
- [OpenSSH 8.8 not connecting using rsa key](https://github.com/go-gitea/gitea/issues/17798)
