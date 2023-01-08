## Steps to run Gitea in rootless docker:
1. Copy systemd service file to default folder and run it:
```bash
cp gitea-rootless-compose.service ~/.config/systemd/user/
systemctl --user enable --now gitea-rootless-compose.service
```
2. Open port `2222` for ssh:
```bash
sudo ufw allow 2222/tcp
```

Configure ssh on client by editing `~/.config/ssh`:
```
Host git.domain.com
	HostName git.domain.com
	Port 2222
	IdentityFile ~/.ssh/your_private_key
	IdentitiesOnly yes
```

Also, add key to ssh agent:
```bash
eval `ssh-agent -s`
ssh-add ~/.ssh/your_private_key
```

With this configuration ssh works on `2222` port without any additional steps.


Links:
- [Install with docker rootless](https://docs.gitea.io/en-us/install-with-docker-rootless/)
