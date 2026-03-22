# 📦 Guide : apt.txt + environment.yml + postBuild

## 📋 Vue d'Ensemble

Vous avez **2 versions** disponibles :

### 1️⃣ ULTRAMINIMAL ⭐ RECOMMANDÉE
- **Kernels** : Python + Bash (SANS kernel R)
- **enrichR** : Via Rscript depuis bash
- **Plus rapide** à builder

### 2️⃣ MINIMAL
- **Kernels** : Python + Bash + R
- **enrichR** : Notebooks R OU Rscript
- **Plus flexible**

---

## 📦 Fichiers Nécessaires

### Pour ULTRAMINIMAL ⭐

```
mon-environnement/
├── apt.txt                          # ← Dépendances système
├── environment_ULTRAMINIMAL.yml     # ← Renommer en environment.yml
└── postBuild_ULTRAMINIMAL           # ← Renommer en postBuild
```

### Pour MINIMAL

```
mon-environnement/
├── apt.txt                          # ← Dépendances système
├── environment_MINIMAL.yml          # ← Renommer en environment.yml
└── postBuild_MINIMAL                # ← Renommer en postBuild
```

---

## 🚀 Utilisation (ULTRAMINIMAL Recommandée)

### Étape 1 : Télécharger les Fichiers

Téléchargez depuis `/outputs/` :
- `apt.txt`
- `environment_ULTRAMINIMAL.yml`
- `postBuild_ULTRAMINIMAL`

### Étape 2 : Renommer les Fichiers

```bash
# Renommer environment.yml
mv environment_ULTRAMINIMAL.yml environment.yml

# Renommer postBuild
mv postBuild_ULTRAMINIMAL postBuild

# Rendre postBuild exécutable
chmod +x postBuild
```

### Étape 3 : Structure Finale

```
mon-environnement/
├── apt.txt              # ✅
├── environment.yml      # ✅
└── postBuild            # ✅ (exécutable)
```

### Étape 4 : Utiliser avec Votre Plateforme

Ces fichiers sont utilisés par des plateformes comme :
- **repo2docker**
- **Binder**
- **JupyterHub**
- **MyBinder.org**

Exemple avec repo2docker :
```bash
repo2docker --no-run .
```

---

## 📋 Contenu des Fichiers

### apt.txt

**Rôle** : Installe les dépendances système (via apt-get)

**Contenu** :
- Éditeurs : vim, nano
- Outils : git, wget, curl, tree, htop
- Compilateurs : gcc, g++, make
- Bibliothèques : libncurses, libcurl, zlib, libxml2
- Fonts : dejavu, liberation
- BLAST : ncbi-blast+
- Java : default-jre, default-jdk

**Quand s'exécute** : Avant conda

---

### environment.yml

**Rôle** : Installe les packages conda/pip

**Différences ULTRAMINIMAL vs MINIMAL** :

| Package | ULTRAMINIMAL | MINIMAL |
|---------|--------------|---------|
| r-base | ✅ | ✅ |
| r-irkernel | ❌ | ✅ |
| r-essentials | ❌ | ✅ |

**Outils bioinformatiques** (identiques) :
- FastQC, MultiQC, Qualimap
- Fastp, Trimmomatic
- BWA, Samtools
- SPAdes
- QUAST
- BLAST, MLST
- Prokka, Bakta
- Roary
- Perl + modules

**Quand s'exécute** : Après apt.txt

---

### postBuild

**Rôle** : Configuration post-installation

**Actions** :
1. Installation kernels Jupyter
   - ULTRAMINIMAL : Python + Bash
   - MINIMAL : Python + Bash + R
2. Installation enrichR (package R)
3. Configuration Prokka (base de données)
4. Création script enrichr_shell.sh
5. Création répertoires (~/data, ~/notebooks, ~/results, ~/blast_db)
6. Build JupyterLab

**Quand s'exécute** : Après environment.yml

---

## 🔧 Ordre d'Exécution

```
1. apt.txt
   ↓ Installation dépendances système
   
2. environment.yml
   ↓ Installation conda/pip
   
3. postBuild
   ↓ Configuration finale
   
4. JupyterLab prêt !
```

---

## 📊 Comparaison des Versions

| Aspect | ULTRAMINIMAL ⭐ | MINIMAL |
|--------|----------------|---------|
| **apt.txt** | Identique | Identique |
| **environment.yml** | Sans r-irkernel | Avec r-irkernel |
| **postBuild** | Kernel bash + Python | Kernels bash + Python + R |
| **Temps build** | Plus rapide (~25-35 min) | Plus long (~35-45 min) |
| **enrichR** | Via Rscript | Rscript OU notebooks R |
| **Simplicité** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

---

## ✅ Vérification Post-Build

