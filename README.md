# Assistant radiologue virtuel responsable

Prototype pédagogique d'IA médicale multimodale pour l'analyse prudente de radiographies thoraciques.

> **Position non clinique.** Ce projet n'est pas un dispositif médical. Il ne doit jamais être utilisé pour diagnostiquer, trier ou orienter un patient. Toute sortie est un résultat expérimental à vérifier par un professionnel qualifié.

---

## Présentation du projet

Ce projet est réalisé dans le cadre du Mastercamp EFREI (Solution Delivery – Filière Data, 2025-2026).

L'objectif est de développer une application web qui reçoit une radiographie thoracique frontale et retourne une sortie JSON structurée contenant : la qualité de l'image, la classe prédite, un score de confiance, des observations visuelles, une justification courte, les limites et un avertissement obligatoire.

| Élément | Détail |
|---|---|
| Entrée | 1 radiographie thoracique frontale |
| Classes de sortie | `normal` / `suspected_opacity` / `uncertain` |
| Finalité | Prototype éducatif, pas d'aide au diagnostic réelle |
| Données | Synthétiques ou publiques, autorisées et dé-identifiées |

---

## Démarrage rapide

```bash
python -m venv .venv
source .venv/bin/activate  # Windows : .venv\Scripts\activate
pip install -r requirements.txt
python eval/run_evaluation.py --mode toy
streamlit run app/streamlit_app.py
```

### API de démonstration

```bash
uvicorn api.main:app --reload
```

Exemple de requête :

```bash
curl -X POST "http://127.0.0.1:8000/predict" \
  -F "file=@data/sample_images/CXR_SYN_002_suspected_opacity.png"
```

### Smoke test

```bash
pip install -r requirements-test.txt
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest -q
python eval/run_evaluation.py --mode toy \
  --out-dir /tmp/assistant-radio-eval \
  --db-path /tmp/assistant-radio-evidence.sqlite
```

---

## Structure du dépôt

```
assistant-radiologue-virtuel/
├── README.md
├── docs/              # appel d'offre, architecture, éthique, évaluation
├── data/              # cas synthétiques et images jouet
├── prompts/           # prompt baseline, prompt amélioré, schéma JSON
├── src/               # inférence jouet, garde-fous, métriques, SQLite
├── api/               # FastAPI (/predict)
├── app/               # Streamlit / Gradio
├── eval/              # évaluation, sorties CSV/JSON, registre d'erreurs
├── tests/             # smoke tests et contrat minimal
├── notebooks/         # notebooks de démarrage
└── finetuning/        # stubs expérimentaux, non obligatoires
```

---

## Format de sortie (contrat JSON)

```json
{
  "image_quality": "bonne | moyenne | mauvaise",
  "predicted_class": "normal | suspected_opacity | uncertain",
  "confidence": 0.0,
  "visual_evidence": "description factuelle des signes visuels observés",
  "justification": "raisonnement clinique bref reliant les signes à la classe prédite",
  "limitations": "facteurs limitants (qualité, artefacts, incertitudes...)",
  "warning": "si confiance < 0.60 ou image de mauvaise qualité, indiquer l'incertitude et recommander relecture/contrôle"
}
```

**Règle garde-fou :** si `confidence < 0.60`, la classe bascule automatiquement en `uncertain`. La classe `uncertain` n'est pas un échec — c'est un garde-fou.

---

## Plan de travail sur 7 semaines

### S1 — Cadrage + dépôt Git
Créer la structure du projet sur GitHub (`data/`, `prompts/`, `src/`, `api/`, `app/`, `eval/`). Télécharger 20 radiographies depuis le dataset RSNA Pneumonia (Kaggle). Rédiger le document de cadrage : entrée = 1 radio thoracique frontale, sortie = 3 classes. Fixer le périmètre et ne plus le modifier.

**Livrable :** dépôt Git + 20 images smoke test + doc cadrage
**Porte :** GO/NO-GO périmètre

---

### S2 — Baseline reproductible
Rédiger le premier prompt pour MedGemma 4B (Hugging Face). Ce prompt doit demander une réponse en JSON avec la classe prédite, le score de confiance, la justification et le warning. Tester sur les 20 images, enregistrer les résultats dans un CSV, calculer les métriques initiales (accuracy, macro-F1). Il s'agit du point de départ à améliorer.

**Livrable :** notebook baseline + résultats initiaux (métriques)
**Porte :** GO/NO-GO baseline

---

