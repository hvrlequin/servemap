# ServeMap — Plan de salle live (v2)

**ServeMap n'est pas un gestionnaire de tables ni une caisse.** C'est un *plan de salle
vivant* : statuts, alertes et assignations posés **spatialement** sur une map, pour
coordonner un service en salle (bar, club, restaurant, événement) **sans se parler**.

Démo autonome, un seul fichier, **aucune dépendance / aucun backend**.

---

## Lancer

**Double-cliquez sur `index.html`.** C'est tout.

Alternative avec serveur local :
```
python -m http.server 8080     # puis http://localhost:8080
```

Le bouton **↺ Réinitialiser** (en haut à droite) recharge les données d'exemple.

---

## Tester les rôles (sélecteur « Je suis » en haut)

L'app simule le RBAC en changeant d'identité. Chaque rôle ne voit que **ses** actions ;
les prix et l'encaissement sont **bloqués côté logique** (couche `api` / `can()`), pas
seulement masqués.

| Rôle | Incarner | Ce qu'il peut faire |
|------|----------|---------------------|
| **Manager** | Nora | Tout : plan de salle (glisser une table, renommer une zone, ajouter), stock, récap, approuver/refuser les demandes de table, activer la délégation, clôturer le service |
| **Serveur** | Alex / Maya | Ses tables, créer une commande, encaisser, « bouteille servie », **prendre** une table non assignée (demande / auto si délégation) |
| **Commis** | Sam / Théo | Envoyer une commande au serveur de son binôme, demander du renfort. Pas d'accès argent/prix |
| **Barmaid** | Lea | File du bar, signaler « prête », signaler un stock urgent. Pas d'accès argent |

### Parcours démo guidé (onglet **Actions**)
Un encart « ▶ Parcours démo » enchaîne un flux **bout-en-bout sans parler** :
1. **Commis Sam** envoie la commande de *Terrasse 1*.
2. **Barmaid Lea** signale *Table 8* prête.
3. **Serveur Alex** voit l'alerte sur la map, la clôture → **la table passe « servie » automatiquement**.
4. **Manager Nora** suit le récap, les overrides et les frictions.

Chaque étape a un bouton « Incarner … → » qui bascule le rôle et le bon onglet.

---

## Ce que montre la map (écran principal)

- **Contours de salle**, **entrée** marquée, **bar** comme élément physique distinct.
- **4 zones** nommées (Terrasse, Salle haute, Carré VIP, Bar principal) — délimitées, renommables par le manager.
- **Tables placées spatialement** (x,y), avec statut coloré, assignation (ou ⚠ *non assignée*), total dû et minuteur.
- **Notifications posées SUR la map** : `commande_prête` sur la table, `stock_urgent` au bar, `renfort` sur la zone. Le panneau latéral n'est qu'un **journal** complémentaire.
- **Escalade** : les alertes passent `normal → urgent` avec le temps ; au-delà d'un seuil la table devient `priorité_attente` (anneau rouge pulsant).

Statuts de table : `libre · en attente · envoyée · prête · servie · à encaisser · aide · priorité`.

---

## Architecture (pensée pour évoluer en SaaS)

- **Modèle de données** conforme au schéma cible : `Staff`, `Zone`, `ShiftAssignment`,
  `TableOverrideRequest`, `TableOverride`, `StaffPermissionOverride`, `Table`, `Order`,
  `Product`, `Notification`.
- **Couche `api`** : toute mutation passe par une fonction qui vérifie `can(role, action)`
  — c'est le point où brancher un vrai backend/DB plus tard. Le blocage argent existe
  **dès cette version** dans la logique (un commis ne peut pas encaisser, même en
  appelant l'API directement).
- **Résolution d'assignation** (ordre strict) : `override actif` → `zone du shift` →
  `non assignée`. Une table non assignée prise par un serveur crée automatiquement un
  `TableOverride`.
- **Notifications ↔ tables** : résoudre une alerte met à jour le statut de la table
  automatiquement (jamais deux systèmes à synchroniser à la main). Qui peut clôturer
  chaque type est défini explicitement.
- **Persistance** : `localStorage` (clé `servemap_v2`).

### Délégation / overrides à tester
- **Maya** a la délégation activée → sa demande de table s'auto-approuve.
- **Alex** ne l'a pas → sa demande (Bar 2) apparaît chez le **Manager** pour validation.
- Le manager peut basculer la délégation par serveur (onglet Actions).

---

## Périmètre (volontairement limité)

Démo de coordination live — **pas** une caisse complète, **pas** un système de
surveillance du staff. La traçabilité (qui a demandé/approuvé) est un filet de sécurité
pour comprendre après coup, jamais un outil de flicage en direct.
