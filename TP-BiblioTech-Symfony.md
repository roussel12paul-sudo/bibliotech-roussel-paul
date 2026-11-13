# TP — BiblioTech : la médiathèque en Symfony

## Contexte

La médiathèque de quartier **BiblioTech** veut mettre son catalogue en ligne. Les adhérents doivent pouvoir parcourir les livres, les filtrer, consulter la fiche d'un livre et savoir s'il reste un exemplaire disponible.

Une graphiste a déjà réalisé les maquettes HTML / CSS du site. Ton travail : modéliser les données, construire le projet Symfony et intégrer les maquettes en Twig, avec de vraies données en base.

**Durée indicative :** 2 jours.
**Travail :** individuel ou en binôme.

### Compétences mobilisées

| Notion | Où elle intervient |
| --- | --- |
| MCD / MLD (Merise) | Partie 1 |
| SQL | Partie 5 |
| POO en PHP | Parties 3, 6 et 7 |
| Symfony : routes, contrôleurs, Doctrine, migrations | Parties 2, 3, 8 et 9 |
| Fixtures et Faker | Partie 4 |
| Twig | Parties 8, 9 et 10 |

### Fichiers fournis

Dans le dossier `maquettes/` :

| Fichier | Contenu |
| --- | --- |
| `catalogue.html` | La page catalogue : filtres et grille des 12 livres |
| `livre.html` | La fiche d'un livre (exemple : *Germinal*) : informations, disponibilité, derniers emprunts, livres du même genre |
| `style.css` | Tout le CSS du site, pour les deux pages |

Ouvre les deux pages HTML dans ton navigateur avant de commencer : ce sont les pages que ton site devra afficher, avec les données de la base.

---

## Partie 1 — Modélisation (3 points)

### L'énoncé

> La médiathèque gère des **livres**. Chaque livre a un titre, un auteur, un genre (Roman, Aventure, Policier, Science-fiction…), une année de parution, un résumé facultatif, une image de couverture facultative et un nombre d'exemplaires possédés.
>
> Les **adhérents** sont enregistrés avec leur nom, leur ville et leur email ; la ville et l'email sont facultatifs, mais un email ne peut pas être utilisé deux fois.
>
> Un adhérent peut faire plusieurs **emprunts**, ou aucun. Un emprunt est fait par un seul adhérent et porte sur un seul livre. On enregistre la date de l'emprunt, et la date de retour quand le livre est rendu : tant que le livre n'est pas rendu, cette date est vide. Un livre peut avoir été emprunté plusieurs fois, ou jamais.

### Travail demandé

1. Rédige le **dictionnaire de données** : code, signification, type, taille, contraintes.
2. Dessine le **MCD** : entités, identifiants, propriétés, associations et cardinalités.
3. Écris le **MLD** : tables, clés primaires soulignées, clés étrangères précédées de `#`.

**Indice :** la structure ressemble beaucoup à celle de la boutique (CLIENT, COMMANDE, PRODUIT). Repère quelle entité joue le rôle de COMMANDE.

**Point d'attention :** la date de retour est facultative. Comment cela se traduit-il dans le MLD, puis dans Doctrine ?

---

## Partie 2 — Le projet Symfony (1 point)

1. Crée un projet `bibliotech` en Symfony 7.4, avec l'option `--webapp`.
2. Configure la connexion à une base nommée `bibliotech` (dans `.env.local` si ton utilisateur MySQL a un mot de passe).
3. Crée la base avec Doctrine.
4. Lance le serveur et vérifie que la page d'accueil de Symfony s'affiche.

---

## Partie 3 — Les entités et la base (3 points)

Crée les trois entités avec `make:entity`, en respectant ton MLD.

### Livre

| Propriété | Type | Contraintes |
| --- | --- | --- |
| titre | string (100) | obligatoire |
| auteur | string (100) | obligatoire |
| genre | string (50) | obligatoire |
| annee | integer | obligatoire |
| resume | text | facultatif |
| couverture | string (255) | facultatif |
| exemplaires | integer | obligatoire |

