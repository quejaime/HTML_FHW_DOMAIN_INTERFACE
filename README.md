Interface Phase II - domaines polygonaux avec trous
===================================================

Ce document decrit l'evolution prevue de l'interface HTML du projet FHW pour
la phase II.

Dans la phase I, l'interface :

    index_polygone_interface.html

permet de saisir un seul polygone de Jordan contenu dans le disque unite,
contenant l'origine. Cette interface correspond au cas simplement connexe.

Dans la phase II, on veut saisir un domaine polygonal multiplement connexe :

    - un bord exterieur polygonal ;
    - un nombre fini de trous polygonaux ;
    - le tout contenu dans le disque unite ;
    - avec l'origine 0 dans le domaine final.

L'interface Phase II est une nouvelle page :

    index_domain_interface.html

L'ancienne interface doit rester disponible et intacte pour le cas simplement
connexe.


1. Objet mathematique vise
--------------------------

Le domaine numerique de depart est de la forme :

    D = interieur(outer_boundary)
        \ (interieur(hole_1) union ... union interieur(hole_h))

ou :

    outer_boundary
        est un polygone de Jordan contenu dans le disque unite ;

    hole_1, ..., hole_h
        sont des polygones de Jordan deux a deux disjoints, strictement
        contenus dans le bord exterieur.

Le nombre h de trous peut etre :

    h = 0, 1, 2, ...

Le cas h = 0 doit redonner le cas simplement connexe de la phase I, mais dans
un format JSON plus general.


2. Workflow de l'interface
--------------------------

L'interface doit fonctionner par etapes.

Etape 0 : choix du nombre de trous

    L'utilisateur choisit un entier h >= 0.

    Tant que h n'est pas fixe, le canevas peut rester inactif.

Etape 1 : saisie du bord exterieur

    L'utilisateur clique dans le disque unite pour definir les sommets du bord
    exterieur.

    La saisie suit les memes principes que dans index_polygone_interface.html :

        - au moins trois sommets avant fermeture ;
        - tous les points dans le disque unite ;
        - pas d'auto-intersection ;
        - fermeture en cliquant pres du premier sommet ;
        - validation finale par winding number autour de l'origine.

    Le bord exterieur doit contenir l'origine.

Etapes suivantes : saisie des trous

    Pour j = 1, ..., h, l'utilisateur saisit le polygone hole_j.

    Chaque trou est saisi comme un polygone ferme, avec au moins trois sommets,
    mais il est soumis a des contraintes supplementaires :

        - le trou ne doit pas s'auto-intersecter ;
        - tous ses sommets doivent etre dans le disque unite ;
        - le trou doit etre strictement contenu dans le bord exterieur ;
        - le trou ne doit pas contenir l'origine ;
        - le trou ne doit pas couper le bord exterieur ;
        - le trou ne doit couper aucun trou deja valide ;
        - le trou ne doit pas etre contenu dans un autre trou deja valide ;
        - aucun trou deja valide ne doit etre contenu dans ce nouveau trou.

    Pour cette phase II, on ne cherche pas a representer des "ilots" dans les
    trous. Les trous sont donc des composantes simples du complementaire, deux
    a deux separees.

Etape finale : export

    Lorsque le bord exterieur et les h trous sont valides, le bouton
    d'export devient actif.

    Le fichier exporte pourra s'appeler par defaut :

        domain.json

    ou, si l'on veut garder une notation proche de la phase I :

        Z_domain.json


3. Contraintes geometriques detaillees
--------------------------------------

3.1. Contraintes sur chaque polygone individuel

Chaque composante de bord doit verifier :

    - au moins trois sommets distincts ;
    - fermeture explicite par repetition du premier point en derniere position ;
    - absence d'auto-intersection ;
    - tous les sommets dans le disque unite.

Comme dans la phase I, la fermeture sera forcee exactement :

    z_p = z_0

lorsque l'utilisateur clique suffisamment pres du premier sommet.


3.2. Contraintes entre composantes

