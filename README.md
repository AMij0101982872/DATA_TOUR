# 🏆 Hackathon — Détection de Fraude Mobile Money

> **Compétition de Machine Learning** | Métrique : Average Precision (PR-AUC)  
> **Auteur :** AKE IVAN JUNIOR | **Rang actuel :** 4ème (score : 0.34986251)

---

## 📌 Contexte

Les services de **mobile money** sont au cœur de l'inclusion financière en Afrique.  
Ce hackathon demande de construire un modèle capable de **détecter les transactions frauduleuses** parmi des millions de paiements numériques.

Chaque transaction doit recevoir une **probabilité entre 0 et 1** :
- `0` → transaction très probablement normale
- `1` → transaction très probablement frauduleuse

---

## 📁 Structure du projet

```
dataset_files/
│
├── 📓 fraud_detection.ipynb      ← Pipeline V1 (baseline complet)
├── 📓 fraud_detection_v2.ipynb   ← Pipeline V2 (version optimisée)
│
├── 📊 submission.csv             ← Soumission V1 (rang 4, score 0.34986)
├── 📊 sample_submission.csv      ← Format attendu par la plateforme
│
├── 🖼️ eda_visualizations.png     ← Graphiques d'exploration des données
├── 🖼️ model_evaluation.png       ← Courbes Précision-Rappel + Feature Importance
├── 🖼️ calibration.png            ← Courbe de calibration des probabilités
├── 🖼️ target_distribution.png    ← Distribution de la variable cible
│
└── 📄 README.md                  ← Ce fichier
```

> ⚠️ `train.csv` (155 MB) et `test.csv` (52 MB) ne sont pas sur GitHub (trop lourds).  
> Télécharge-les depuis la plateforme du hackathon.

---

## 🔄 Pipeline Data Science — Étapes

```
Données brutes
     │
     ▼
1. EDA (Exploration)          → Comprendre la structure, détecter les anomalies
     │
     ▼
2. Feature Engineering        → Créer des variables comportementales
     │
     ▼
3. Modélisation (CV 5 folds)  → LightGBM + XGBoost + CatBoost
     │
     ▼
4. Ensemble                   → Combinaison pondérée des modèles
     │
     ▼
5. Calibration                → Transformer les scores en probabilités fiables
     │
     ▼
6. Soumission                 → submission.csv
```

---

## ⚙️ Features créées (Feature Engineering)

| Catégorie | Features | Description |
|-----------|----------|-------------|
| **Anomalies de balance** | `origin_balance_error`, `dest_balance_error`, `has_balance_anomaly` | Écart entre le montant transféré et la variation de solde |
| **Ratios** | `amount_to_origin_balance`, `amount_to_dest_balance` | Proportion du montant par rapport au solde |
| **Indicateurs binaires** | `origin_balance_negative_before`, `dest_balance_zero_before` | Signaux d'alerte simples |
| **Comportement émetteur** | `origin_tx_count`, `origin_mean_amount`, `origin_unique_dest`... | Statistiques globales par compte émetteur |
| **Comportement destinataire** | `dest_tx_count`, `dest_unique_senders`... | Statistiques globales par compte destinataire |
| **Vélocité** | `origin_tx_count_period`, `origin_velocity_ratio` | Fréquence d'activité par période |
| **Z-scores** | `amount_zscore_in_op`, `amount_zscore_origin` | Montant inhabituel pour ce type/compte |
| **Target Encoding** (V2) | `te_operation`, `te_origin_account`, `te_destination_account` | Taux de fraude historique par catégorie |

---

## 🤖 Modèles utilisés

| Modèle | Rôle | Paramètres clés |
|--------|------|-----------------|
| **LightGBM** | Modèle principal | Optimisé avec Optuna (30 essais) |
| **XGBoost** | Diversité d'ensemble | `scale_pos_weight` pour le déséquilibre |
| **CatBoost** | Robustesse | Gestion naturelle des catégorielles |

**Validation croisée :** StratifiedKFold 5 folds (garantit le même ratio fraude/normal dans chaque fold)

---

## 📈 Résultats

| Version | Score PR-AUC | Rang |
|---------|-------------|------|
| V1 (Ensemble LGB+XGB+CAT) | 0.34986251 | **4ème** |
| V2 (LGB + Optuna + Target Encoding) | En cours | 🎯 |

**Score du 1er :** 0.35434681 — Écart à combler : **0.00448**

---

## 🚀 Comment reproduire les résultats

### 1. Installer les dépendances
```bash
pip install lightgbm xgboost catboost optuna scikit-learn pandas numpy matplotlib seaborn
```

### 2. Placer les données
Télécharger `train.csv` et `test.csv` depuis la plateforme et les placer dans ce dossier.

### 3. Exécuter le notebook
```
Ouvrir fraud_detection_v2.ipynb dans Jupyter ou VS Code
→ Kernel : Python 3 (Anaconda base)
→ Run All Cells
→ Le fichier submission_v2.csv sera généré automatiquement
```

---

## 💡 Pourquoi PR-AUC et pas Accuracy ?

Le jeu de données est **fortement déséquilibré** :  
- ~99% de transactions normales  
- ~1% de transactions frauduleuses

Avec l'Accuracy, un modèle qui prédit "tout normal" obtiendrait **99% d'accuracy** sans détecter une seule fraude.

Le **PR-AUC (Average Precision)** mesure la capacité du modèle à **classer les fraudes en haut de la liste de risque**, ce qui est exactement ce qu'on veut en détection de fraude.

---

## 👥 Équipe

| Nom | Rôle |
|-----|------|
| AKE IVAN JUNIOR | Data Scientist — Modélisation & Feature Engineering |

---

*Hackathon organisé dans le cadre de la formation Data Science — HESTIM 2026*