### Adherent

| Propriété | Type | Contraintes |
| --- | --- | --- |
| nom | string (50) | obligatoire |
| ville | string (50) | facultatif |
| email | string (100) | facultatif |

### Emprunt

À toi de déduire les propriétés et les deux relations à partir du MCD :

- la date d'emprunt et la date de retour, en `datetime_immutable` ;
- la relation vers l'adhérent et la relation vers le livre, avec la bonne cardinalité et le bon caractère obligatoire ou facultatif ;
- dans `Adherent` et dans `Livre`, une collection `emprunts` qui permet de retrouver leurs emprunts.

Génère ensuite la migration, **lis-la**, puis applique-la.

**À vérifier dans phpMyAdmin :** la table `emprunt` contient deux clés étrangères, et la colonne `date_retour` accepte `NULL`.

---

## Partie 4 — Les fixtures (3 points)

Installe les fixtures et Faker, puis écris `src/DataFixtures/AppFixtures.php`.

### 4.1 Les livres

Les 12 livres du catalogue, à saisir dans un tableau PHP, comme le catalogue de sneakers :

| Titre | Auteur | Genre | Année | Exemplaires |
| --- | --- | --- | --- | --- |
| 1984 | George Orwell | Science-fiction | 1949 | 3 |
| Dune | Frank Herbert | Science-fiction | 1965 | 2 |
| Fondation | Isaac Asimov | Science-fiction | 1951 | 2 |
| Germinal | Émile Zola | Roman | 1885 | 3 |
| L'Étranger | Albert Camus | Roman | 1942 | 2 |
| Le Chien des Baskerville | Arthur Conan Doyle | Policier | 1902 | 2 |
| Le Comte de Monte-Cristo | Alexandre Dumas | Aventure | 1844 | 2 |
| Le Crime de l'Orient-Express | Agatha Christie | Policier | 1934 | 3 |
| Le Tour du monde en 80 jours | Jules Verne | Aventure | 1872 | 4 |
| Les Misérables | Victor Hugo | Roman | 1862 | 2 |
| Madame Bovary | Gustave Flaubert | Roman | 1857 | 1 |
| Vingt mille lieues sous les mers | Jules Verne | Aventure | 1870 | 3 |

- **Résumé :** généré par Faker, avec `$faker->paragraph()`.
- **Couverture :** `'https://placehold.co/400x600?text=' . urlencode($titre)`.

### 4.2 Les adhérents

20 adhérents générés par Faker en français : nom, ville et email unique.

### 4.3 Les emprunts

60 emprunts, chacun avec un adhérent et un livre tirés au hasard :

- la date d'emprunt est comprise dans les 6 derniers mois ;
- environ 7 emprunts sur 10 sont **rendus** : la date de retour est fixée entre 7 et 30 jours après la date d'emprunt ;
- les autres sont **en cours** : la date de retour reste vide.

**Indices :**

- `$faker->boolean(70)` renvoie `true` 7 fois sur 10.
- `$dateEmprunt->modify('+' . $faker->numberBetween(7, 30) . ' days')` renvoie une nouvelle date, sans modifier `$dateEmprunt` : c'est l'intérêt de `DateTimeImmutable`.

**Défi :** un livre ne peut pas avoir plus d'emprunts en cours que d'exemplaires. Après avoir écrit la partie 6, ne crée un emprunt en cours que si le livre a encore un exemplaire disponible ; sinon, crée un emprunt rendu. Pour que le livre « voie » l'emprunt pendant les fixtures, rattache-le avec `$livre->addEmprunt($emprunt)` plutôt qu'avec `$emprunt->setLivre($livre)`.

Charge les fixtures et vérifie le contenu des tables.

---

