# ⬛ Registre Officiel ` XPLT `

Ce dépôt contient le registre public, transparent, vérifiable et append-only du jeton communautaire souverain **`XPLT`**.

Il a été conçu pour garantir :
- la **traçabilité complète** de toutes les émissions, réserves et distributions,
- une **auditabilité ouverte**, sans besoin de blockchain ni smart contract,
- une **résistance aux falsifications** par versionnement Git et signatures cryptographiques.

>  **Ce registre ne repose sur aucune `blockchain` et n'utilise `aucun gas ni frais de réseau` en étant 100% interopérable avec des outils classiques (Git, JSON, shell).**

## ⬛ Principes Fondateurs

### 🟪 Quantité maximale émise
Le nombre total de **tokens `XPLT`** est **strictement plafonné à `1 111 111 111` unités**.  
Aucune émission au-delà de cette quantité n’est possible, ni techniquement ni statutairement.  
Toute nouvelle émission (`mint`) est toujours vérifiable depuis la racine (`/source/`).

### 🟪 Aucun burn
**`XPLT`** ne prévoit **aucun mécanisme de destruction définitive de tokens**.  
Lorsqu’un utilisateur retourne des tokens (désengagement, retrait volontaire, etc.), ceux-ci sont simplement **renvoyés à la réserve centrale** sous forme d’opération `return`.

> Le solde total circulant + la réserve = total émis depuis la source.

### 🟪 Append-only
Les opérations sont enregistrées sous forme de **lignes JSON** (`.jsonl`) et **jamais modifiées ni supprimées**.  
Chaque ajout est **horodaté (`ts`)**, identifié par un numéro de transaction unique (`tx`) et signé (`sig`).  
Les commits sont également **signés GPG** au niveau Git, garantissant l’intégrité temporelle.

> Toute modification a posteriori d’un fichier ou commit annule la validité de l’ensemble du registre.

### 🟪 Simplicité & auditabilité
Le format repose sur :
- des **fichiers texte** (`.jsonl`) — facilement exploitables en ligne de commande, via `grep`, `jq`, `diff`, etc.
- une **structuration temporelle par jour et par utilisateur**
- un **suivi Git complet** (logs, diffs, historique des commits vérifiables)

Cela garantit une transparence maximale, même sans outil spécialisé.

### 🟪 PoWet (Proof of Witness)
Chaque transaction est validée publiquement par un ou plusieurs "forgeurs" via :
- une **signature de commit GPG** (preuve d’émission par un membre autorisé),
- un **hash interne (`sig`)** calculé à partir des champs triés de l'entrée.

Ces preuves permettent de reconstruire la totalité de l’historique, de détecter toute incohérence, et d'identifier chaque témoin d'une opération.

<pre lang="md"><code>```json
 {
  "ts": "2025-06-17T14:52:10.327Z",       "Horodatage ISO 8601"
  "tx": 248,                              "Numéro de transaction unique"
  "op": "envoyer",                        "Type d'opération"
  "from": "alice",                        "Compte émetteur"
  "to": "bob",                            "Compte destinataire"
  "amount": 100,                          "Montant transféré"
  "initiator": "alice",                   "Initiateur"
  "ref": null,                            "Référence externe optionnelle"
  "note": null,                           "Note libre optionnelle"
  "version": 1,                           "Version du format"
  "sig": "f379ea...c345e09"               "Signature" 
 }
 ```</code></pre>

---

## ⬛ Structure du dépôt

Ce dépôt est organisé en **sous-dossiers temporels (`YYYY/MM/YYYY-MM-DD.jsonl`)**, chacun ne contenant que des **append-only logs**.

```plaintext
xplt/
├── source/                        ← Emissions contrôlées depuis la source vers la réserve
│   └── YYYY/MM/YYYY-MM-DD.jsonl
├── reserve/                       ← Registre interne de la réserve (mouvements entrants/sortants)
│   └── YYYY/MM/YYYY-MM-DD.jsonl
├── registre/                      ← Journal public global de toutes les opérations (mint, distribute, return)
│   └── YYYY/MM/YYYY-MM-DD.jsonl
├── balances/                      ← Historique journalier par pseudo (append-only, non-source de vérité)
│   └── <pseudo>/YYYY/MM/YYYY-MM-DD.jsonl
├── balances.json                  ← État courant reconstitué à chaque commit (source temporaire de lecture rapide)
├── README.md                      ← Ce fichier
└── .gitattributes                 ← Configuration `diff=json` pour que Git affiche les `.jsonl` proprement


+---------+         +----------+          +-------------+
| Source  | ───▶──▶ | Réserve  | ───▶──▶ | Utilisateur |
+---------+         +----------+          +-------------+
                       ▲     ▲
                       │     └─ retour via `return`
                       └─ mise en réserve via `source`