### Vérifier les Kernels

```bash
jupyter kernelspec list

# ULTRAMINIMAL devrait afficher :
# Available kernels:
#   python3    /path/to/python3
#   bash       /path/to/bash

# MINIMAL devrait afficher :
# Available kernels:
#   python3    /path/to/python3
#   bash       /path/to/bash
#   ir         /path/to/R
```

### Vérifier les Outils

```bash
# Outils bioinformatiques
fastqc --version
trimmomatic -version
spades.py --version
quast.py --version
blastn -version
prokka --version
roary --version

# R et enrichR
R --version
Rscript -e "library(enrichR)"
```

### Vérifier le Script enrichR

```bash
ls -la ~/enrichr_shell.sh
# Devrait être exécutable (-rwxr-xr-x)

# Tester
echo -e "TP53\nBRCA1" > test_genes.txt
~/enrichr_shell.sh -f test_genes.txt -o test_results/
```

---

## 🆘 Dépannage

### Problème : enrichR ne s'installe pas

```bash
# Réinstaller manuellement
Rscript -e "install.packages('enrichR', repos='https://cloud.r-project.org/', dependencies=TRUE)"
```

### Problème : Kernel R absent (MINIMAL)

```bash
# Réinstaller kernel R
Rscript -e "IRkernel::installspec(user = FALSE)"
```

### Problème : Prokka database

```bash
# Reconfigurer
prokka --setupdb
```

### Problème : postBuild ne s'exécute pas

```bash
# Vérifier permissions
chmod +x postBuild

# Exécuter manuellement
./postBuild
```

---

## 📖 Exemples d'Utilisation

### Utilisation Locale (repo2docker)

```bash
# 1. Installer repo2docker
pip install jupyter-repo2docker

# 2. Builder l'image
cd mon-environnement/
repo2docker --no-run .

# 3. Lancer le container
docker run -p 8888:8888 -v $(pwd)/data:/home/jovyan/data <image_id>
```

### Utilisation MyBinder

1. Créer un dépôt Git avec les 3 fichiers
2. Aller sur https://mybinder.org
3. Entrer l'URL du dépôt
4. Cliquer "launch"

---

## 💡 Conseils

### Pour ULTRAMINIMAL ⭐

**Avantages** :
- ✅ Plus rapide à builder
- ✅ Plus léger
- ✅ Plus simple (un seul workflow bash)
- ✅ enrichR fonctionne via script

**Utiliser si** :
- Vous préférez bash
- Vous voulez la rapidité
- Vous n'avez pas besoin de notebooks R

### Pour MINIMAL

**Avantages** :
- ✅ Notebooks R interactifs
- ✅ Plus de flexibilité
- ✅ Analyse interactive en R

**Utiliser si** :
- Vous aimez R interactif
- Vous voulez explorer les données
- Le temps de build ne vous dérange pas

---

## 🎯 Workflows Typiques

### Workflow ULTRAMINIMAL (Bash uniquement)

```bash
# Dans notebook Bash

# 1. Quality control
fastqc *.fastq.gz -o results/fastqc/

# 2. Trimming
trimmomatic PE input_R1.fq.gz input_R2.fq.gz ...

# 3. Assembly
spades.py -1 R1.fq.gz -2 R2.fq.gz -o assembly/

# 4. Annotation
prokka assembly/contigs.fasta --outdir annotation/

# 5. Pan-genome
roary *.gff -f roary_output/

# 6. enrichR (via Rscript)
~/enrichr_shell.sh -f core_genes.txt -o enrichr_results/
```

### Workflow MINIMAL (Bash + R)

```bash
# Notebook Bash : Pipeline bioinformatique
fastqc → trimmomatic → spades → prokka → roary
```

```r
# Notebook R : Analyses
library(enrichR)
library(tidyverse)

pangenome <- read_csv("roary/gene_presence_absence.csv")
core <- filter(pangenome, n_strains >= 0.99 * total)

enriched <- enrichr(core$Gene, "GO_Biological_Process_2023")
ggplot(enriched[[1]], ...) + ...
```

---

## 📝 Résumé

**Fichiers** :
- ✅ `apt.txt` - Dépendances système (identique pour les deux)
- ✅ `environment.yml` - Packages conda (ULTRAMINIMAL ou MINIMAL)
- ✅ `postBuild` - Configuration (ULTRAMINIMAL ou MINIMAL)

**Recommandation** :
- 🌟 **ULTRAMINIMAL** pour la plupart des cas
- ⭐ **MINIMAL** si vous voulez notebooks R

**Temps** :
- ULTRAMINIMAL : ~25-35 min
- MINIMAL : ~35-45 min

---

**Vos fichiers sont prêts ! Choisissez votre version et buildez ! 🚀**
