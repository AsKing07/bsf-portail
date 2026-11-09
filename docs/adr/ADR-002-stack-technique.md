# ADR-002 — Justifier la stack : framework, versions, base de données, build, tests

- **Date** : 2026-11-09
- **Statut** : **Validée**
- **Auteur** : Charbel Sonon — rédaction assistée par Claude (copilote IA). L'IA propose, elle ne valide pas.
- **Validée par** : Charbel Sonon, Product Owner junior du Portail, le 2026-11-09 (les six décisions, une par une). Relecture de l'équipe : à travers la pull request, qui exige une approbation.
- **Ticket** : BSF-44
- **Décisions liées** : D-007 (stack imposée), D-012 (rien d'autre n'est imposé), D-014 et D-016 (deux services, dépôts séparés), D-017 (Spring Boot et PostgreSQL pressentis), D-019 (OIDC)
- **Documents concernés** : registre des décisions (une entrée à créer à la validation), `ARCHITECTURE.md`, `ENGINEERING.md` §11, `.env.example`, tickets BSF-51, BSF-52, BSF-53, BSF-11

**Niveau de preuve.** Chaque fait important porte une marque :
✅ vérifié le 09/11/2026 sur la source citée en fin de document · 🔶 calculé à partir d'une politique publiée · ⚠️ connaissance générale, **non revérifiée** pour cet ADR.

---

## Contexte

**Ce qui est imposé, et ce qui ne l'est pas.** Seuls **Angular, Java, GitHub et Jira** nous sont imposés (D-007). Tout le reste est notre choix, et il devra être **justifié devant le jury** (D-012). Spring Boot et PostgreSQL ont été *pressentis* (D-017) ; « pressenti » n'est pas « justifié ». C'est l'objet de cet ADR.

**Contraintes du projet.**
- Équipe de 5, environ 5 h par personne et par semaine, 2 à 3 personnes à l'aise techniquement (D-011, R-21).
- Horizon : déploiement pilote le 14/04/2027, soutenance finale le 29/04/2027. Toute version choisie doit être **supportée au moins jusqu'à fin avril 2027**, avec de la marge.
- Architecture **proportionnée** : une application web (SPA), deux services back (Portail et `bsf-auth`, déjà décidés en D-014 et D-016), une base PostgreSQL par service (BSF-53). Rien de plus.
- Données fictives uniquement, dépôts publics (D-038).

**Donnée d'entrée du spike BSF-9.** Spring Boot 4.1.2 fonctionne pour notre besoin d'authentification (Spring Authorization Server, intégré à Spring Security 7) : 8 tests verts. Le prototype tournait sous Java 24 avec `release 17` dans le POM : **ces deux valeurs étaient provisoires** et sont tranchées ici.

**Critères de décision.**

| # | Critère |
|---|---|
| C1 | Support éditeur couvrant avril 2027, avec marge |
| C2 | Maintenabilité : peu de mécanismes à connaître, conventions standard |
| C3 | Facilité de test |
| C4 | Risque de livraison pour cette équipe : temps disponible et courbe d'apprentissage (et non « c'est ce qu'on connaît ») |
| C5 | Déploiement : conteneurisable, compatible avec les offres d'hébergement envisagées (ADR-006) |
| C6 | Intégration et sécurité : OIDC/JWT, OpenAPI, contrats avec les autres groupes |

---

## Décision 1 — Framework back-end

| Option | Avantages | Inconvénients | Coût | Risques |
|---|---|---|---|---|
| **A — Spring Boot 4.1.x** | ✅ Validé par le spike BSF-9 (Spring Authorization Server). Même framework pour le Portail et `bsf-auth` : une seule compétence à acquérir. Version 4.1 supportée jusqu'en juin 2027 (donnée du spike, consignée dans BSF-44) | Version majeure récente : moins de tutoriels et de réponses en ligne que pour la 3.x ⚠️. Noms de starters de sécurité changés depuis Boot 4 | Nul : déjà en place dans `bsf-auth` | Changements entre versions mineures de la 4.x : on fige la 4.1.x et on n'applique que des correctifs |
| **B — Quarkus** | Démarrage rapide, mode développement agréable ⚠️ | Pas de serveur d'autorisation embarqué équivalent à Spring Authorization Server : il faudrait un fournisseur externe lourd ou écrire le protocole ⚠️. Deux frameworks si `bsf-auth` reste en Spring | Reprise du spike et de la formation | Dépasse la capacité de l'équipe (C4) ; défait un résultat prouvé |
| **C — Spring Boot 3.5 ou 4.0** | Plus de documentation disponible | Support open source de la 3.5 terminé en juin 2026 ; celui de la 4.0 s'arrête en décembre 2026 (données du spike) : **ne couvrent pas la soutenance** | Nul | Échoue sur C1. *Listée pour mémoire : ce n'est pas une alternative réelle.* |