Le bord exterieur et les trous ne doivent pas se couper.

Deux trous distincts ne doivent pas se couper.

Un trou doit etre strictement a l'interieur du bord exterieur. Une strategie
simple est :

    - verifier que tous les sommets du trou sont a l'interieur du bord
      exterieur ;
    - verifier qu'aucun segment du trou ne coupe un segment du bord exterieur.

Pour deux trous deja valides hole_i et hole_j, on verifie :

    - aucune intersection de segments ;
    - aucun sommet de hole_i dans hole_j ;
    - aucun sommet de hole_j dans hole_i.

Cette derniere verification evite les inclusions de trous les uns dans les
autres.


3.3. Origine

L'origine doit appartenir au domaine final D.

Il faut donc verifier :

    - l'origine est a l'interieur du bord exterieur ;
    - l'origine n'est a l'interieur d'aucun trou.


4. Orientation des bords
------------------------

Pour faciliter le futur traitement Python, il faut fixer une convention.

Convention recommandee :

    - bord exterieur : orientation positive, sens trigonometrique ;
    - trous          : orientation negative, sens horaire.

L'interface n'a pas besoin d'obliger l'utilisateur a dessiner dans le bon sens.
Elle peut corriger automatiquement l'orientation au moment de l'export.

Pour cela, on calcule l'aire orientee du polygone :

    signed_area = 1/2 * somme_k (x_k y_{k+1} - x_{k+1} y_k)

Avec les coordonnees mathematiques usuelles :

    - signed_area > 0 : orientation positive ;
    - signed_area < 0 : orientation negative.

Avant export :

    - si le bord exterieur a une aire orientee negative, on inverse l'ordre
      des sommets ;
    - si un trou a une aire orientee positive, on inverse l'ordre des sommets.

Apres inversion, il faut conserver la fermeture exacte :

    dernier point = premier point.


5. Format JSON propose
----------------------

Le format recommande est explicite et lisible :

    {
      "format": "FHW_POLYGONAL_DOMAIN",
      "version": 1,
      "bounded_by_unit_disk": true,
      "contains_origin": true,
      "number_of_holes": 2,
      "orientation_convention": {
        "outer_boundary": "counterclockwise",
        "holes": "clockwise"
      },
      "outer_boundary": [
        {"re": 0.8, "im": 0.0},
        {"re": 0.2, "im": 0.7},
        {"re": -0.7, "im": 0.2},
        {"re": 0.8, "im": 0.0}
      ],
      "holes": [
        {
          "label": "hole_1",
          "points": [
            {"re": 0.2, "im": 0.1},
            {"re": 0.3, "im": 0.2},
            {"re": 0.1, "im": 0.25},
            {"re": 0.2, "im": 0.1}
          ]
        },
        {
          "label": "hole_2",
          "points": [
            {"re": -0.35, "im": -0.1},
            {"re": -0.2, "im": -0.05},
            {"re": -0.25, "im": -0.25},
            {"re": -0.35, "im": -0.1}
          ]
        }
      ]
    }

Remarques :

    - outer_boundary est une liste directe de points, car il n'y a qu'un seul
      bord exterieur.

    - holes est une liste d'objets, afin de permettre plus tard d'ajouter des
      metadonnees par trou si necessaire.

    - les points sont stockes sous la forme {"re": ..., "im": ...}, comme dans
      la phase I.

    - number_of_holes doit etre egal a len(holes).


6. Pourquoi ne pas utiliser une seule liste de composantes ?
-----------------------------------------------------------

On pourrait imaginer un format plus abstrait :

    "boundary_components": [
      {"type": "outer", ...},
      {"type": "hole", ...}
    ]

Mais pour le projet FHW, le format separe :

    outer_boundary
    holes

est preferable pour l'instant :

    - il est plus lisible pour les etudiants ;
    - il evite les ambiguites cote Python ;
    - il correspond directement a la description geometrique du domaine ;
    - il rend explicite la difference entre bord exterieur et trous.


