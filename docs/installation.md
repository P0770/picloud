# Installation pas à pas

Toutes les valeurs entre chevrons (`<...>`) et le domaine `mondomaine.ch` sont à remplacer par les vôtres.

## 1. Système

1. Flasher **Raspberry Pi OS Lite 64 bits** avec Raspberry Pi Imager, en activant SSH et en créant un utilisateur.
2. Se connecter depuis le réseau local :
   ```bash
   ssh <utilisateur>@<ip-du-pi>
   ```
3. Mettre le système à jour :
   ```bash
   sudo apt update && sudo apt full-upgrade -y
   ```

## 2. SSD

1. Formater le SSD en ext4 (⚠️ efface son contenu) :
   ```bash
   sudo mkfs.ext4 /dev/nvme0n1
   sudo mkdir -p /mnt/ssd
   ```
2. Le monter automatiquement à chaque démarrage, en l'ajoutant à `/etc/fstab` :
   ```bash
   echo '/dev/nvme0n1 /mnt/ssd ext4 defaults 0 2' | sudo tee -a /etc/fstab
   sudo mount -a
   df -h /mnt/ssd
   ```
   Pour plus de robustesse, on peut utiliser l'UUID du disque (`sudo blkid`) au lieu de `/dev/nvme0n1`.
3. Créer l'arborescence :
   ```bash
   sudo mkdir -p /mnt/ssd/nextcloud /mnt/ssd/jellyfin/config /mnt/ssd/jellyfin/media
   ```

## 3. Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER      # puis se reconnecter
docker compose version
```

## 4. Services

```bash
git clone https://github.com/P0770/picloud.git
cd picloud
cp .env.example .env
nano .env                          # remplir les valeurs
docker compose up -d
docker compose ps
```

Accès local : Nextcloud sur le port 8080, Jellyfin sur 8096, Portainer sur 9000.

## 5. Tunnel Cloudflare

1. Le domaine doit être géré par Cloudflare (DNS avec le proxy activé, nuage orange).
2. Dans **Zero Trust → Networks → Tunnels**, créer un tunnel de type **Cloudflared**, environnement **Docker**, et copier son jeton dans `TUNNEL_TOKEN` du fichier `.env`.
3. Ajouter les noms d'hôte publics. Comme `cloudflared` est dans le même réseau Docker que les services, on peut viser directement le nom du conteneur :

   | Nom d'hôte public | Service |
   |---|---|
   | `cloud.mondomaine.ch` | `http://nextcloud:80` |
   | `media.mondomaine.ch` | `http://jellyfin:8096` |

   *Variante utilisée à l'origine, avec des conteneurs lancés séparément : viser l'adresse du pont Docker de l'hôte, `http://172.17.0.1:<port>`.*

4. **Interfaces d'administration (Portainer) :** ne pas les publier telles quelles. Soit les garder accessibles uniquement en local, soit les protéger par une application **Cloudflare Access** (connexion par code envoyé par email).

## 6. Nextcloud

1. Ouvrir `https://cloud.mondomaine.ch` et vérifier la connexion avec le compte administrateur.
2. Si le domaine est refusé, l'ajouter aux domaines de confiance :
   ```bash
   docker exec -u www-data nextcloud php occ config:system:set trusted_domains 1 --value=cloud.mondomaine.ch
   ```
3. **Dossier partagé avec Jellyfin :**
   - activer l'application **External storage support** ;
   - ajouter un stockage externe « Local » pointant vers `/mnt/media` ;
   - donner le dossier au serveur web : `docker exec nextcloud chown -R www-data:www-data /mnt/media` ;
   - activer l'**API de partage** dans Administration → Partage.
4. Créer un compte par membre de la famille.

## 7. Jellyfin

Ouvrir `http://<ip-du-pi>:8096`, créer le compte administrateur, puis ajouter les bibliothèques à partir des sous-dossiers de `/media`.
