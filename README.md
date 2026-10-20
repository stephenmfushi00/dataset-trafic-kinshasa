Dataset de trafic routier adapté à Kinshasa
Ce dépôt accompagne mon mémoire de master sur la proposition et la validation d’un dataset de trafic routier adapté à Kinshasa. Il présente la préparation des données, l’annotation assistée et les expériences réalisées pour étudier leur utilité en détection de véhicules.

YOLOv8n intervient comme outil de préannotation et comme détecteur de référence. Le travail porte sur la constitution et l’évaluation du dataset ; il ne compare pas les différentes versions de YOLO.
De la collecte aux annotations corrigées
Les vidéos ont été collectées sur plusieurs sites de Kinshasa, notamment la 7e Rue, le Bypass, Mont-Ngafula et la Route Universitaire. La collecte s’est étendue sur environ trois semaines, pour une durée cumulée de 188,89 minutes, soit environ 3 h 08 min 53 s.

Les images extraites ont fait l’objet d’un tri et de contrôles : doublons, lisibilité, correspondance entre images et annotations, classes et coordonnées des boîtes. Les effectifs détaillés doivent être lus dans les rapports d’extraction et les manifestes, qui permettent de distinguer images disponibles, sélectionnées et retenues.

L’annotation a progressé en deux lots :
1. Premier lot de 508 images : préannotation par un modèle préentraîné, puis correction dans CVAT. Le premier fine-tuning utilise 346 images d’entraînement, 84 de validation et 78 de test.
2. Deuxième lot de 500 images : préannotation avec le premier modèle adapté, puis correction humaine. Un deuxième fine-tuning repart du premier best.pt, avec les images d’entraînement initiales et les nouvelles images admissibles.
Ces deux lots constituent un noyau de 1 008 images corrigées. Leur correction a demandé environ sept jours : cinq pour le premier lot et deux pour le second. Ces durées décrivent le suivi du projet ; elles ne mesurent pas un gain de productivité par comparaison chronométrée.
Deux étapes d’apprentissage à distinguer
La première étape est une annotation semi-automatique avec réentraînement supervisé itératif : YOLO propose des boîtes, puis les annotations sont corrigées avant leur utilisation.

La seconde étape est un essai de self-training avec pseudo-étiquettes figées : le premier modèle adapté préannote 9 000 images, puis une partie des annotations automatiques est utilisée sans correction humaine dans une expérience d’enrichissement. Les pseudo-étiquettes sont produites une fois avant cet entraînement ; elles ne sont pas actualisées à chaque époque.

Cette distinction explique l’évolution de la méthode : l’assistance à l’annotation précède l’expérience utilisant des données pseudo-étiquetées.
Découpage et expérience finale
Les images sont regroupées par séquence vidéo pour limiter la présence de scènes voisines dans l’entraînement et l’évaluation. Les séquences de validation et de test restent réservées lors de l’ajout de nouvelles images.

Après application des exclusions documentées :
Ensemble	Images retenues	Utilisation
Images corrigées d’entraînement	554	Entraînement des deux modèles finaux
Validation corrigée	84	Suivi de l’apprentissage
Test corrigé	78	Évaluation comparative
Pseudo-images ajoutées à l’entraînement	3 847	Enrichissement du second modèle final


Parmi les 1 008 images corrigées, 292 nouvelles images issues de séquences réservées sont exclues de l’ajout à l’entraînement. Les 716 images conservées se répartissent donc en 554 images d’entraînement, 84 de validation et 78 de test.

Parmi les 9 000 images préannotées, 131 sans détection et 5 022 avec détections provenant de séquences réservées sont exclues. Il reste 3 847 images pseudo-étiquetées pour l’entraînement.

Deux expériences finales repartent indépendamment du même premier modèle adapté :
- Modèle de référence : entraînement sur les 554 images corrigées.
- Modèle avec pseudo-étiquettes : entraînement sur les mêmes 554 images, enrichies de 3 847 pseudo-images, soit 4 401 images.
Les deux modèles sont évalués sur les mêmes ensembles corrigés. Le nombre d’époques effectivement exécutées et certains paramètres choisis par l’optimiseur automatique peuvent différer. La comparaison porte donc sur la procédure d’enrichissement telle qu’elle a été réalisée.
Classes du dataset
Identifiant	Nom dans les annotations	Désignation
0	car	Voiture
1	motorcycle	Moto
2	tricycle	Tricycle
3	minibus	Minibus
4	bus	Bus
5	truck	Camion
6	handcart	Charrette


L’ordre des classes doit rester identique dans les annotations, la configuration du dataset et le modèle.
Résultats des deux premiers fine-tunings
Modèle	Précision test	Rappel test	mAP50 test	mAP50–95 test
Premier fine-tuning	82,75 %	46,53 %	52,33 %	27,94 %
Deuxième fine-tuning	76,04 %	46,92 %	48,21 %	26,33 %


