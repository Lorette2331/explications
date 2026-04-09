# explications
# ⚓ Projet de Détection d'Objets Maritimes

Ce document récapitule les verrous techniques rencontrés lors du développement et les solutions apportées pour optimiser l'algorithme de détection.

---

## 1. Problème de rapidité de traitement
### 🔒 Le verrou
Au début, le traitement de l’image complète était trop lent. L’algorithme **K-Means** devait analyser tous les pixels simultanément (plusieurs millions), ce qui bloquait l'affichage et créait une attente trop longue pour l'utilisateur.

### ✅ La solution
Nous avons divisé l’image en **trois parties verticales** traitées séquentiellement :
* **Ordre de traitement :** Droite ➡️ Centre ➡️ Gauche.
* **Affichage progressif :** Le site affiche l'état d'avancement (Tiers 1/3, 2/3, puis 3/3).
* **Résultat :** Une meilleure rapidité perçue et une réduction de la charge de calcul instantanée.

---

## 2. Précision de la détection
### 🔒 Le verrou
L'algorithme générait des **faux positifs** en confondant les bateaux avec :
* Des rochers ou des parties de la côte.
* Des reflets du soleil sur l'eau.

### ✅ La solution
Mise en place de deux couches de filtrage :
1.  **Zone d'eau uniquement :** Utilisation des clusters K-Means pour identifier la couleur de l'eau et limiter la détection à cette zone.
2.  **Filtres morphologiques :**
    * Validation de la **surface** (minimum / maximum).
    * Contrôle de la **forme** (ratio largeur/hauteur cohérent).

---

## 3. Volume de données à traiter
### 🔒 Le verrou
Les images haute résolution originales saturaient les ressources de calcul.

### ✅ La solution
Application d'un redimensionnement (downscaling) systématique avant le traitement :


# Facteur de réduction appliqué
scale = 0.65

---

## 3. Optimisation du Volume de Données (Suite)
L’image est redimensionnée pour devenir plus petite tout en conservant les informations structurelles essentielles. 

**Bénéfices du redimensionnement :**
* **Réduction drastique** du nombre de pixels à analyser.
* **Accélération** significative des calculs mathématiques.
* **Maintien d'une précision optimale** pour la détection des contours.

---

## 4. Correction des erreurs de détection
### 🔒 Le verrou
Même avec les filtres automatisés, certaines détections restaient incorrectes (cas complexes ou reflets atypiques). Il était indispensable de permettre une validation par l'œil humain.

### ✅ La solution : Système d’annotation manuelle
Nous avons intégré un module d’interaction directe sur l'interface :
* **Action :** L'utilisateur clique sur l'objet détecté.
* **Classification :** Sélection du type d'objet : `Bateau`, `Bouée` ou `Faux positif`.
* **Persistance :** Les annotations sont sauvegardées dans un fichier structuré : `annotations.json`.

**Apprentissage progressif :**
Lors des analyses suivantes, le programme relit ce fichier pour corriger ses prédictions et améliorer la classification globale des objets de manière itérative.

---

## 5. Ergonomie et Expérience Utilisateur (UX)
### 🔒 Le verrou
Initialement, le site se contentait d'afficher les images brutes. Il était difficile pour l'utilisateur de suivre l'avancement du traitement ou d'intervenir en cas d'erreur.

### ✅ La solution : Interface Interactive
L'interface a été enrichie avec les fonctionnalités suivantes :
* **Double vue :** Affichage comparatif (Image originale vs Image résultat).
* **Feedback dynamique :** Barre de chargement progressif par tiers d'image.
* **Outils d'examen :** Possibilité d'agrandir les images pour inspecter les détails.
* **Labellisation par clic :** Intégration de boutons de classification directement sur l'image.

---

## 💡 Conclusion
Grâce à ces optimisations techniques et ergonomiques, nous avons réussi à bâtir un système robuste capable de :
1.  **Accélérer** le traitement de données massives.
2.  **Améliorer** la précision de détection via des filtres morphologiques.
3.  **Corriger** les erreurs grâce à une boucle de rétroaction humaine.
4.  **Rendre le système interactif** et agréable à utiliser.

**Le projet repose désormais sur un workflow complet :**
> Segmentation (K-Means) ➡️ Détection d'objets ➡️ Annotation manuelle ➡️ Apprentissage continu.X

---

## 📂 Détail des fonctionnalités par page

### 🏠 Page d'Accueil : Le centre de contrôle
La page d'accueil sert de point d'entrée pédagogique et technique. 
* **Résumé du projet :** Présentation des objectifs de segmentation et de détection.
* **Guide de fonctionnement :** Liste étape par étape du pipeline de traitement (du chargement à la génération du journal).
* **Accès rapide :** Boutons d'action directs pour basculer vers la segmentation ou la galerie.

### 🖼️ Page "Répertoire d'images" : Gestion locale
Cette page est dédiée à la manipulation des fichiers sources avant l'analyse.
* **Visionneuse de Photos Locale :** Un espace pour prévisualiser les images sans avoir besoin de les uploader sur un serveur distant.
* **Sélection de dossier :** Possibilité de cibler un répertoire spécifique sur l'ordinateur.
* **Bouton d'importation :** Une fois les fichiers choisis, ils sont chargés dans l'application pour être prêts pour le traitement K-Means.

### 🧪 Page "Kmeans" : L'intelligence du projet
C'est ici que le moteur de vision par ordinateur entre en scène.
* **Paramétrage :** Réglage des clusters pour la segmentation.
* **Visualisation du traitement :** Affichage des résultats après l'application de l'algorithme.
* **Interactivité :** C'est sur cette page que l'utilisateur peut effectuer les corrections manuelles (annotations) et voir les rectangles de détection s'afficher sur les bateaux identifiés.

---