*Micronaut relève de la même famille que Quarkus et n'a pas été étudié séparément.*

**Décision retenue : A — Spring Boot 4.1.x.**
**Justification.** C6 d'abord : le seul composant que le spike a prouvé est Spring Authorization Server, qui ne vit que dans cet écosystème. C1 ensuite : la branche 4.1 est la seule qui couvre la soutenance. C2 : un seul framework pour deux services.
**Compromis accepté.** Version jeune, avec moins de ressources en ligne ; et un support qui s'arrête deux mois après la soutenance. Cela ne nous coûte rien : le projet s'arrête avant, et nous figeons la 4.1.x.

---

## Décision 2 — Base de données

| Option | Avantages | Inconvénients | Coût | Risques |
|---|---|---|---|---|
| **A — PostgreSQL 18** | Cinq ans de support par version majeure, sans niveau « LTS » (✅ politique du projet) : fin de vie en novembre 2030 🔶. ✅ Pris en charge par Azure Database for PostgreSQL Flexible Server (version mineure 18.6 au 09/11/2026), la piste d'hébergement de D-018. Index uniques **partiels** ⚠️ : permettent de faire respecter en base « au plus une affectation active par alternant » (D-024, règle 1) | Version majeure sortie en septembre 2025 ⚠️ : moins de retours d'expérience | Nul (installation neuve, aucune donnée à migrer) | Compatibilité du pilote JDBC, d'Hibernate et de l'outil de migration avec la 18 : **non vérifiée ici** ⚠️ |
| **B — PostgreSQL 17** | Plus mûre. Fin de vie novembre 2029 🔶, largement au-delà d'avril 2027. Mêmes index partiels | Une version majeure de retard sans bénéfice identifié pour ce projet | Nul | Aucun risque particulier |
| **C — MariaDB / MySQL** | Très répandues ⚠️ | Pas d'index unique partiel ⚠️ : la règle D-024 n'est plus garantie par la base mais seulement par le code. Aucune offre étudiante identifiée dans D-018 | Reprise de BSF-52 et BSF-53 | Perte d'une garantie d'intégrité que les critères de BSF-53 demandent côté base |

*SQL Server n'est pas retenu comme option : il n'apparaît dans aucune piste d'hébergement gratuite de D-018.*

**Décision retenue : A — PostgreSQL 18**, image épinglée sur une version mineure précise (pas de `latest`, BSF-52).
**Justification.** C1 : support le plus long, 🔶 jusqu'en novembre 2030, pour une marge de trois ans et demi après la soutenance. C5 : géré par Azure. C6/intégrité : les index partiels et les contraintes `CHECK` permettent de refuser en base un statut invalide ou une seconde affectation active, ce que BSF-53 exige.
**Compromis accepté.** Version majeure récente. Les tests d'intégration tourneront sur **la même image** que l'environnement local (voir décision 6) : une incompatibilité apparaîtrait dès la CI.
**Condition de sortie.** Vérifier la compatibilité pilote/Hibernate/outil de migration avec la 18 dans BSF-52 ou BSF-53. Si elle échoue : repli sur la 17, sans coût tant qu'aucune base n'est déployée.

---

## Décision 3 — Version de Java

