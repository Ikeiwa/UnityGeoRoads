# UnityGeoRoads

## Récupération de données

Nous avons dû récupérer des données GeoGML pour créer la route, étant donné que Unity ne connaît pas ce format de fichier, nous devions trouver une solution. 
Soit écrire un parseur de ce genre de données, soit trouver une librairie existante. 
Nous avons opté pour la seconde solution et modifié un peu le parseur pour permettre certains types non pris en compte (lien parseur).

## Conversion des données

Les données étant au format GPS, nous avons dû les convertir. 
Cependant, Unity gère les nombres flottant en 32bits et les données passées sont bien plus élevées. 
Il y aura donc une perte de données obligatoire ([lien explication]()). 
Tout comme pour la lecture des données, une librairie l'avait déjà fait dans les issues du parseur de GeoGML ([lien convertisseur GPS to UCS]()).

private void FindMetersPerLat(float lat) // Compute lengths of degrees
{
    float m1 = 111132.92f;    // latitude calculation term 1
    float m2 = -559.82f;        // latitude calculation term 2
    float m3 = 1.175f;      // latitude calculation term 3
    float m4 = -0.0023f;        // latitude calculation term 4
    float p1 = 111412.84f;    // longitude calculation term 1
    float p2 = -93.5f;      // longitude calculation term 2
    float p3 = 0.118f;      // longitude calculation term 3

    lat = lat * Mathf.Deg2Rad;

    // Calculate the length of a degree of latitude and longitude in meters
    metersPerLat = m1 + (m2 * Mathf.Cos(2 * (float)lat)) + (m3 * Mathf.Cos(4 * (float)lat)) + (m4 * Mathf.Cos(6 * (float)lat));
    metersPerLon = (p1 * Mathf.Cos((float)lat)) + (p2 * Mathf.Cos(3 * (float)lat)) + (p3 * Mathf.Cos(5 * (float)lat));
}

private Vector3 ConvertGPStoUCS(Vector2 gps)
{
    FindMetersPerLat(_LatOrigin);
    // Calc current lat
    float zPosition = metersPerLat * (gps.x - _LatOrigin);     
    float xPosition = metersPerLon * (gps.y - _LonOrigin);
    return new Vector3((float)xPosition, 0, (float)zPosition);
}

## Création des routes

Avec des données extraites on génère les différentes routes dans unity, pour cela on génère un modèle 3D par route à l’aide d’une liste de points, chaque point contient un vecteur 2D et une hauteur afin de rendre plus simple les calculs.
Le mesh de la route est fait de segments perpendiculaires à la route connectés par deux triangles, la première étape est de trouver leur direction et longueur, dans le cas des points de départ et de fin c’est simple on calcul la normale de la route et on prend sa largeur de base. Cependant la situation se complique dans les coins car le segment doit être plus long sinon la route sera comme pincée.

![parralelle_droite]()

A gauche tous les segments ont la même longueur, à droite le segment du centre est rallongé pour éviter le pincement. Pour cela on utilise une méthode simple, on commence par calculer la tangente du coin:
__(Direction sortie normalisé + Direction entrée normalisé) le tout normalisé__
Ensuite on prend la normale de la tangente (miter), comme on travaille en 2D il suffit de faire __[-tangente.x , tangente.y]__
Et enfin pour sa longueur on divise la largeur de base de la route par le produit scalaire du miter et de la normale de la direction d’entrée.
Pour rajouter un peu plus de qualité on a aussi arrondis le coin extérieur de la route, pour cela il suffit de prendre un cercle au centre du segment avec comme diamètre la largeur de la route et de générer plusieurs points sur l’arc du cercle présent entre la normale du segment d’entrée et de sortie.

![texture_flou]()


