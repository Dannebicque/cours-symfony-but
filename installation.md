# Installation

Sous l'image docker MMI

Connexion sur le container Symfony de docker

```bash
docker exec -ti symfony /bin/bash
```

Installation de symfony dans /var/www

```bash
cd /var/www
symony new nomduprojet
ou
composer create-project symfony/skeleton nomduprojet
```

Installation des dépendances (toutes les dépendances de webapp)

```bash
cd /var/www/nomduprojet
composer require webapp
```

Dupliquer le .env (qui ne doit jamais être modifié avec vos données)

```bash
cp .env .env.local
```

Configurer apache, en créant une configuration qui pointe vers le dossier nomduprojet

Relancer apache