## Partie 5 — SQL (2 points)

Écris et exécute ces requêtes dans phpMyAdmin. Colle-les dans un fichier `requetes.sql` à la racine du projet.

1. Les livres de Jules Verne, du plus ancien au plus récent.
2. Le nombre de livres par genre.
3. Les emprunts **en cours**, avec le nom de l'adhérent et le titre du livre.
4. Les 5 livres les plus empruntés, avec leur nombre d'emprunts.
5. Les adhérents qui n'ont **jamais** emprunté de livre.

**Indice pour la requête 5 :** `LEFT JOIN` et `IS NULL`, comme dans le tuto SQL.

---

## Partie 6 — La logique métier dans l'entité Livre (2 points)

Ajoute ces méthodes dans `src/Entity/Livre.php` :

| Méthode | Retourne |
| --- | --- |
| `getNombreEmprunts(): int` | Le nombre total d'emprunts du livre |
| `getEmpruntsEnCours(): int` | Le nombre d'emprunts dont la date de retour est vide |
| `getExemplairesDisponibles(): int` | Le nombre d'exemplaires moins les emprunts en cours, jamais négatif |
| `estDisponible(): bool` | `true` s'il reste au moins un exemplaire disponible |

Dans `src/Entity/Emprunt.php`, ajoute aussi :

| Méthode | Retourne |
| --- | --- |
| `estEnCours(): bool` | `true` si le livre n'a pas encore été rendu |

**Indices :**

- `$this->emprunts` est une collection : `count($this->emprunts)` donne son nombre d'éléments.
- `getEmpruntsEnCours()` peut boucler sur les emprunts et utiliser `estEnCours()`.
- `max(0, $valeur)` empêche un résultat négatif.

---

## Partie 7 — Le repository (2 points)

Dans `src/Repository/LivreRepository.php`, écris avec le QueryBuilder :

| Méthode | Rôle |
| --- | --- |
| `findGenres(): array` | La liste des genres, sans doublon, triée |
| `findAuteurs(): array` | La liste des auteurs, sans doublon, triée |
| `findCatalogue(?string $genre, ?string $auteur, ?string $tri): array` | Les livres filtrés par genre et/ou auteur. Tri : par titre par défaut, `annee_desc` ou `annee_asc` |
| `findMemeGenre(Livre $livre, int $nombre = 4): array` | Les livres du même genre, sauf celui affiché |
| `findPlusEmpruntes(int $nombre = 5): array` | Les livres les plus empruntés |

Dans `src/Repository/EmpruntRepository.php` :

| Méthode | Rôle |
| --- | --- |
| `findDerniersPourLivre(Livre $livre, int $nombre = 5): array` | Les derniers emprunts d'un livre, du plus récent au plus ancien |

**Rappels :**

- Les valeurs passent toujours par `setParameter()`.
- `findGenres()` et `findAuteurs()` utilisent `DISTINCT` et `array_column()`.
- `findPlusEmpruntes()` utilise un `join`, un `COUNT` et un `groupBy`, comme `findMeilleuresVentes()`.

---

## Partie 8 — La page catalogue (2 points)

### Ce qui est attendu

- La route `/` nommée `app_catalogue`, dans un contrôleur `CatalogueController`.
- Le CSS fourni intégré au projet, pour toutes les pages.
- L'en-tête et le pied de page dans `base.html.twig`.
- La grille affiche les livres de la base, avec une seule carte dans une boucle.
- Le compteur affiche le nombre réel de livres (« 1 livre », « 12 livres »).
- Le badge est vert avec « x / y disponibles » si le livre est disponible, rouge avec « Indisponible » sinon.
- Les menus déroulants des genres et des auteurs sont générés à partir de la base.
- Les filtres et le tri **fonctionnent**, et les menus gardent la valeur choisie après le filtrage.
- « Réinitialiser » ramène au catalogue sans filtre.

### Indices

