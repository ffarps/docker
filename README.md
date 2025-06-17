# Guide to Docker management
## why
```bash
docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          506       16        455.9GB   440.4GB (96%)
Containers      24        0         492.5MB   492.5MB (100%)
Local Volumes   418       11        26.78GB   25.11GB (93%)
Build Cache     1057      0         1.38GB    1.38GB
```
## Thats WHY

### A Practical Guide to Docker Cleanup

Ever run df -h and wonder where hundreds of gigabytes of disk space went? If you use Docker, you've likely found the culprit. Docker's images, volumes, and build cache can accumulate over time, consuming a massive amount of storage.

This guide provides a simple, step-by-step workflow to diagnose the problem and safely reclaim your disk space.

#### Step 1: Diagnose the Problem

Before deleting anything, find out what's using the space. Docker has a built-in command for this.
```
docker system df
```

This command gives you a clear breakdown of space used by:

- **Images**: The templates for your containers. Old and dangling layers from builds are often the biggest problem.
- **Containers**: Stopped containers that you're no longer using.
- **Local Volumes**: Persistent data for your applications (e.g., databases, logs).
- **Build Cache**: To speed up future image builds.

The RECLAIMABLE column is key—it tells you how much space you can get back without affecting any running containers.

#### Step 2: The Safe & Quick Cleanup

This is the first command you should always try. It's a safe, non-destructive way to perform basic cleanup.
```
docker system prune
```

This will remove:

- All stopped containers
- All unused networks
- All dangling images (layers that don't belong to any tagged image)
- All dangling build cache

It's the low-hanging fruit and a great first step.

#### Step 3: The Aggressive Cleanup (Where the Real Savings Are)

If docker system df showed that most of your space is taken by "Images" or "Local Volumes," you'll need a more powerful command.

### Reclaiming Image Space

If you have lots of old, unused images, this is the command for you. It removes all images that aren't associated with at least one container (running or stopped).

This will free up a LOT of space if you have many unused images.
```
docker system prune -a
```

You'll have to re-download or rebuild images when you need them again, but that's a small price to pay for reclaiming potentially hundreds of gigabytes.

### Reclaiming Volume Space

**⚠️ DANGER ZONE: This command permanently deletes data. ⚠️**

Volumes store the persistent data for your applications. This command will remove all volumes that are not currently attached to a container. If you have a stopped database container, its data volume will be considered "unused" and will be deleted forever.

Proceed with caution.

Only run this if you are sure you don't need the data in your unused volumes.
```
docker volume prune
```
_Bonus: Targeted Deletion_

Sometimes you don't want to wipe everything. Here's how to be more specific.

Find the Biggest Images

List all your images, sorted by size, to find the main offenders.
```
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" | sort -rh | head -n 20
```
Delete Specific Images or Volumes
Once you have the ID or name, you can delete items one by one.

Delete one or more specific images
```
docker rmi <IMAGE_ID_1> <IMAGE_ID_2>
```
Delete one or more specific volumes
```
docker volume rm <VOLUME_NAME_1> <VOLUME_NAME_2>
```

Recommended Workflow
- Diagnose: ```docker system df```
- Quick Prune: ```docker system prune```
- Check Again: ```docker system df```
- Deep Prune (if needed): docker system prune -a and (carefully!) docker volume prune.

Happy Dockering