### S3 — Comparaison de prompts
Rédiger 2 autres versions du prompt (plus détaillées, avec instruction sur quand répondre `uncertain`). Faire tourner les 3 prompts sur les mêmes images. Comparer dans un tableau : taux de JSON valide, présence du warning, hallucinations, justification courte. Retenir le meilleur prompt pour la suite.

**Livrable :** tableau de comparaison + analyses (JSON valide ≥ 95%, hallucination = 0)

---

### S4 — Amélioration légère
Ajouter la règle d'incertitude automatique (`confidence < 0.60` → `uncertain`). Option : ajouter un classifieur léger (CNN ou ResNet pré-entraîné) comme support de confiance. Enregistrer tous les résultats dans SQLite. Comparer les métriques avant/après pour démontrer l'amélioration mesurée.

**Livrable :** modèle amélioré + justification + dashboard métriques (CSV + SQLite)
**Porte :** GO/NO-GO évaluation

---

### S5 — Démo web
Mettre en place l'API FastAPI avec un endpoint `/predict` recevant une image et retournant le JSON. Connecter à une interface Streamlit permettant d'uploader une radio et d'afficher le résultat avec warning. Objectif : latence < 10 secondes. Rédiger le guide utilisateur.

**Livrable :** prototype web déployé + guide utilisateur
**Porte :** GO/NO-GO démo

---

### S6 — Analyse d'erreurs
Étudier manuellement 30 cas et catégoriser chaque erreur : faux négatif (anomalie non détectée), faux positif (anomalie inventée), incertitude acceptable, hallucination textuelle. Identifier le top 5 des causes de panne. Proposer une action corrective pour chaque type d'erreur. Constituer le registre des constats (CSV commenté).

**Livrable :** rapport d'erreurs + registre CSV + actions correctives documentées

---

### S7 — Soutenance finale
Préparer les slides et une courte vidéo démo. Finaliser le rapport : dataset utilisé, prompts, métriques, erreurs, limites et risques. Vérifier que le smoke test passe. Présenter des preuves concrètes : JSON réels, métriques, logs, avertissements et limites explicites.

**Livrable :** slides + vidéo démo + rapport final + dépôt propre
**Porte :** GO/NO-GO défense

---

## Livrables attendus

| Niveau | Attendu |
|---|---|
| MUST | Baseline reproductible, sortie JSON valide, warning obligatoire, logs, métriques, mini-rapport |
| SHOULD | Prompt amélioré, règle d'incertitude, comparaison baseline/amélioration, analyse d'erreurs sur cas commentés |
| COULD | LoRA expérimental (Gemma 4 via Unsloth), MedGemma via PEFT/QLoRA, localisation visuelle, ablations de prompts |

---

## Critères d'évaluation

| Critère | Poids |
|---|---|
| Périmètre + dataset | 15% |
| Baseline fonctionnelle | 15% |
| Amélioration mesurée | 20% |
| Intégration application | 15% |
| Évaluation + erreurs | 20% |
| Éthique + limites | 10% |
| Oral professionnel | 5% |

---

## Références techniques

| Ressource | Usage | Référence |
|---|---|---|
| Unsloth – Gemma 4 | Fine-tuning LoRA/QLoRA expérimental | [Guide Gemma 4](https://unsloth.ai/docs/models/gemma-4/train) |
| MedGemma | Baseline médicale image-texte | [Model card HuggingFace](https://huggingface.co/google/medgemma-4b-pt) |
| MIMIC-CXR | Radiographies thoraciques (accès contrôlé) | [PhysioNet](https://physionet.org/content/mimic-cxr/2.1.0/) |
| CheXpert | Radiographies + rapports associés | [Stanford AIMI](https://aimi.stanford.edu/datasets/chexpert-chest-x-rays) |
| RSNA Pneumonia | Dataset principal recommandé | [Kaggle](https://www.kaggle.com/c/rsna-pneumonia-detection-challenge) |

---

## Points de vigilance

- Ne pas inventer d'information clinique absente de l'image.
- Ne pas supprimer la classe `uncertain` — c'est un garde-fou, pas un échec.
- Ne pas afficher uniquement des réussites en soutenance.
- Ne jamais commiter de données patient réelles, identifiantes ou ambiguës.
- Ne pas présenter le prototype comme validé médicalement.

---

## Licence

Le code pédagogique de ce dépôt est publié sous licence MIT. Les datasets externes, modèles et bibliothèques utilisés conservent leurs licences propres. Vérifier et documenter les droits d'usage avant toute expérimentation.
