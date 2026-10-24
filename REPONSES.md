# Réponses TP2

## Q1
Strapi lit DATABASE_CLIENT, DATABASE_HOST, DATABASE_PORT, DATABASE_NAME, DATABASE_USERNAME, DATABASE_PASSWORD (et optionnellement DATABASE_URL, DATABASE_SSL, DATABASE_SCHEMA, DATABASE_POOL_MIN/MAX). DATABASE_HOST doit valoir "postgres" (nom du service Compose, résolu par le DNS interne de Docker). On n'utilise pas localhost car dans un conteneur localhost désigne le conteneur lui-même, pas le conteneur PostgreSQL.

## Q2
.env.example est un modèle documentant les variables nécessaires, avec des valeurs factices. Le vrai .env contient des secrets (mots de passe, clés JWT) : il ne doit jamais être commité car l'historique Git est permanent et partagé, donc un secret publié est compromis.
## Q3
`docker compose down` arrête et supprime les conteneurs et réseaux, mais conserve les volumes : les données de PostgreSQL survivent (testé avec une table `test`). `docker compose down -v` supprime aussi les volumes : la base est recréée vide au prochain démarrage (la table `test` n'existe plus).

## Q4
Compose interpole lui-même les `${VAR}` du fichier YAML. `$$` est l'échappement qui produit un `$` littéral : la variable est alors résolue par le shell à l'intérieur du conteneur, à l'exécution du healthcheck, et non par Compose au moment de lire le fichier.
## Q5
- `"Host": "postgres"` : pgAdmin tourne dans son propre conteneur ; il joint PostgreSQL via le nom du service Compose, résolu par le DNS interne de Docker vers l'IP du conteneur postgres. `localhost` désignerait le conteneur pgAdmin lui-même.
- `:ro` : montage en lecture seule (read-only). Le conteneur peut lire `servers.json` mais ne peut pas le modifier.
- `${PGADMIN_PORT:-5050}` : substitution avec valeur par défaut. Si `PGADMIN_PORT` n'est pas définie (ou est vide) dans le `.env`, Compose utilise 5050. Le port 5050 de l'hôte est redirigé vers le port 80 du conteneur.