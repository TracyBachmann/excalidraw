# TD DORA — Excalidraw

## Contrat de définitions — Phase 0

Contrat figé le **10/09/2026 à 17:07**.

```yaml
# dora-definitions.yml : contrat d'équipe, versionné avec le code
#
# TD phase 0 : à remplir avant de lancer le moindre outil.
# Chaque ligne est une décision. Une décision non prise ici sera prise par
# l'outil, sans que vous le sachiez.
#
# Remplacez chaque "???". Les commentaires indiquent la section du support
# qui traite la question.

application: excalidraw

deploiement:
  # Que compte-t-on comme déploiement ? Regardez les valeurs d'environment
  # renvoyées par l'API avant de répondre. (support 2.3, 7.3)
  compte_comme_deploiement: "déploiement sur l'environnement Production – excalidraw ayant atteint le statut success"
  exclut: ["environnements Preview et environnements de production correspondant aux exemples d'intégration"]

  # Quel instant fait foi : la création de l'évènement, ou le passage au
  # statut success ? Les deux existent et ne sont pas simultanés. (support 5.2)
  horodatage: "created_at du premier statut success"

changement:
  # Committer date sur la branche par défaut, ou ouverture de la PR ?
  # Le support tranche pour la métrique DORA. L'essentiel est de figer votre
  # convention et de ne plus en changer. (support 2.2)
  point_de_depart: "committer date du commit"

incident:
  # Vous n'exploitez pas la production d'excalidraw. Que décidez-vous
  # d'appeler "incident" ? S'agit-il d'un proxy ou de la métrique DORA ?
  # (support 2.4, 5.1)
  definition: "dégradation du service en production nécessitant une intervention"
  source: "donnée d'incident de production ; absente des données publiques du dépôt"
  debut: "détection de l'incident"
  fin: "rétablissement du service"
  rattachement_deploiement: "relation explicite entre l'incident et le déploiement qui l'a causé"

rework:
  # Comment reconnaîtrez-vous un déploiement non planifié ? La convention
  # doit être décidée avant la collecte, sinon la donnée n'existera pas.
  # (support 2.6)
  marqueur: "référence commençant par hotfix/"

fenetre_de_reference: "90 jours glissants"
agregation: "médiane (P50), avec P90 en complément"

# ---------------------------------------------------------------------------
# Phase 3 : à remplir après avoir vu les chiffres.
# Qu'auriez-vous écrit différemment ? Ne modifiez pas les lignes ci-dessus.
# Une définition changée en cours de route rend la série inexploitable.
# Notez ici, et datez.
#
revision_envisagee: |
  10/09/2026 : après la phase 3, je préciserais qu'un label GitHub comme
  "bug" n'est qu'un proxy et ne doit pas être assimilé directement à un
  incident DORA. Ce proxy dépend de la discipline de saisie et il faut
  conserver un rattachement explicite entre l'incident et le déploiement
  qui l'a causé.
```

---

## Phase 1 — Reconnaissance

### 1. Combien de valeurs différentes d'`environment` trouvez-vous ? Listez-les.

Dans les 100 déploiements récupérés le 10/09/2026, j'ai trouvé **7 valeurs distinctes** :

- `Production – excalidraw`
- `Production – excalidraw-package-example`
- `Production – excalidraw-package-example-with-nextjs`
- `Production – docs`
- `Preview – excalidraw`
- `Preview – excalidraw-package-example`
- `Preview – excalidraw-package-example-with-nextjs`

Le guide indiquait six valeurs lors de son relevé, mais les données récupérées pendant le TD contiennent aussi `Production – docs`.

### 2. Lesquelles correspondent à une mise en production au sens de DORA ? Lesquelles faut-il exclure ?

Pour le système mesuré, je retiens uniquement **`Production – excalidraw`**.

J'exclus les environnements `Preview`, car ce ne sont pas des mises en production. J'exclus aussi `Production – excalidraw-package-example`, `Production – excalidraw-package-example-with-nextjs` et `Production – docs`, car ils correspondent à d'autres artefacts que l'application Excalidraw elle-même.

### 3. De quel facteur la deployment frequency serait-elle surestimée si tous les déploiements étaient comptés ?

Parmi les 100 objets Deployment observés, seulement **3** correspondent à `Production – excalidraw`.

Le facteur de surestimation serait donc :

`100 / 3 ≈ 33,3`

La deployment frequency serait donc surestimée d'environ **33 fois** si je comptais tous les déploiements sans filtrer l'environnement.

### 4. Quelle erreur d'implémentation cela illustre-t-il ?