La mAP50–95 de validation progresse de 35,67 % à 38,00 %, mais celle du test diminue. Le premier modèle a donc été retenu pour produire les préannotations des 9 000 images et comme point de départ commun des expériences finales.
Résultats de l’expérience finale
Modèle	Précision test	Rappel test	mAP50 test	mAP50–95 test
Référence : annotations corrigées	81,95 %	48,31 %	52,48 %	27,84 %
Avec pseudo-étiquettes	69,42 %	48,38 %	53,75 %	29,57 %


À partir des valeurs non arrondies, l’enrichissement apporte environ 1,28 point de mAP50 et 1,72 point de mAP50–95. La précision diminue d’environ 12,52 points, tandis que le rappel varie peu. L’ajout de données automatiques apporte donc un gain modeste sur la mAP, avec un compromis défavorable sur la précision rapportée.

Un comptage complémentaire, à un seuil de confiance de 0,25 et une IoU d’appariement de 0,50, donne :
Modèle	Vrais positifs	Faux positifs	Faux négatifs
Référence	652	418	278
Avec pseudo-étiquettes	658	331	272


Ce comptage utilise un seuil fixe et sa propre convention d’appariement. Il ne remplace pas la mAP et ne doit pas être confondu avec les valeurs globales de précision et de rappel rapportées par Ultralytics.

Les résultats présentés proviennent des entraînements et exports du projet. Aucun nouvel entraînement n’a été lancé pour mettre à jour cette documentation.
Codes disponibles et utilisation dans Colab
À ce stade, les codes sont regroupés dans l’archive Codes_memoire_trafic_Kinshasa_GitHub (1).zip, présente à la racine du dépôt. Le dossier scripts/ n’est pas publié séparément à la racine. Il faut télécharger et extraire l’archive, puis vérifier son arborescence avant d’utiliser un chemin d’exécution.

L’organisation décrite pour les premiers traitements est la suivante :
- scripts/00_preannotation_initiale.py : première assistance à l’annotation ;
- scripts/01_fine_tuning.py : lecture des données, audit, découpage par séquence, entraînement et évaluation ;
- scripts/02_preannotation_cvat.py : préannotations et préparation des exports pour CVAT ;
- scripts/03_reentrainement.py : ajout des nouvelles données autorisées, deuxième fine-tuning et comparaison.
Ces scripts correspondent aux premières étapes. Leur présence ne garantit pas que les notebooks de l’expérience finale et des exports détaillés soient inclus dans cette archive.

Pour les traitements Colab :
1. Extraire les codes et repérer le dossier contenant les scripts et les dépendances.
2. Activer un GPU.
3. Installer les dépendances depuis le fichier requirements.txt.
4. Copier config_drive.example.json vers config_drive.json et renseigner les identifiants des fichiers réellement accessibles au compte Google utilisé.
5. Vérifier la présence des images, des annotations, des poids de départ et du manifeste de découpage.
Exemple à adapter au chemin obtenu après extraction :
%pip install -r /content/dataset-trafic-kinshasa/requirements.txt
%run /content/dataset-trafic-kinshasa/scripts/01_fine_tuning.py
Le script de réentraînement exige notamment le best.pt et le split.csv du premier entraînement. Les scripts ciblent Colab et peuvent télécharger des données, lancer un apprentissage et écrire des résultats dans Drive. Les fichiers temporaires de Colab doivent être sauvegardés avant la fermeture de la session.

Ne pas publier de configuration personnelle contenant des informations d’accès. Les images et les poids ne sont pas fournis séparément comme dataset dans ce dépôt.
Limites de la validation
- Une annotation automatique reste une proposition : elle peut contenir des boîtes incorrectes ou oublier des objets.
- Les contrôles de format ne remplacent pas la vérification visuelle.
- Le test contient 78 images et 930 objets, mais seulement deux bus. Les résultats des catégories rares doivent être interprétés avec prudence.
- Le regroupement des images doit correspondre aux séquences vidéo d’origine. Les préfixes ajoutés lors des exports doivent être pris en compte avant de retrouver ces séquences.
- Le test a été consulté pendant les expériences successives. Un nouvel ensemble de séquences réservé à l’avance renforcerait une validation finale indépendante.
- Une seconde annotation indépendante n’a pas été réalisée sur l’ensemble du noyau corrigé.
- La performance d’un détecteur apporte un élément de validation de l’utilité du dataset ; elle ne démontre pas à elle seule sa représentativité pour toute la ville de Kinshasa.
Références méthodologiques
- Adhikari et Huttunen (2021). Iterative Bounding Box Annotation for Object Detection. ICPR. Article.
- Lee (2013). Pseudo-Label: The Simple and Efficient Semi-Supervised Learning Method for Deep Neural Networks. ICML Workshop on Challenges in Representation Learning.
- Varma et al. (2019). IDD: A Dataset for Exploring Problems of Autonomous Navigation in Unconstrained Environments. WACV. Article.
- Documentation Ultralytics : entraînement.
- Documentation CVAT : format YOLO.