- Le CSS va dans `assets/styles/app.css`, déjà chargé par `{{ importmap('app') }}`.
- Les paramètres après le `?` se lisent avec `$request->query->get('genre')`.
- Pour garder une option sélectionnée : `{{ genre == filtres.genre ? 'selected' }}`.

---

## Partie 9 — La fiche d'un livre (2 points)

### Ce qui est attendu

- La route `/livre/{id}` nommée `app_livre`, limitée aux chiffres. Un id inexistant donne une page 404.
- Les cartes du catalogue mènent à la fiche du livre.
- Le fil d'Ariane : « Catalogue » mène au catalogue, le genre mène au catalogue **filtré** sur ce genre.
- Les informations du livre : genre, titre, auteur, résumé, année, nombre d'exemplaires et nombre total d'emprunts.
- Le bloc de disponibilité :
    - vert avec « x exemplaire(s) disponible(s) sur y » si le livre est disponible ;
    - rouge avec « Aucun exemplaire disponible pour le moment » sinon. Dans ce cas, le bouton « Réserver ce livre » est désactivé.
- Le tableau des **derniers emprunts** : nom et ville de l'adhérent, date d'emprunt au format `jj/mm/aaaa`, statut « En cours » ou « Rendu le jj/mm/aaaa ».
- La section « Du même genre », masquée s'il n'y a aucun autre livre du genre.

### Indices

- Le filtre `|date('d/m/Y')` formate une date.
- Depuis un emprunt, on atteint l'adhérent avec `emprunt.adherent.nom`.
- La carte livre est utilisée sur deux pages : place-la dans un partiel `_carte.html.twig` et utilise `include`.
- Les accords au pluriel : `exemplaire{{ nombre > 1 ? 's' }}`.

---

## Partie 10 — La page « Les plus empruntés » (bonus, jusqu'à 2 points)

Aucune maquette n'est fournie : tu réutilises les styles existants.

- La route `/populaires` nommée `app_populaires`, reliée au lien « Les plus empruntés » du menu.
- Un titre, puis les 5 livres les plus empruntés avec `findPlusEmpruntes()`, en réutilisant le partiel `_carte.html.twig`.
- Le lien du menu de la page en cours est mis en avant avec la classe `actif`, grâce à `app.current_route`.

**Pour aller plus loin :** une page `/adherent/{id}` qui affiche un adhérent et l'historique de ses emprunts, accessible en cliquant sur son nom dans le tableau de la fiche livre.

---

## Livrables

1. Le dictionnaire de données, le MCD et le MLD (photo, PDF ou image d'un outil comme Looping).
2. Le lien vers le dépôt GitHub du projet, avec des commits réguliers.
3. Le fichier `requetes.sql` à la racine du projet.

## Barème

| Partie | Points |
| --- | --- |
| 1. Modélisation | 3 |
| 2. Projet Symfony | 1 |
| 3. Entités et base | 3 |
| 4. Fixtures | 3 |
| 5. SQL | 2 |
| 6. Logique métier | 2 |
| 7. Repository | 2 |
| 8. Page catalogue | 2 |
| 9. Fiche d'un livre | 2 |
| **Total** | **20** |
| 10. Bonus | + 2 |

## Liste de vérification avant de rendre

- [ ] `php bin/console doctrine:fixtures:load -n` fonctionne sans erreur.
- [ ] `php bin/console debug:router` affiche `app_catalogue` et `app_livre`.
- [ ] Le catalogue affiche les 12 livres, et chaque filtre fonctionne seul et combiné aux autres.
- [ ] Aucun livre n'affiche un nombre d'exemplaires disponibles négatif.
- [ ] `/livre/9999` affiche une page 404.
- [ ] Aucun texte de la maquette n'est resté écrit en dur à la place d'une donnée.
- [ ] Aucune adresse `href="#"` ne reste dans les templates (sauf le bouton « Réserver »).