Cela correspond à l'erreur **« compter les builds comme des déploiements »**, ou plus généralement compter comme déploiement de production un événement qui n'en est pas un.

La correction est de filtrer explicitement l'environnement de production du système mesuré.

### 5. Que contient le tableau des statuts d'un déploiement ? Pourquoi l'existence d'un Deployment ne suffit-elle pas ?

Un Deployment décrit une tentative ou un événement de déploiement, mais son existence ne prouve pas que le changement est réellement arrivé en production.

Il faut regarder ses statuts. Pour le déploiement `6374481865`, le tableau contient un statut `success`. C'est ce statut qui permet de considérer le déploiement comme réussi.

### 6. Quel horodatage faut-il retenir ?

Je retiens le `created_at` du **statut `success`**, et non simplement le `created_at` de l'objet Deployment.

Dans l'exemple observé, les deux valeurs sont identiques (`2026-09-10T14:53:28Z`), mais la convention reste le timestamp du statut `success`, car c'est lui qui atteste que la production a été atteinte.

Mon contrat de phase 0 avait bien tranché ce point avec :

`horodatage: "created_at du premier statut success"`

---

## Phase 2 — Collecte outillée

### Résultats obtenus sur `excalidraw/excalidraw`

Fenêtre : **90 jours**, depuis le **12/06/2026**.

| Métrique | Valeur obtenue |
|---|---:|
| Deployment frequency | `0.167 /jour` — 15 déploiements |
| Délai médian entre deux déploiements | `4.5 jours` |
| Change lead time P50 | `43.8 h` |
| Change lead time P90 | `8.9 jours` |
| Commits analysés / nombre de lots | `79 / 14` |

Les trois autres métriques sont alors à `n/a` : failed deployment recovery time, change fail rate et deployment rework rate.

### 7. Quelle est la taille moyenne d'un lot ? Que dit le support de ce chiffre ?

Le collecteur a analysé **79 commits répartis dans 14 lots**.

`79 / 14 ≈ 5,6 commits par lot`

On est donc sur des lots de quelques commits. Cela va dans le sens de la capability DORA **« travail par petits lots »**, présentée dans le support comme l'un des leviers les plus rentables car elle agit à la fois sur le débit et sur la stabilité.

Je ne peux cependant pas dire qu'un lot de 5,6 commits est « bon » de manière absolue sans contexte supplémentaire.

### 8. Comparez P50 et P90. Quel est le rapport et que signifie cet écart ?

Le P50 est de **43,8 h**. Le P90 est de **8,9 jours**, soit **213,6 h**.

`213,6 / 43,8 ≈ 4,9`

Le P90 est donc environ **4,9 fois plus élevé que la médiane**.

Cela indique qu'une partie minoritaire des changements prend beaucoup plus de temps que le cas habituel. Le support explique qu'un P50 correct avec un P90 beaucoup plus haut peut révéler une catégorie de changements qui se bloque dans le flux.

Cela ne signifie pas que toute la chaîne de livraison est presque cinq fois plus lente : le P90 décrit la longue traîne, pas le cas typique.

### 9. Dans quel ordre de grandeur se situe la deployment frequency par rapport à la distribution 2024 ?

Avec 15 déploiements en 90 jours, la cadence moyenne est de l'ordre d'**un déploiement tous les six jours**, et le délai médian observé est de 4,5 jours.

On est donc dans un ordre de grandeur plutôt hebdomadaire, loin du repère « déploiement à la demande » donné pour le cluster elite.

Je ne classe toutefois pas l'équipe Excalidraw dans un cluster précis à partir de cette seule valeur. Les clusters DORA sont des repères statistiques issus d'une enquête, pas des seuils fixes, et une métrique isolée ne permet pas de juger la performance globale d'une équipe.

### 10. Pourquoi le délai médian est-il plus lisible que la fréquence brute pour une équipe qui déploie peu ?

Quand le nombre de déploiements est faible, une fréquence exprimée en `/jour` est assez abstraite et dépend beaucoup de la fenêtre choisie.

Dire **« un déploiement tous les 4,5 jours en médiane »** est plus parlant. La médiane est aussi moins sensible à une fenêtre arbitraire ou à quelques journées atypiques.

### 11. Quelles sont les trois métriques à `n/a` et qu'ont-elles en commun ?

Les trois métriques à `n/a` sont :

- Failed deployment recovery time
- Change fail rate
- Deployment rework rate

Elles ont en commun de nécessiter une information supplémentaire qui n'est pas déductible des seuls commits et déploiements publics. Il faut disposer d'une source d'incidents, d'un lien explicite avec le déploiement responsable et, pour le rework, d'un marqueur permettant d'identifier un déploiement non planifié.

