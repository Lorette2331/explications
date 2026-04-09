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
## 🧠 Plan détaillé de l’algorithme

### Introduction de la méthode
Le projet repose sur une approche de vision par ordinateur hybride : nous combinons un algorithme d'apprentissage non supervisé (**K-Means**) avec des techniques de filtrage spatial et morphologique.

---

## Étape 1 : Définition de la Zone d'Intérêt (ROI)

Pour optimiser le traitement et éviter les erreurs dues au décor (ciel, côtes, pontons), l'algorithme ne travaille pas sur toute l'image.

- **Le principe :** On définit un rectangle horizontal qui correspond uniquement à la surface de l'eau.  
- **L'avantage :** Cela réduit drastiquement le "bruit" visuel et permet à l'IA de se concentrer sur les contrastes entre l'eau et les objets flottants.

---

## Étape 2 : Segmentation par K-Means

C'est l'étape de simplification mathématique.

- **La logique :** On demande à l'algorithme de regrouper les pixels de la zone en K groupes (clusters) de couleurs.  

- **L'analyse du "Bleu" :** Parmi ces groupes, l'un d'eux va majoritairement représenter l'eau (la couleur dominante, bleue ou grise). Un autre groupe représente les objets clairs (blancs ou métalliques).  

- **Binarisation :** Une fois les clusters identifiés, on crée un masque noir et blanc.  
  Tout ce qui ressemble au "cluster bleu" de l'eau est transformé en noir (supprimé).  
  Tout ce qui tranche avec ce bleu devient blanc (candidat à la détection).

---

## Étape 3 : Filtrage et Nettoyage Morphologique

À ce stade, l'image contient encore des erreurs (reflets de soleil, écume).

- **Opérations de morphologie :**  
  On utilise des fonctions de **dilatation** et d'**érosion**.  

  - La dilatation permet de "souder" les morceaux d'un même bateau qui auraient été séparés  
  - L'érosion permet d'effacer les tous petits points blancs isolés qui ne sont que du bruit  

- **L'objectif :** Obtenir des taches blanches pleines et nettes sur un fond noir parfait.

---

## Étape 4 : Détection de contours et Analyse de surface

L'ordinateur va maintenant transformer ces taches en objets identifiés.

- **Extraction des contours :**  
  On utilise un algorithme qui convient aux frontières des zones blanches pour en calculer la surface et les coordonnées.  

- **Filtrage par taille :**  
  Pour chaque forme trouvée, on vérifie sa dimension.  
  Si elle est trop petite pour être un bateau (selon un seuil que nous avons défini), elle est rejetée.  

➡️ C'est le filtrage final du bruit.

---

## Étape 5 : Marquage et Localisation (Bounding Boxes)

Dernière étape technique : la matérialisation.

- **Encadrement :**  
  Pour chaque objet validé, on calcule le rectangle englobant (**Bounding Box**).  

- **Positionnement :**  
  On récupère les coordonnées du centre de chaque rectangle pour confirmer que l'objet est bien situé dans la zone de l'eau.  

- **Résultat :**  
  L'algorithme renvoie le nombre exact d'objets détectés et leurs positions précises.
