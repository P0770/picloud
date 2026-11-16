# Dépannage

Problèmes rencontrés pendant la mise en place du serveur, avec leur cause et leur solution.

## Connexion SSH refusée : `Host key verification failed`

**Cause :** après une réinstallation du système, le Pi présente une nouvelle clé d'identification, différente de celle mémorisée par l'ordinateur.

**Solution :** supprimer l'ancienne clé connue, puis se reconnecter.
```bash
ssh-keygen -R <ip-du-pi>
```

## Les données de Nextcloud sont sur la carte SD

**Cause :** sans volume explicite, Docker crée un volume par défaut sur le disque système (la carte SD).

**Solution :** recréer le conteneur avec ses données sur le SSD (`/mnt/ssd/nextcloud:/var/www/html`, déjà prévu dans `docker-compose.yml`).

## Le SSD n'est plus monté après un redémarrage

**Cause :** le disque n'était pas déclaré dans `/etc/fstab`.

**Solution :** voir l'étape 2 de [l'installation](installation.md). Vérification : `df -h /mnt/ssd`.

## Portainer inaccessible par le tunnel

**Cause :** le tunnel visait le port HTTPS de Portainer (9443), avec un certificat que Cloudflare refusait.

**Solution :** exposer le port HTTP 9000 et faire viser ce port par le tunnel. La connexion reste chiffrée entre l'utilisateur et Cloudflare, et le tunnel lui-même est chiffré.

## Nextcloud refuse le nom de domaine

**Cause :** le domaine n'est pas dans la liste `trusted_domains`.

**Solution :** variable `NEXTCLOUD_TRUSTED_DOMAINS` à la création, ou commande `occ` (voir l'étape 6 de l'installation).

## Le stockage externe de Nextcloud est illisible

**Cause :** le dossier appartient à un autre utilisateur que celui du serveur web.

**Solution :**
```bash
docker exec nextcloud chown -R www-data:www-data /mnt/media
```

## Partage de fichiers « non autorisé »

**Cause :** l'API de partage est désactivée.

**Solution :** l'activer dans Administration → Partage.

## Le domaine affiche une erreur 502

**Cause :** ce n'est pas une panne de Cloudflare. Le proxy répond toujours, mais il ne trouve plus le serveur à l'autre bout du tunnel : le Pi est éteint, ou le conteneur `cloudflared` est arrêté.

**Vérification :**
```bash
docker compose ps
docker logs cloudflared --tail 50
```
