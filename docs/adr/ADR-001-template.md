# ADR-001 — Modèle d'Architecture Decision Record

> Copier ce fichier en `ADR-0XX-titre-court.md` dans ce même dossier, pour chaque décision structurante.
>
> Une décision est **structurante** si l'un de ces trois critères est rempli :
> - la défaire coûterait plus d'une journée de travail ;
> - elle engage les autres groupes du programme (Référentiel, Passeport) ;
> - elle devra être justifiée devant le jury.
>
> Si aucun des trois n'est rempli, une ligne dans la description du ticket Jira suffit. On ne rédige pas un ADR pour le plaisir d'en avoir.

## Index des ADR

| N° | Titre | Statut | Ticket |
|---|---|---|---|
| 001 | Modèle d'ADR (ce fichier) | — | — |
| 002 | Justifier la stack : framework, versions, base, build | À rédiger | BSF-44 |
| 003 | Stratégie d'authentification | À rédiger | BSF-54 |
| 004 | Adaptateur vers le Référentiel de compétences | À rédiger | — |
| 006 | Hébergement — **échéance 31/01/2027** | À rédiger | BSF-55 |

> Tenir cet index à jour à chaque ADR ajouté. Un dossier d'ADR sans index devient illisible au bout de six décisions.

---

## ADR-0XX — &lt;Titre court et explicite&gt;

- **Date** : AAAA-MM-JJ
- **Statut** : Proposée | **Validée** | Rejetée | Remplacée par ADR-0YY
- **Auteur** : &lt;nom&gt;
- **Validée par** : &lt;nom et fonction — obligatoire pour le statut « Validée »&gt;
- **Ticket** : BSF-XX
- **Documents concernés** : &lt;documents de suivi à mettre à jour&gt;

### Contexte

Quel problème se pose, dans quelles contraintes (temps, compétences disponibles, dépendances, règles de l'école), et pourquoi une décision est nécessaire **maintenant** plutôt que plus tard.

### Options étudiées

| Option | Avantages | Inconvénients | Coût | Risques |
|---|---|---|---|---|
| A — | | | | |
| B — | | | | |
| C — | | | | |

> Au moins deux options réelles. Une option écrite uniquement pour être écartée ne compte pas : le jury le verra.

### Décision retenue

&lt;Option choisie, formulée sans ambiguïté.&gt;

### Justification

Pourquoi cette option plutôt que les autres, au regard du contexte ci-dessus. **Mentionner explicitement le compromis accepté** — une décision sans compromis est généralement une décision mal instruite.

« C'est ce qu'on connaît » n'est pas une justification recevable. En revanche, « les compétences disponibles dans l'équipe réduisent le risque de ne pas livrer avant le jalon du JJ/MM » en est une : elle nomme le critère et l'échéance.

### Conséquences

- **Positives** :
- **Négatives / dette assumée** :
- **Impact sur les autres groupes** :
- **Ce qu'il faut mettre à jour** :

### Comment revenir en arrière

Ce qu'il faudrait faire pour défaire cette décision, et à quel coût estimé. Si la réponse est « impossible », le dire — c'est une information importante pour les décisions suivantes.

---

## Règles de tenue de ce dossier

1. **Un ADR validé ne se modifie pas.** Si la décision change, on en rédige un nouveau et on passe l'ancien en « Remplacée par ADR-0YY ». L'historique des décisions fait partie du livrable.
2. **Le statut « Validée » exige un nom** dans le champ « Validée par ». Une proposition de l'équipe n'est pas une décision validée.
3. **Les numéros ne sont jamais réutilisés**, même si un ADR est rejeté.
4. La décision est reportée dans le registre des décisions du projet, avec son identifiant D-0XX, pour que les deux sources concordent.
