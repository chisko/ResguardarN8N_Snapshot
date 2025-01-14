# How to Create an Instance of n8n from a Snapshot, Update its Version, and Backup Using Snapshots

This guide provides step-by-step instructions to create a reliable instance of n8n, update its version, and secure its data using snapshots. Follow each step carefully to ensure a smooth process.

## Documentation
For further reference, check the official documentation: https://docs.n8n.io/hosting/installation/docker/#updating

## 1. Add Your User to the Docker Group

To run Docker commands without `sudo`:

```bash
whoami
sudo usermod -aG docker user
```

Log out and log back in for the changes to take effect:

```bash
exit
```

## 2. Verify the Current Version of n8n

Check the n8n version running inside the container:

```bash
docker exec -it n8n n8n --version
```

## 3. Inspect the Container to Find the Mount and Correct Directory

Run the following command to inspect the n8n container and locate the correct mount point:

```bash
docker inspect n8n
```

## 4. Update Domain Name (If Changed)

If the domain name for your n8n instance has changed, update your Nginx configuration:

```bash
sudo nano /etc/nginx/sites-available/n8n
```

Save changes with `Ctrl + O`, press `Enter`, and then exit with `Ctrl + X`.

## 5. Verify the Container ID

List all containers to get the correct Container ID:

```bash
docker ps -a
```

## 6. Backup Configuration Files from the Container

Access the container and list the configuration files:

```bash
docker exec -it --user node n8n sh
ls -la /home/node/.n8n
```

Backup the configuration files to your local machine:

```bash
docker cp <container_id>:/home/node/.n8n ./n8n_backup
```

Exit the container:

```bash
exit
```

Verify the backup file on your local machine:

```bash
ls -la ./n8n_backup
```

## 7. Update n8n to a Specific Version

Pull the desired version of the n8n Docker image:

```bash
docker pull docker.n8n.io/n8nio/n8n:1.73.1
```

## 8. Stop and Remove the Existing Container

Stop and remove the old container:

```bash
docker stop <container_id>
docker rm <container_id>
```

## 9. Create a Docker Volume for Persistent Data

Create a volume to store n8n data persistently:

```bash
docker volume create n8n_data
```

Copy the backup data into the new volume:

```bash
docker run --rm -v n8n_data:/data -v $(pwd)/n8n_backup:/backup busybox sh -c "cp -r /backup/* /data/"
```

Inspect the volume to confirm the data:

```bash
docker volume inspect n8n_data
```

## 10. Adjust Permissions for the Volume

Ensure the `node` user inside the container has proper permissions:

```bash
sudo chown -R 1000:1000 /var/lib/docker/volumes/n8n_data/_data
sudo chmod -R 700 /var/lib/docker/volumes/n8n_data/_data
```

## 11. Verify the Backup Data in the Container

Start the container and check that the data is properly restored:

```bash
docker exec -it --user node n8n sh -c "ls -la /home/node/.n8n"
```

## 12. Create a New n8n Container with the Restored Data

Run a new n8n container with the restored data and updated version:

```bash
docker run -d --restart unless-stopped \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="your-domain.com" \
-e WEBHOOK_TUNNEL_URL="https://your-domain.com/" \
-e WEBHOOK_URL="https://your-domain.com/" \
-v n8n_data:/home/node/.n8n \
n8nio/n8n:1.73.1
```

## Important Additional Information (Must Read)

### How Persistent Volumes Work
The `n8n_data` volume is stored outside of the container, ensuring that all your workflows and configurations are preserved even if the container is stopped or removed. Here's how it works:

1. **Data remains safe when stopping or restarting the container**:
   - Use `docker stop n8n` to stop the application.
   - Use `docker start n8n` to restart it, and your data will still be intact.

2. **Volume persists beyond container deletion**:
   - If you delete the container with `docker rm n8n`, the data in the `n8n_data` volume will remain safe.
   - You can reuse the volume by mapping it to a new container:

   ```bash
   docker run -d --restart unless-stopped \
   --name n8n \
   -p 5678:5678 \
   -v n8n_data:/home/node/.n8n \
   n8nio/n8n:latest
   ```

By leveraging Docker volumes, you ensure that your n8n instance remains resilient and your data secure through updates or container changes.

