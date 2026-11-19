# minecraft-server-config

This setup is the ultimate solution: it automatically downloads the latest backup, fetches the server and mods, creates backups regularly, and uploads them to Docker Hub.

# How to run
1. Set env variables:
- `CF_API_KEY` - The API key from CurseForge Studio.
- `MC_DATA_PATH` - The relative path to the Minecraft world directory.
- `MC_BACKUPS_PATH` - The relative path to the backups folder.
- `MC_SERVER_PORT` - The server port.
- `HUB_REPO_NAME` - The Docker Hub repository for fetching and uploading backups Format 'NICKNAME/REPO_NAME'
- `HUB_LOGIN` - Your Docker Hub username.
- `HUB_PASSWORD` - Your Docker Hub access token.
- `DOCKER_IMAGE_TO_PULL` - The name of the Docker image to pull. Format: 'NICKNAME/REPO_NAME:TAG'

If you don’t want to upload backups, you can skip setting the login and password. It’s recommended to save these variables in a local `.env` file and use `source .env` to load them before starting the server.

2. Download and start the Docker setup:
Get the docker-compose.yml file and launch the setup.
```bash
curl -o docker-compose.yml https://raw.githubusercontent.com/L3odr0id/minecraft-server-config/refs/heads/main/docker-compose.yml
docker compose up -d
```

## To copy the world backup from the image

1. Pull the image
```bash
docker image pull <tag>
```

2. Create container from image with dummy command
```bash
docker container create <image_id> echo "Hello"
```

docker container create h1v3m1nd/remote-mc-backup:20260713145909 echo "Hello"

3. Copy from container
```bash
docker cp <container_id>:/data.tgz old_backup_world.tgz
```

docker cp 42507f94d1f326f0e42500c7f4cda91306f2ab738c267fd50df333d01dfbf78e:/data.tgz old_backup_world.tgz

4. Untar
```bash
tar -xvzf old_backup_world.tgz -C old_backup_folder
```
## Update scoreboard for BlazeandCave's Advancements

```
/function bacap_rewards:update_score
```

## File Permissions

When running the container, directory permissions can sometimes block the server from writing data. The [itzg's documentation](https://docker-minecraft-server.readthedocs.io/en/latest/variables/) notes that the server runs under a user with UID=1000 and GUID=1000.

Fix permissions:

```bash
source .env
sudo chown -R 1000:1000 "$MC_DATA_PATH" "$MC_BACKUPS_PATH"
sudo chmod -R u+rwX "$MC_DATA_PATH" "$MC_BACKUPS_PATH"
```

## Add your used to docker group

```bash
sudo usermod -aG docker $USER
```

## Download world from server to you local device

1. Archive
```bash
sudo tar -cvzf <path> world
```

2. Download
```bash
scp <name>@<host>:<remote-path> <local-path>
```

Local MultiMC path on macos to saves:
`/Applications/MultiMC.app/Data/instances/26.2/.minecraft/saves/`

## Fix broken generation

1. Download ForceBlend for Fabric from https://github.com/Mauro0805/ForceBlend/releases
2. Install it as a mod
3. Edit `<game>/config/forceblend.properties`
```
run-mode=once
automatic-processing=true
automatic-backup=false
detailed-logging=true
maximum-chunks-per-run=unlimited
processing-threads=all
```
4. Open singleplayer worlds

The issues was found that some chunks are stored in an old way that ForceBlend does not work with. They do not include xPos and zPos at root level.
`Edit -> Optimize world` with tick erase cache.