7. Algorithmes HTML/JavaScript a reutiliser
-------------------------------------------

Depuis index_polygone_interface.html, on peut reutiliser :

    - conversion pixel -> complexe ;
    - conversion complexe -> pixel ;
    - test du disque unite ;
    - test d'orientation de trois points ;
    - test d'intersection de segments ;
    - test de fermeture proche du premier point ;
    - winding number autour de l'origine ;
    - export JSON par Blob.

Il faudra ajouter :

    - une machine d'etats pour la saisie successive :

          choose_holes_count
          draw_outer_boundary
          draw_hole_1
          ...
          draw_hole_h
          ready_to_export

    - une structure de donnees pour stocker plusieurs polygones ;

    - des tests d'intersection entre le polygone courant et les composantes
      deja validees ;

    - des tests d'inclusion polygonale pour verifier qu'un trou est dans le
      bord exterieur et hors des autres trous ;

    - une normalisation de l'orientation avant export.


8. Structure interne possible de l'interface
--------------------------------------------

Une structure de donnees simple cote JavaScript :

    const domain = {
      outerBoundary: null,
      holes: []
    };

Pendant la saisie :

    let currentPoints = [];
    let requestedHoleCount = 0;
    let currentStage = "choose_holes_count";
    let currentHoleIndex = 0;

Quand un polygone est ferme et valide :

    - s'il s'agit du bord exterieur :

          domain.outerBoundary = normalizedPolygon;
          currentStage = requestedHoleCount === 0
              ? "ready_to_export"
              : "draw_hole";

    - s'il s'agit d'un trou :

          domain.holes.push(normalizedPolygon);
          currentHoleIndex += 1;

          si currentHoleIndex === requestedHoleCount :
              currentStage = "ready_to_export"
          sinon :
              continuer avec le trou suivant


9. Points de vigilance
----------------------

9.1. Bord des trous proche du bord exterieur

Un trou tres proche du bord exterieur peut etre valide mathematiquement, mais
difficile numeriquement pour l'algorithme FHW.

Dans une premiere version, on peut accepter cette situation. Plus tard, on
pourra ajouter une marge minimale entre composantes.


9.2. Trous tres petits

Un trou tres petit risque d'etre mal represente apres les premieres iterations.
L'interface peut simplement l'autoriser, mais le moteur Python devra peut-etre
emettre des diagnostics.


9.3. Orientation utilisateur

Il ne faut pas demander a l'utilisateur de dessiner dans le bon sens. C'est une
contrainte inutilement fragile. L'interface doit corriger l'orientation a
l'export.


9.4. Cas h = 0

Le cas sans trou doit etre accepte et exporte dans le meme format :

    "number_of_holes": 0,
    "holes": []

Cela permet d'utiliser une seule fonction Python future :

    load_domain_json.py

pour les domaines simplement et multiplement connexes.


10. Futur cote Python
---------------------

La prochaine brique Python associee sera probablement :

    load_domain_json.py

Elle devra :

    - lire FHW_POLYGONAL_DOMAIN ;
    - verifier la version ;
    - convertir outer_boundary en tableau numpy complexe ;
    - convertir chaque trou en tableau numpy complexe ;
    - verifier les fermetures ;
    - verifier les orientations ;
    - recalculer les diagnostics geometriques essentiels ;
    - renvoyer une structure simple, par exemple :

          {
            "outer_boundary": np.ndarray,
            "holes": list[np.ndarray]
          }

Il ne faut pas encore melanger cette brique avec le moteur FHW lui-meme.


11. Decision de projet
----------------------

Pour la phase II, la bonne discipline est :

    - conserver index_polygone_interface.html pour la phase I ;
    - creer index_domain_interface.html pour les domaines a trous ;
    - exporter un format JSON clair et testable ;
    - ecrire ensuite un lecteur Python independant ;
    - integrer seulement apres dans le moteur FHW multiplement connexe.

Cette separation permet de garder le projet lisible, testable, et conforme a
l'esprit modulaire du projet FHW.
