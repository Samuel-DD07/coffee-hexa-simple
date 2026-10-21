# Réponses

**1. Pourquoi le domaine n'importe ni Spring ni JPA ?**

Parce qu'il s'en fiche de la base et du web, c'est juste des règles métier. Du coup on le teste en JUnit pur, sans Spring ni H2, et ça tourne en quelques ms.

**2. Pourquoi pas de @Service ni @Transactional sur le service ?**

Pour garder l'application sans Spring. Le bean est déclaré dans `UseCaseConfig`. Les transactions passent par le port `TransactionRunner`, implémenté par `SpringTransactionRunner` avec un `TransactionTemplate`.

**3. Pourquoi Coffee et CoffeeJpaEntity séparés ?**

JPA veut un constructeur vide, des setters et des listes modifiables. `Coffee` fait exprès l'inverse. Les mélanger casserait le domaine. Le mapper coûte rien, MapStruct le génère.

**4. Pourquoi la règle du prix est dans le domaine et en SQL mais pas dans le DTO ?**

Le domaine fait foi, pas envie de dupliquer la règle. Le DTO vérifie juste la forme. Le `CHECK` SQL c'est au cas où quelqu'un écrit en base sans passer par l'appli.

**5. Pourquoi 422 pour un prix de 500 et 400 pour un nom vide ?**

Nom vide : bloqué par `@NotBlank` sur le DTO, requête mal formée, 400. Prix 500 : le JSON est valide mais le domaine refuse, donc 422.

**6. Sans @EntityGraph ?**

N+1 : un `select` pour les cafés puis un par café pour les avis, donc 4 au lieu de 1 avec les données de démo. On le voit en comptant les `select` dans les logs (SQL en debug en dev).

**7. H2 vers PostgreSQL ?**

`pom.xml` (driver postgres + `flyway-database-postgresql`, virer h2 et h2console) et les `.properties` (URL, user, mdp). Domaine et application : rien.

**8. Ajouter un champ origin ?**

Avant la recette : on modifie `Coffee`, V1 et `CoffeeJpaEntity` ensemble (sinon `validate` refuse de démarrer), puis le mapper, les DTO et le seed.

Après la recette : on touche plus à V1. On fait un `V2__add_origin_to_coffee.sql` avec une colonne nullable, puis le même code.