### 12. Quel est le maillon faible de l'instrumentation ?

Le maillon faible est **le lien entre un incident et le déploiement qui l'a causé**.

Les données publiques donnent les commits et les déploiements, mais elles ne permettent pas de savoir automatiquement qu'un incident précis a été provoqué par un déploiement précis. Sans cette relation explicite, le change fail rate et le recovery time ne peuvent pas être calculés correctement.

### 13. Pourquoi la règle « un nouveau déploiement moins de 24 h après = échec » est-elle mauvaise ?

Elle peut produire un **faux positif** : une équipe peut simplement faire deux déploiements planifiés et parfaitement réussis à quelques heures d'intervalle.

Elle peut aussi produire un **faux négatif** : un déploiement peut provoquer un incident qui n'est corrigé que plus de 24 heures plus tard, ou être résolu autrement que par un nouveau déploiement. La règle ne détecterait alors pas l'échec réel.

Il faut donc une relation explicite avec l'incident, et non une simple proximité temporelle.

---

## Phase 3 — Le proxy et ses limites

### 14. Combien d'issues `bug` le collecteur trouve-t-il ? Combien sont rattachées à un déploiement ?

Le collecteur trouve **23 issues portant le label `bug`** sur sa requête de fenêtre, mais **0 n'est rattachée à un déploiement**.

Le change fail rate reste donc à `n/a`.

### 15. Que montre la vérification dans GitHub ?

Au moment de la vérification :

- **144 issues**, toutes catégories confondues, ont été créées depuis le 12/06/2026 ;
- **765 issues** portent le label `bug` depuis la création du dépôt ;
- **0 issue portant le label `bug`** n'a été créée depuis le 12/06/2026.

### 16. Comment expliquer ces nombres ?

Le label `bug` a bien été utilisé historiquement, mais il n'est plus appliqué aux nouvelles issues pendant la fenêtre étudiée.

Le nombre `23` affiché par le collecteur ne correspond donc pas à 23 nouveaux bugs créés pendant les 90 jours. Le script envoie le paramètre `since` à l'API des issues GitHub. Pour cet endpoint, `since` filtre les issues **mises à jour** depuis la date donnée, pas celles qui ont été créées depuis cette date.

On récupère donc des anciennes issues `bug` qui ont été modifiées récemment. Le proxy ne mesure pas ce que son intitulé pourrait laisser penser.

### 17. Quelle quatrième explication ajouter à un change fail rate de 0 % ?

J'ajouterais : **le marqueur choisi comme proxy n'est plus renseigné, ou la convention de saisie a changé**.

Même avec de vrais incidents, une métrique peut afficher artificiellement 0 % ou `n/a` si le label ou le champ utilisé pour les reconnaître n'est plus appliqué.

### 18. Quelle métrique est la plus sensible à la discipline de saisie ? Est-ce un hasard ?

Le support désigne le **deployment rework rate** comme la métrique la plus sensible à la discipline de saisie.

Ce n'est pas un hasard. Pour mesurer le rework, il faut décider à l'avance comment marquer les déploiements non planifiés (`hotfix`, branche dédiée, champ spécifique, etc.). La phase 3 montre exactement le même problème avec le label `bug` : si le marqueur n'est pas appliqué de manière stable, le chiffre produit devient trompeur.

---

## Phase 4 — Produire la donnée manquante

J'ai instrumenté mon fork `TracyBachmann/excalidraw` avec un workflow GitHub Actions qui crée des événements Deployment sur l'environnement `production`.

J'ai ensuite produit **5 déploiements**, dont un avec la référence :

`hotfix/correctif-urgent`

J'ai aussi créé un incident simulé contenant :

`caused_by: 6375473317`

L'incident a été ouvert à `16:06:03Z` et fermé à `16:09:09Z`, soit environ **3 min 06 s**.

### Résultats obtenus sur mon fork

| Métrique | Valeur obtenue |
|---|---:|
| Deployment frequency | `0.056 /jour` — 5 déploiements |
| Délai médian entre deux déploiements | `0.1 h` |
| Change lead time P50 | `0.0 h` |
| Change lead time P90 | `0.0 h` |
| Commits analysés / nombre de lots | `4 / 4` |
| Failed deployment recovery time | `0.1 h` |
| Change fail rate | `20.0 %` |
| Deployment rework rate | `20.0 %` |
| Issues `incident` | `1` |
| Incidents rattachés | `1` |

Les valeurs de lead time à `0.0 h` sont dues à des commits et déploiements artificiels très rapprochés ; l'affichage du collecteur arrondit à une décimale.

### 19. Que peut-on calculer maintenant qui ne l'était pas avant ?