| Option | Avantages | Inconvénients | Coût | Risques |
|---|---|---|---|---|
| **A — Java 25 (LTS)** | ✅ Compatible Spring Boot 4.1.2 (minimum 17, compatible jusqu'à 27). LTS publiée en septembre 2025 ⚠️. Support Oracle annoncé jusqu'en septembre 2030 (projection d'un document daté de mai 2024 ✅/⚠️) | Chaque poste doit installer un JDK 25 (Charbel exécute le spike sous 24) ; certains outils (plugins d'IDE ou de Maven) peuvent retarder sur la version la plus récente ⚠️ | Installation d'un JDK par poste | Outillage en retard : détecté par la CI, qui fige la version |
| **B — Java 21 (LTS)** | Outillage très répandu ⚠️. Support Oracle jusqu'en septembre 2028 ✅ | Une LTS de retard ; mêmes fonctions utiles à ce projet (fils virtuels disponibles dès la 21 ⚠️) | Nul | Aucun risque particulier |
| **C — Java 17 (LTS)** | C'est le minimum de Boot 4.1 | ✅ Support premium Oracle terminé en septembre 2026 : c'est la plus courte marge | Nul | Échoue presque sur C1. *Listée pour mémoire.* |

**Décision retenue : A — Java 25 (LTS).**
**Justification.** C1 : la LTS la plus récente, donc la plus longue. Aucun bénéfice identifié à rester en 21 pour un projet neuf, et un changement de version en cours de route (21 → 25) serait pire qu'un choix net aujourd'hui.
**Compromis accepté.** Un peu plus de friction d'installation et un risque de retard d'outillage, contre deux ans de support supplémentaires par rapport à la 21 (septembre 2030 contre septembre 2028, selon la même source). La version est **figée dans le POM et dans la CI** : le même JDK partout.
**Repli.** Descendre en 21 revient à changer la valeur de `release` et l'image de la CI.

---

## Décision 4 — Version d'Angular

| Option | Avantages | Inconvénients | Coût | Risques |
|---|---|---|---|---|
| **A — Angular 22** | ✅ Publiée le 03/06/2026. Support actif jusqu'en juin 2027, support à long terme (LTS) jusqu'en juin 2028 : **la seule branche en support actif à la soutenance**. Exigences ✅ : Node ^22.22.3 \|\| ^24.15.0 \|\| ^26.0.0 ; TypeScript 6.0 ; RxJS ^6.5.3 \|\| ^7.4.0 | Publiée il y a cinq mois : des bibliothèques peuvent ne pas encore la déclarer compatible. **Compatibilité d'`angular-oauth2-oidc` non vérifiée** (R-24, BSF-10) ⚠️ | Aucun code front n'existe : changer de version aujourd'hui ne coûte rien | La bibliothèque de connexion OIDC ne suit pas : repli sur la 21 |
| **B — Angular 21** | ✅ Support à long terme jusqu'en juin 2027 : couvre la soutenance **de justesse**, en correctifs de sécurité seulement. Node ^20.19.0 \|\| ^22.12.0 \|\| ^24 ; TypeScript 5.9 | Aucune correction de bogue ordinaire après le 03/06/2026. Deux mois de marge seulement | Nul | Une faille ou un bogue non critique ne serait pas corrigé |
| **C — Angular 20** | — | ✅ Fin du support le 28/11/2026 | — | Échoue sur C1. *Listée pour mémoire.* |

**Décision retenue : A — Angular 22**, avec Node 24 (LTS) et TypeScript 6.0 ⚠️ *(Node 24 est dans la plage déclarée ; son statut LTS n'a pas été revérifié).*
**Justification.** C1 : support actif sur toute la durée du projet, plus un an d'LTS au-delà. Le choix est sans regret **aujourd'hui** parce qu'aucune ligne d'Angular n'est écrite.
**Compromis accepté.** Une version de cinq mois, donc un risque de bibliothèques en retard.
**Condition de sortie.** La **première action de BSF-10** est de vérifier qu'`angular-oauth2-oidc` fonctionne avec Angular 22. Si ce n'est pas le cas, on retombe sur la 21 **avant** le premier écran codé. La version d'Angular n'a pas à être alignée avec celle des autres groupes (D-016 : front indépendant).

---

## Décision 5 — Outil de build

| Option | Avantages | Inconvénients | Coût | Risques |
|---|---|---|---|---|
| **A — Maven + Maven Wrapper** (back) · **npm** (front) | ✅ Déjà utilisé par le spike (`mvn clean test`) ; ✅ pris en charge par Boot 4.1 (Maven 3.6.3 ou plus). Cycle de vie fixe (`compile`, `test`, `verify`) : correspond directement aux quatre étapes de BSF-51. Fichier déclaratif lisible par tous. Le wrapper fige la version de Maven pour tous les postes et la CI | XML verbeux ⚠️. Constructions incrémentales moins rapides ⚠️ | Nul | Peu de souplesse pour une logique de build spécifique : sans objet ici |
| **B — Gradle** (Kotlin DSL) + wrapper | ✅ Pris en charge par Boot 4.1 (8.14 ou plus, et 9.x). Concis ; constructions incrémentales plus rapides ⚠️ | Un langage de description de build à apprendre en plus (C4). Plus de façons de mal écrire un build ⚠️. Son point fort, les builds multi-modules, **ne sert pas** : nos deux services sont dans deux dépôts séparés | Réécriture du spike et formation | Courbe d'apprentissage pour 5 personnes à 5 h par semaine |

*Front : npm (par défaut avec l'Angular CLI, `npm ci` en CI avec un fichier de verrouillage) contre pnpm ou Yarn : aucun bénéfice identifié ici qui justifierait un outil de plus.*

**Décision retenue : A — Maven avec Maven Wrapper côté back, npm côté front.**
**Justification.** C2 et C4 : un seul chemin à apprendre, déjà pratiqué dans le spike. C3/CI : les commandes de BSF-51 s'en déduisent sans configuration spécifique.
**Compromis accepté.** On renonce à la vitesse des builds incrémentaux de Gradle : à la taille de ce projet, le gain n'est pas mesurable pour nous.
**Hors décision.** Les outils de format et de lint (côté Java et côté Angular) se choisissent dans BSF-51.

---

## Décision 6 — Bibliothèques de test

**Back-end.**

| Option | Avantages | Inconvénients | Coût | Risques |
|---|---|---|---|---|
| **A — JUnit Jupiter + AssertJ + Mockito (via `spring-boot-starter-test`), Spring Security Test, Testcontainers (PostgreSQL)** | ✅ Le starter apporte JUnit Jupiter, AssertJ et Mockito ; ✅ JUnit 6.0.3 géré par Boot 4.1.1. Spring Security Test permet les tests de permissions (403) exigés par `ENGINEERING.md` §6 ⚠️. Les tests d'intégration tournent sur **le vrai PostgreSQL**, donc prouvent les contraintes de base (douze statuts, une affectation active) | Docker requis en local et en CI (déjà requis par BSF-52). Tests d'intégration plus lents ⚠️ | Quelques secondes par démarrage de conteneur ⚠️ | Docker indisponible sur un poste : seuls les tests unitaires tournent |
| **B — Mêmes outils, avec H2 en mémoire** | Aucun Docker, très rapide | H2 ne reproduit pas fidèlement PostgreSQL ⚠️ : index partiels, énumérations et `CHECK` ne seraient pas éprouvés. Des tests verts qui ne prouvent pas ce qu'on prétend prouver | Nul | **Faux sentiment de sécurité** sur les règles d'intégrité |

**Front-end.**

| Option | Avantages | Inconvénients | Coût | Risques |
|---|---|---|---|---|
| **A — Vitest** | ✅ Lanceur par défaut des nouveaux projets de l'Angular CLI (angular.dev). Aucune migration : projet neuf | Moins de ressources que Karma/Jasmine sur des cas anciens ⚠️ | Nul | La page officielle qualifie d'*expérimentale* la migration d'un projet existant : sans objet pour un projet neuf |
| **B — Karma + Jasmine** | Historique, très documenté ⚠️ | Plus le lanceur par défaut ; je n'ai pas pu confirmer son statut de dépréciation sur la page consultée | Nul | Aller à contre-courant du défaut de l'outil |

**Décision retenue.** Back : **A** (JUnit Jupiter, AssertJ, Mockito, Spring Security Test, Testcontainers PostgreSQL, **sur la même image que l'environnement local**). Règles métier (transitions de D-048) testées en **tests unitaires sans contexte Spring**. Front : **Vitest**.
**Justification.** C3 : le risque principal du projet est le contrôle des droits et des statuts ; ces tests doivent prouver quelque chose. Les tests d'intégration sur le vrai moteur sont le seul moyen d'éprouver les contraintes côté base exigées par D-025 et BSF-53.
**Compromis accepté.** Docker obligatoire et des tests d'intégration plus lents, contre des tests qui prouvent réellement les règles d'intégrité.
**Reporté.** Test de bout en bout du parcours P0 : **Playwright** est envisagé ⚠️ mais **non étudié ici** ; à confirmer au jalon Prototype (`ENGINEERING.md` §5), sans engagement avant. Tests de contrat : hors cet ADR.

---

## Synthèse

| Sujet | Retenu | Support publié jusqu'à | Repli |
|---|---|---|---|
| Framework back-end | Spring Boot 4.1.x | juin 2027 (donnée du spike) | aucun réaliste après S4 |
| Base de données | PostgreSQL 18 | novembre 2030 🔶 | PostgreSQL 17 |
| Java | 25 (LTS) | septembre 2030 (Oracle, projection) | Java 21 |
| Angular | 22, Node 24, TypeScript 6.0 | LTS juin 2028 ✅ | Angular 21 |
| Build | Maven + Wrapper ; npm | — | — |
| Tests | JUnit Jupiter, AssertJ, Mockito, Testcontainers ; Vitest | — | — |

## Ce que cet ADR ne décide pas

| Sujet | Où il sera traité |
|---|---|
| Outil de migration de base de données | BSF-53 (son critère d'acceptation exige un choix justifié, dans l'ADR ou le README) |
| Accès aux données (JPA/Hibernate, JDBC, autre) | Premier ticket de persistance |
| Format et lint (Java, Angular) | BSF-51 |
| Image Docker et version mineure de PostgreSQL | BSF-52 |
| Authentification | ADR-003 (BSF-54) |
| Hébergement | ADR-006 (BSF-55), échéance 31/01/2027 |
| Test de bout en bout | Jalon Prototype |

## Ce que nous ne choisissons pas

- **Pas de microservices** au-delà des deux services déjà décidés (Portail et `bsf-auth`, D-014). Aucun besoin identifié ne le justifie, et nous serions incapables de défendre les coûts de coordination devant le jury.
- **Pas de Kubernetes** : une application, une API, une base relationnelle.

---

## Conséquences

- **Positives** : une stack unique et supportée sur toute la durée du projet ; réutilisation directe du spike ; tests qui prouvent les règles d'intégrité ; marge de support confortable.
- **Négatives / dette assumée** : trois composants récents (Boot 4.x, Angular 22, PostgreSQL 18) donc moins de ressources en ligne ; JDK 25 à installer sur chaque poste ; Docker obligatoire pour les tests d'intégration.
- **Impact sur les autres groupes** : **aucun**. Le Référentiel et le Passeport restent libres de leur stack (D-016). Seuls comptent les contrats OpenAPI et la validation des jetons par JWKS, indépendants du langage.
- **Ce qu'il faut mettre à jour** :
  - BSF-51 : JDK 25, Maven Wrapper, Node 24, `npm ci`.
  - BSF-52 : image `postgres:18.x` épinglée ; même version utilisée par Testcontainers.
  - BSF-53 : PostgreSQL 18 ; vérifier la compatibilité de l'outil de migration.
  - `bsf-auth` : le POM passe de `release 17` à `25` (à porter par BSF-11) ; la version de Spring Boot reste 4.1.x.
  - BSF-10 : première action = vérifier `angular-oauth2-oidc` avec Angular 22.
  - Fichiers de suivi : une entrée dans le registre des décisions, `ARCHITECTURE.md`, `ENGINEERING.md` §11.
  - L'index de ce dossier (fait dans cette PR).

## Comment revenir en arrière

*Estimations, non mesurées.*

| Décision | Retour arrière | Coût estimé |
|---|---|---|
| Java 25 → 21 | Changer `release` et l'image de la CI | Moins d'une heure |
| Angular 22 → 21 | Changer les versions du `package.json` | Faible tant qu'aucun code front n'existe ; croît avec le code |
| PostgreSQL 18 → 17 | Changer l'image et rejouer les migrations | Nul tant qu'aucune base n'est déployée ; ensuite, export puis restauration |
| Maven → Gradle | Réécrire les fichiers de build des deux dépôts | Environ une demi-journée à une journée par dépôt |
| Spring Boot → autre framework | Réécriture du back-end et de `bsf-auth` | Impraticable dans le calendrier après S4 : **décision à considérer comme irréversible** |

## Sources consultées le 09/11/2026

- Angular — versions et support : <https://angular.dev/reference/releases> ; compatibilité Node/TypeScript : <https://angular.dev/reference/versions> ; tests : <https://angular.dev/guide/testing/migrating-to-vitest>
- Spring Boot 4.1 — exigences système : <https://docs.spring.io/spring-boot/4.1/system-requirements.html>
- JUnit géré par Spring Boot : <https://howtodoinjava.com/junit/junit-version-with-spring-boot/>
- PostgreSQL — politique de support : <https://www.herodevs.com/blog-posts/postgresql-eol-dates-every-versions-release-end-of-life-timeline> ; versions prises en charge par Azure : <https://learn.microsoft.com/azure/postgresql/flexible-server/concepts-supported-versions>
- Java LTS : <https://www.petefreitag.com/blog/java-lts-versions/> *(page de mai 2024 : dates de Java 25 projetées)*
- Spike BSF-9 et commentaire du 02/11/2026 sur BSF-44 (supports de Spring Boot).
