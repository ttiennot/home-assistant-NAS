# Home Assistant sur Synology DS423

Installation de Home Assistant en conteneur sur un Synology DS423, avec les
lumières Govee H61B5 et H60A1 en contrôle local. La partie volets roulants est
à compléter une fois la motorisation identifiée.

## 1. Installer Home Assistant

Le DS423 a un processeur ARM et 2 Go de RAM : pas de machine virtuelle, donc
Home Assistant tourne en conteneur (sans add-ons).

1. Dans DSM, ouvrir le **Centre de paquets** et installer **Container Manager**.
2. Dans **File Station**, créer le dossier `docker/homeassistant` sur le volume 1.
   Si le dossier partagé `docker` est sur un autre volume, adapter le chemin
   `/volume1/...` dans `docker-compose.yml`.
3. Dans **Container Manager > Projet > Créer** :
   - Nom du projet : `homeassistant`
   - Chemin : `docker/homeassistant`
   - Source : « Créer un fichier docker-compose.yml », puis coller le contenu
     du fichier `docker-compose.yml` de ce dépôt.
4. Valider et lancer le projet. Le premier téléchargement prend quelques minutes.
5. Ouvrir `http://IP-DU-NAS:8123` et créer le compte administrateur.

Si le pare-feu de DSM est activé (**Panneau de configuration > Sécurité >
Pare-feu**), autoriser depuis le réseau local :

| Port | Protocole | Usage |
|---|---|---|
| 8123 | TCP | Interface de Home Assistant |
| 4002 | UDP | Réponses des lumières Govee |

## 2. Lumières Govee (H61B5 et H60A1)

Les deux modèles sont pris en charge par l'intégration officielle
**Govee lights local** : contrôle en Wi-Fi local, sans cloud ni clé d'API.

1. Dans l'application **Govee Home**, pour chaque lumière : ouvrir l'appareil,
   puis **Paramètres (roue dentée) > Contrôle LAN** et l'activer. Si l'option
   n'apparaît pas, mettre à jour le firmware de la lumière.
2. Dans Home Assistant : **Paramètres > Appareils et services > Ajouter une
   intégration > Govee lights local**. Les lumières sont détectées
   automatiquement.

Conditions pour que la détection fonctionne :

- le NAS et les lumières sont sur le même réseau (même sous-réseau, pas de
  réseau Wi-Fi « invité » isolé) ;
- le conteneur est bien en `network_mode: host`.

Fonctions disponibles : marche/arrêt, luminosité, couleur, température de
blanc et scènes. Le réglage segment par segment reste dans l'application Govee.

## 3. Volets roulants AvosDim

À compléter selon la motorisation :

| Motorisation | Solution |
|---|---|
| Somfy RTS | ESPSomfy-RTS (ESP32 + module CC1101), en Wi-Fi |
| Somfy io | Somfy Connectivity Kit ou TaHoma Switch, intégration Overkiz |
| Filaire | Module Shelly 2PM derrière l'interrupteur |
| Radio AvosDim | À étudier selon la référence du moteur |

## 4. Mises à jour et sauvegardes

- **Mise à jour** : dans Container Manager, onglet **Image**, mettre à jour
  l'image `home-assistant`, puis reconstruire le projet.
- **Sauvegarde** : toute la configuration est dans `docker/homeassistant`.
  L'inclure dans une tâche Hyper Backup suffit.
