# 📖 Guide pour les Collègues — Détection de Fraude Mobile Money

> Ce document explique le projet étape par étape pour quelqu'un qui découvre le code.

---

## 🎯 C'est quoi le problème ?

On a des **millions de transactions mobile money** (envois d'argent, paiements, dépôts...).  
Parmi elles, une très petite partie est **frauduleuse**.  
Notre mission : **trouver ces fraudes** avant qu'elles causent des dommages.

Le modèle doit donner pour chaque transaction une **probabilité** :
- Proche de **0** → probablement normale
- Proche de **1** → probablement frauduleuse

---

## 📊 Les données — C'est quoi chaque colonne ?

| Colonne | Type | Signification |
|---------|------|---------------|
| `id` | texte | Identifiant unique de la transaction |
| `period` | nombre | Période temporelle simulée |
| `operation` | texte | Type d'opération anonymisé (op_01, op_02...) |
| `amount` | décimal | Montant de la transaction (rescalé) |
| `origin_account` | texte | Compte qui envoie l'argent (anonymisé) |
| `origin_balance_before` | décimal | Solde du compte émetteur **avant** la transaction |
| `origin_balance_after` | décimal | Solde du compte émetteur **après** la transaction |
| `destination_account` | texte | Compte qui reçoit l'argent (anonymisé) |
| `destination_balance_before` | décimal | Solde du compte destinataire **avant** la transaction |
| `destination_balance_after` | décimal | Solde du compte destinataire **après** la transaction |
| `fraud_flag` | 0 ou 1 | **Cible** — 1 = fraude, 0 = normal (absent dans test.csv) |

---

## 🔍 Ce qu'on a découvert dans les données (EDA)

### Le problème du déséquilibre
```
Transactions normales  : ~99%  ████████████████████████████████████████ 
Transactions frauduleuses : ~1%  █
```
C'est pour ça qu'on utilise **PR-AUC** et pas l'accuracy classique.

### Signal clé : les anomalies de balance
En théorie :
```
solde_après = solde_avant - montant  (pour l'émetteur)
solde_après = solde_avant + montant  (pour le destinataire)
```
Quand cette équation ne se vérifie pas → **signal fort de fraude**.

---

## ⚙️ Les features créées — Pourquoi ?

### 1. Anomalies de balance
```python
origin_balance_error = |balance_after - (balance_before - amount)|
```
Si > 0 : quelque chose d'anormal s'est passé dans la transaction.

### 2. Comportement du compte (agrégations)
```
origin_tx_count      → Ce compte fait-il beaucoup de transactions ?
origin_unique_dest   → Envoie-t-il de l'argent à beaucoup de destinataires différents ?
origin_mean_amount   → Quel est son montant habituel ?
```
Un compte qui envoie soudainement beaucoup à de nouveaux destinataires = suspect.

### 3. Vélocité
```
origin_tx_count_period → Combien de transactions ce compte a-t-il faites dans cette période ?
```
Beaucoup de transactions en peu de temps = comportement inhabituel.

### 4. Z-score du montant
```
amount_zscore_in_op = (montant - moyenne_de_l'opération) / écart-type
```
Si = 3 : ce montant est 3 fois plus grand que d'habitude pour ce type d'opération.

### 5. Target Encoding (V2 uniquement)
```
te_origin_account = taux_de_fraude_historique_de_ce_compte
```
Si un compte a déjà été impliqué dans des fraudes → probabilité plus haute.

> ⚠️ **Important** : le Target Encoding est calculé **à l'intérieur de chaque fold** pour éviter de "tricher" en utilisant la cible pour prédire la cible.

---

## 🤖 Comment fonctionne le modèle ?

### LightGBM (le modèle principal)
C'est un algorithme de **gradient boosting** :
- Il construit des centaines d'arbres de décision en séquence
- Chaque arbre corrige les erreurs du précédent
- Résultat : un modèle très puissant sur les données tabulaires

```
Transaction → [Feature 1, Feature 2, ..., Feature N]
                         ↓
               Arbre 1 → Arbre 2 → ... → Arbre 500
                         ↓
               Probabilité de fraude (0 à 1)
```

### Pourquoi 5 folds ?
```
Données train
┌──────────────────────────────────────────────┐
│ Fold 1 │ Fold 2 │ Fold 3 │ Fold 4 │ Fold 5  │
└──────────────────────────────────────────────┘

Itération 1 : entraîne sur [2,3,4,5], évalue sur [1]
Itération 2 : entraîne sur [1,3,4,5], évalue sur [2]
...etc.
```
On obtient ainsi une évaluation **fiable et sans biais** de la performance.

---

## 📈 Calibration — Pourquoi c'est important ?

Sans calibration, le modèle peut prédire 0.9 même si seulement 60% des cas sont des fraudes.

Avec calibration (Platt Scaling / Régression Isotonique) :
```
Score brut 0.9 → Probabilité calibrée 0.60
```
Les probabilités **correspondent à la réalité**.

---

## 🗂️ Ordre d'exécution des notebooks

```
1. fraud_detection.ipynb      → Version de base, bonne pour comprendre
2. fraud_detection_v2.ipynb   → Version avancée, meilleur score
```

**Pour exécuter :**
1. Ouvrir dans VS Code ou Jupyter
2. Sélectionner le kernel `base (Python 3.13.5)` (Anaconda)
3. `Run All` (ou Shift+Entrée cellule par cellule)
4. Récupérer `submission_v2.csv` et soumettre

---

## ❓ Questions fréquentes

**Q : Pourquoi train.csv et test.csv ne sont pas sur GitHub ?**  
R : Ils font 155 MB et 52 MB — GitHub refuse les fichiers > 100 MB. Télécharge-les depuis la plateforme.

**Q : C'est quoi Optuna ?**  
R : Un outil qui teste automatiquement des centaines de combinaisons d'hyperparamètres et garde la meilleure.

**Q : C'est quoi scale_pos_weight ?**  
R : Un paramètre qui dit au modèle "les fraudes sont rares, donne-leur plus d'importance". Sans ça, le modèle ignorerait les fraudes.

**Q : Pourquoi les comptes sont anonymisés ?**  
R : Pour protéger les données personnelles des utilisateurs.

---

*Document rédigé par AKE IVAN JUNIOR — Hackathon Fraude Mobile Money 2026*
