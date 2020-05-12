# Upgrade Gotify

Update Gotify to the Latest Version 🌀

## Upgrade Process

- Stop the Systemd Service

```bash
sudo systemctl stop gotifypush
```

```bash
sudo systemctl daemon-reload
```

- Open the Gotify Installed Location

```bash
cd /var/www/gotifypush
```

- Rename the Older version

```bash
sudo mv gotify-linux-amd64 gotify-linux-amd64-old
```

- Download the Gotify Latest Version from the Github Release Page <https://github.com/gotify/server/releases>
- Download via `wget`

```bash
wget https://github.com/gotify/server/releases/download/<Latest-Release>/gotify-linux-amd64.zip
```

```bash
unzip /gotify-linux-amd64.zip
```

- Give Execute Permission

```bash
chmod +x gotify-linux-amd64
```

- Verify your Installation - you should see that your system is running the latest version of Gotify

```bash
./start.sh
```

- Remove the Downloaded assets & Older version of Gotify

```bash
rm -rf gotify-linux-amd64.zip LICENSE licenses gotify-linux-amd64-old
```

- if your installation Test is passed Press CTRL + C to Stop & Start the systemd Service

```bash
sudo systemctl start gotifypush
```

```bash
sudo systemctl daemon-reload
```

```bash
sudo systemctl restart gotifypush
```

```bash
sudo service apache2 restart
```
