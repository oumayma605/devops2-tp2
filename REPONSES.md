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
## Q6
Copier `package.json` et `package-lock.json` puis lancer `npm ci` avant `COPY . .` permet d'exploiter le cache des couches Docker : une couche invalidée entraîne le rejeu de toutes les suivantes, et le code change bien plus souvent que les dépendances. Ainsi `npm ci` (la couche la plus lente, environ 94 s) n'est rejoué que si les dépendances changent.

Observation : après modification de `backend/config/server.js`, les étapes WORKDIR, COPY package*.json et `npm ci` sont restées en `CACHED` (0,0 s) ; `COPY . .` a été rejouée, de même que `npm run build` et la copie finale. Après retour au fichier d'origine, toutes les couches ont été réutilisées du cache (build en 1,5 s contre 171 s au premier build).

## Q7
`USER node` fait tourner le processus avec un utilisateur non privilégié au lieu de root : si l'application est compromise, l'attaquant a des droits limités (principe du moindre privilège). `--chown=node:node` donne la propriété des fichiers copiés à l'utilisateur `node` : sans cela ils appartiendraient à root et Strapi ne pourrait pas y écrire (cache, fichiers temporaires, uploads). Le faire directement dans le `COPY` évite une couche `RUN chown -R` supplémentaire qui alourdirait l'image.