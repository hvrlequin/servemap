# ServeMap — Plan de salle live

**ServeMap n'est pas une caisse ni un gestionnaire de tables. C'est un *plan de salle vivant*** :
statuts, alertes et assignations posés **spatialement** sur une carte, pour coordonner un
service en salle (bar, club, restaurant, événement) **sans se parler**.

> Démo web autonome — **un seul fichier**, **aucune dépendance**, **aucun backend**.
> Profil de démonstration actuel : **Mirano** (club, Bruxelles).

---

## 🔗 Démo en ligne

**👉 https://hvrlequin.github.io/servemap/**

(Ou télécharger `index.html` et **double-cliquer** — ça marche aussi hors-ligne.)

---

## ▶️ Lancer en local

| Méthode | Comment |
|---|---|
| Le plus simple | **Double-cliquer `index.html`** (fonctionne en `file://`, hors-ligne). |
| Petit serveur | Dans le dossier : `python -m http.server 8080` puis ouvrir `http://localhost:8080`. |

Bouton **↺** (en haut à droite) = recharger les données d'exemple.

---

## ✨ Ce que fait la démo

- **Plan de salle interactif** (SVG) : Parterre (piste + DJ), Balcon (carrés VIP), Loges, Bar central, Fumoir, entrée/vestiaire. Thème **or & noir**.
- **4 rôles** avec une identité incarnable (sélecteur « Je suis ») : **Serveur, Runner, Bar, Responsable**. Chaque rôle ne voit que ses actions ; les prix et l'encaissement sont **bloqués côté logique** pour Runner/Bar (pas seulement masqués).
- **Deux vues complémentaires** dans le rail : **⚡ Ma mission** (perso, la file « à faire ») et **👁 Salle en direct** (vue **commune** de toute la salle par zone — ce que toute l'équipe voit).
- **Notifications posées sur la carte** + escalade dans le temps (normal → urgent → priorité) ; **le statut d'une table se met à jour automatiquement** quand l'alerte liée est résolue.
- **Routage d'alerte** : quand le bar dit « bouteille prête », c'est affiché au **serveur** (qui clôture) **et** au **runner** du binôme (en lecture, pour aider à monter).
- **Prise de commande rush-proof** : sélecteur par catégories (Bouteilles / Verres / Softs / Autre), tap = ajouté, + **suggestions de mixers** (choisis une vodka → on te propose Red Bull, Tonic… ; gin → Tonic ; whisky → Coca ; champagne → coupes).
- **Assignations & overrides** : résolution stricte *override → zone → non assignée* ; un serveur peut « prendre » une table non assignée (auto si délégation, sinon validation du responsable).
- **Stock** (paliers disponible / bientôt épuisé / indisponible, déduit à la vente), **récap responsable**, **vue mobile serveur** (barre du bas : À faire / Plan / Ma salle), **bouton flottant ＋ Commande**.
- **Persistance** locale (`localStorage`).

### Comment tester en 30 secondes
1. **👁 Salle en direct** → « toute la soirée en un écran ».
2. **▶ Démo guidée** (en haut) → Runner Sam lance un bon → Bar Lea « prête » → ça s'affiche chez le serveur *et* le runner → le serveur « Servie ✓ » → la table change de statut seule.
3. **＋ Commande** sur un carré VIP → tape une vodka → les mixers sont proposés automatiquement.

---

## 🏗️ Architecture & code

Tout tient dans **`index.html`** (HTML + CSS + JS *inline*). L'organisation est pensée pour être
**« backend-shaped »** : toute mutation passe par une couche `api` qui vérifie les autorisations,
et le modèle de données suit un **schéma cible** prêt à être branché sur une vraie base.

**Repères dans le fichier** (cherchez ces symboles) :

| Ce que tu veux changer | Où, dans `index.html` |
|---|---|
| Nom / ville / libellés de rôles de l'établissement | `const ESTAB = { … }` |
| Données de démo (zones, tables, produits, staff, commandes, notifs) | `function seed()` |
| Permissions par rôle (RBAC) | `const CAN = { … }` + `function can()` |
| Accords bouteille → mixers | `const PAIRINGS` + `mixersFor()` |
| Statuts de table & couleurs | `const STATUS`, `const NTYPE` |
| Position d'une table sur le plan | champ `x`/`y` de la table dans `seed()` (repère du plan dans `renderMap`) |
| Toute action métier (avec contrôle d'accès) | `const api = { … }` |

**Concepts clés :**
- **RBAC** — `can(role, action)` est la source de vérité ; chaque méthode de `api` la vérifie. Exemple : `api.encaisser` refuse un Runner **même appelé directement** (le blocage argent n'est pas qu'un `display:none`).
- **Résolution d'assignation** — `resolveAssignment(table)` : `override actif` → `zone du shift` → `non assignée`.
- **Notifications** — `escalate()` (urgence/priorité dans le temps), `canResolveNotif()` (qui peut clôturer quoi), `todoRelevant()` (à qui l'afficher), `syncTableFromResolvedNotif()` (lien statut-alerte ↔ statut-table).
- **Modèle de données (schéma cible)** — `Staff, Zone, ShiftAssignment, TableOverrideRequest, TableOverride, StaffPermissionOverride, Table, Order, Product, Notification`.
- **Persistance** — `load()/save()` sur `localStorage` (clé `servemap_mirano_v2` — la **bumper** si tu changes le seed, sinon l'ancien état persiste).

---

## 🏢 Adapter à un autre établissement

Le moteur est générique : **un établissement = le bloc `ESTAB` + le `seed()`**.
1. Modifier `ESTAB` (nom, ville, libellés de rôles).
2. Dans `seed()` : redéfinir les **zones** (nom + rectangle `x,y,w,h`), les **tables** (nom, zone, position `x,y`, `min` bouteille), la **carte** (`products`), le **staff** et les **assignations**.
3. **Bumper `LS_KEY`** pour forcer le rechargement.

Aucune autre partie du code n'est à toucher pour un rebranding.

---

## ✅ Réel vs exemple (profil Mirano)

Calé sur l'**identité publique** de Mirano (source : miranobrussels.com) :
- **Vérifié** : identité **or & noir**, néons, **ancien cinéma Art Déco** ; architecture **parterre + balcon** (d'où les loges) ; **Bruxelles**, club le week-end.
- **Exemple à confirmer** (le site ne les publie pas) : **plan détaillé**, **noms exacts** des espaces/bars, **carte**, **tarifs**, **minimums bouteille**, capacité. Le Fumoir et les positions précises sont supposés.

👉 Pour une version fidèle : fournir le **vrai plan** (emplacement des carrés/loges/bar) et la **vraie carte** ; il suffit alors d'éditer `ESTAB` + `seed()`.

---

## 🚀 Déployer (obtenir une URL publique)

Le site est **statique** (un fichier) → il se déploie partout.

**Option A — GitHub Pages (recommandé, gratuit).**
1. Créer un dépôt GitHub et y pousser ce dossier.
2. Repo → *Settings* → *Pages* → *Build and deployment* → *Deploy from a branch* → branche `main`, dossier `/root`.
3. L'URL publique apparaît en haut de la page Pages (type `https://<user>.github.io/servemap/`).
   *(Pages est gratuit sur dépôt **public** ; sur dépôt privé il nécessite un plan payant.)*

**Option B — Netlify Drop (sans dépôt, ~30 s).**
Aller sur `app.netlify.com/drop` et **glisser `index.html`** → URL instantanée.

**Option C — N'importe quel hébergement statique** (Vercel, Cloudflare Pages, un simple serveur web) : déposer `index.html` à la racine.

---

## 🧭 Passation développeur / prochaines étapes

Cette démo est **front-only** ; le schéma de données est déjà la cible d'un vrai produit. Ordre logique pour industrialiser :
1. **Backend + base de données** reprenant le modèle ci-dessus ; déplacer `CAN`/`can()` et les méthodes `api.*` côté serveur (les vérifs d'accès doivent vivre sur le backend).
2. **Comptes & authentification** par employé (chacun son appareil, plus de sélecteur « Je suis »).
3. **Temps réel multi-appareils** (WebSocket/SSE) : les alertes et statuts se propagent en direct entre serveurs, runners, bar, responsable.
4. **Multi-établissements** (multi-tenant) : le bloc `ESTAB` devient une configuration d'établissement en base.
5. Historique/analytics de service, gestion fine du stock, addition/impression, intégration caisse.

---

## 📁 Structure du dépôt

```
ServeMap/
├── index.html    # l'application complète (HTML + CSS + JS, aucune dépendance)
├── README.md     # ce fichier
└── .gitignore
```

---

## 📄 Licence & contact

Prototype privé — usage et diffusion à définir par le propriétaire.
Contact projet : jonathandann43@gmail.com