Grâce à l'instrumentation et au rattachement explicite de l'incident, je peux maintenant calculer les trois métriques auparavant absentes :

- Failed deployment recovery time : **0,1 h**
- Change fail rate : **20 %**
- Deployment rework rate : **20 %**

La différence n'est pas un nouvel algorithme : c'est la présence des données qui manquaient auparavant.

### 20. Combien de temps a demandé la production de cette donnée par rapport à la tentative de la déduire ?

Je n'ai pas chronométré séparément et précisément toute la phase 3 et toute la phase 4, donc je préfère ne pas inventer une durée totale.

En revanche, une fois l'instrumentation en place, l'incident simulé lui-même n'a duré qu'environ **3 minutes** et a suffi à produire une donnée exploitable. Le point important est que créer une convention et enregistrer explicitement le lien `caused_by` donne une information fiable, alors que tenter de déduire ce lien à partir d'indices indirects ne le permet pas.

### 21. Le change fail rate de 20 % est-il représentatif ?

Non. Il correspond à **1 déploiement déclaré défaillant sur seulement 5 déploiements**, dans un historique fabriqué spécialement pour le TD.

Le résultat de 20 % est mathématiquement correct sur cet échantillon, mais il n'est pas représentatif d'une vraie exploitation.

Pour qu'il le devienne, il faudrait accumuler des déploiements réels sur une période suffisamment longue, enregistrer systématiquement les incidents de production et conserver un rattachement fiable entre chaque incident et son déploiement responsable.

---

## Phase 5 — Lecture critique des outils

### Vérification des projets

Vérification effectuée le 10/09/2026 :

| Outil | Observation |
|---|---|
| Apache DevLake | Projet actif ; dernière version observée `v1.0.3-beta17`, publiée le 05/09/2026 ; activité de commits encore présente en septembre 2026 |
| Middleware | Dépôt encore actif ; dernière version observée `0.3.1`, publiée le 30/05/2025 ; commits encore présents en août 2026 |
| Four Keys | Dépôt archivé et en lecture seule ; dernière activité de code en janvier 2024 |

Le relevé DevLake diffère légèrement du tableau du sujet, qui mentionne `v1.0.3-beta16`. Le dépôt ayant évolué, j'ai retenu l'état constaté pendant le TD.

### 22. Ces outils calculeraient-ils automatiquement le change fail rate d'Excalidraw ?

Pas correctement avec les seules données publiques que j'ai observées.

Un outil comme DevLake ou Middleware peut automatiser la collecte et le calcul, mais il a toujours besoin d'une donnée permettant d'identifier les incidents et surtout de les rattacher aux déploiements qui les ont causés.

Si cette relation n'existe pas dans les sources, un outil professionnel ne peut pas la deviner de manière fiable. Ma conclusion de la phase 2 ne change donc pas : le problème principal est la donnée disponible et son sens, pas la taille du programme qui fait le calcul.

### 23. Pourquoi l'outillage arrive-t-il en dernier ?

Parce qu'un dashboard ne peut pas décider à la place de l'équipe ce qu'est un déploiement, un incident ou un rework.

Il faut d'abord clarifier les définitions, discuter des points de friction et savoir ce qu'on veut améliorer. Ensuite seulement, l'instrumentation sert à produire la donnée nécessaire de façon stable et à éviter les vérifications manuelles.

Commencer directement par un gros outil risquerait surtout d'automatiser des conventions floues et de produire des chiffres précis mais faux.

### 24. Quelle habitude faut-il prendre avant d'adopter un outil trouvé en ligne ?

Il faut vérifier l'état réel du projet avant de l'adopter : date des derniers commits, dernières releases, issues, documentation, mainteneurs et éventuel statut `archived`.

Four Keys montre qu'un outil peut rester cité dans beaucoup de tutoriels alors que le projet n'est plus maintenu. Il faut donc vérifier la source actuelle plutôt que se fier uniquement à un article ou un tutoriel ancien.

---

## Restitution

La ligne de mon contrat qui peut expliquer un écart avec un autre binôme est notamment :

```yaml
compte_comme_deploiement: "déploiement sur l'environnement Production – excalidraw ayant atteint le statut success"
```

avec l'exclusion :

```yaml
exclut: ["environnements Preview et environnements de production correspondant aux exemples d'intégration"]
```

Si un autre binôme inclut des environnements `Preview`, `docs` ou les exemples d'intégration, il obtiendra mécaniquement une deployment frequency différente alors qu'il travaille sur le même dépôt.

De la même manière, un changement de fenêtre, d'horodatage ou de point de départ du lead time rendrait les chiffres non directement comparables.
