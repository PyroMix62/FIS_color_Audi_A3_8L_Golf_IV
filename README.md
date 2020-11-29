# FIS_color_Audi_A3_8L_Golf_IV
 Dashboard Audi A3_8L Golf IV screen color

https://youtu.be/0HaM3IB35uY

https://amesis.forumpro.fr/t84-fispluscolor-ecran-couleur-sur-golf-iv-immo-3


FR : Description du projet : Sur le compteur de l’audi A3 8L et Volskwagen Golf IV, Ajout d’un écran TFT couleur en complément ou en remplacement de l’écran par défaut. Utilisation avec l'interface et sélection directe avec le commodo de l'ordinateur de bord
L’objectif est de pouvoir afficher dans le porte instrument les données suivantes : 
- Vitesse instantanée
- Rapport de vitesse engagée
- Distance restante à parcourir (fonction du niveau du réservoir et de la conso moyenne)
- Consommation moyenne depuis le dernier plein d’essence
- Distance parcourue depuis le dernier plein d’essence
- Consommation instantanée
- Conso moyenne et distance parcourue pour un parcours 1, 2 ré-initialisables
Fait : Dépose du compteur d'origine Installation du nouvel écran 1,8”. Création du programme d'affichage. Calcul et affichage vitesse instantanée, rapport engagée, distance restante, distance parcourue, conso moyenne, conso instantanée
A Faire :  intégration du bouton de commande du commodo, récupération des info température, différentes alarmes afin de supprimer définitivement l’écran d’origine pour un plus grand écran unique .
Améliorer le calcul de la consommation instantanée en fonction des paramètres moteur
Composants utilisés: Arduino MKRZero ,Ecran TFT 1,8" 

Projet création ODB pour Audi A3

Le projet va s’articuler autour d’un arduino MkrZero qui est un microcontroleur programmable en C++ et d’un affichage TFT de 1.8’’ couleur
Le but est de pouvoir afficher dans le porte instrument les données suivantes : 
-	Vitesse instantanée
-	Rapport de vitesse engagée
-	Distance restante à parcourir (fonction du niveau du réservoir et de la conso moyenne)
-	Consommation moyenne depuis le dernier plein d’essence
-	Distance parcourue depuis le dernier plein d’essence
-	Consommation instantanée
-	Conso moyenne et distance parcourue pour un parcours 1, 2 ré-initialisables


Récupération des données 

Pour millésime > 2000. Tout se passe sur la prise bleue du porte instrument
Borne 1 : +12 V après contact
Borne 9 : masse
Borne 30 12V permanent 

Borne 25 : signale conso
On récupère des impulsions 12V proportionnelles  à la consommation du moteur par unité de temps
![alt text](https://github.com/AmesisProject/FIS_color_Audi_A3_8L_Golf_IV/blob/main/Photo/1.png) 

Borne 3 : Sortie 1 tachymètre
Idem à la borne 25 : on récupère des impulsions 12V proportionnelles à la vitesse de sortie de boîte de vitesse

Borne 11 : régime moteur : 
On récupère un signal 12V  de forme carrée dont la fréquence est proportionnelle au régime moteur
![alt text]https://github.com/AmesisProject/FIS_color_Audi_A3_8L_Golf_IV/blob/main/Photo/2.png

Borne 5 : Transmetteur de niveau de carburant
On récupère ici un signal numérique de type K-Line. Mais c’est en fait la partie analogique (amplitude du signal) qui transmet le niveau de carburant. Elle varie d’environ 1.5V (vide) à 3V (plein)
![alt text]https://github.com/AmesisProject/FIS_color_Audi_A3_8L_Golf_IV/blob/main/Photo/3.png

Traitement des données

Le montage s’articule autour d’un arduino mkrzero qui possède plusieurs entrées / sorties numériques analogiques ainsi qu’un lecteur de carte µSD. Ce dernier va donc enregistrer les données des différents capteurs, les traités , les enregistrés sur une carte SD (lors de la coupure du contact) et gérer l’affichage sur un petit écran TFT couleur

![alt text]https://github.com/AmesisProject/FIS_color_Audi_A3_8L_Golf_IV/blob/main/Photo/4.png

https://www.arduino.cc/en/Main/ArduinoBoardMKRZero&

Ce qu’il faut retenir est qu’un arduino fonctionne sous 5V et que ses entrées /sorties ne tolère que du 3,3V. Il va donc falloir créer un petit montage (appelé shield ) qui va permettre de connecter l’arduino à la voiture et de les rendre compatibles
![alt text]https://github.com/AmesisProject/FIS_color_Audi_A3_8L_Golf_IV/blob/main/Photo/5.png

Alimentation du montage

Les arduino  possède une mémoire volatile qui s’efface lors de la coupure de leur alimentation. (ce n’est pas  tout à fait vrai, il existe une eeprom mais le nombre de changements de valeurs est  limité avant la destruction du composant) .
On a 3 possibilités 
-	Laisser le montage constamment alimenté grâce au 12V permanent. (non retenu)
-	Utiliser le 12V contact et sauvegarder sur la carte SD en permanence toutes les modifications (non retenu car l’écriture sur la carte SD ralentie fortement le programme)
-	Laisser le temps à l’arduino de sauvegarder sur la carte SD les données avant de s’éteindre (solution retenue)

Pour cela l’arduino sera alimenté par un régulateur de tension 7805 T1 de 5V et stabilisé par un petit condensateur  C2

Lorsque le contact est mis le régulateur est directement alimenté par le +12V contact (avec une diode D2 pour éviter les retours de courant) et délivre 5V sur la borne d’alimentation Vin de l’arduino
Un pont diviseur de tension formé avec les résistances R3 R4 indique à l’arduino (borne 7 input) la présence du +12V contact 
Le programme de l’arduino impose alors à sa borne 11 (output) une sortie haute de 3,3V

Les transistors T1 et T2 agissent comme un interrupteur et laissent à leur tour passé le courant (12V permanent) via la diode D1

Lorsque l’on coupe le contact le courant ne passe plus au travers de la diode D2. Mais le régulateur est toujours alimenté par D1 tant que l’arduino imposera une tension de 3,3V à sa borne 11
L’arduino est programmé pour détecter la coupure de courant sur sa borne 7 (via une interruption) . Il exécute alors la procédure de sauvegarde sur la carte SD et bascule ensuite la sortie 11 à 0V ce qui a pour effet de  couper l’alimentation via les transistors T1et T2. L’arduino s’éteint  jusqu’au prochain allumage

Transformation des signaux d’entrées

Les signaux 12V de l’entrée tachymètre et régime moteur sont simplement rabaissées à 3V par deux ponts diviseur de tension (R5 & R8 pour le tachy et R6 & R9 pour le régime moteur) et connectés respectivement aux bornes 6 et 5 de l’arduino

Le signal conso étant très sensible à l’impédance connecté en parallèle,  on utilise un transistor 2N3904 (T4) couplé à une résistance (R7) d’un mega ohms. Le signal récupéré au collecteur de T4 est connecté à la borne  4 de l’arduino

Pour le signal de niveau de jauge de carburant seule l’amplitude nous intéresse. On isole donc le signal par la diode  D3 et l’on vient charger le condensateur de faible capacité C1 ce qui permet de filtrer le signal et d’obtenir une tension stable proportionnelle au niveau de la jauge. Le signale est ensuite enregistré via la borne analogique A6 de l’arduino

Les boutons des commodo (Menu +, Menu – et reset) seront enregistrés via les bornes 8,9 et A1 qui sont mises au potentiel haut de 3.3V par défaut via les résistances R11, R12, R13 

Implantation des composant

J’utilise une platine de prototypage pour réaliser le support de l’arduino ce qui évite les typons et les bains acides

![alt text]https://github.com/AmesisProject/FIS_color_Audi_A3_8L_Golf_IV/blob/main/Photo/6.png

Cotés cuivres les pistes sont interrompues grâce à un petit coup de perceuses (cf ronds rouge sur la vue côté cuivre)

![alt text]https://github.com/AmesisProject/FIS_color_Audi_A3_8L_Golf_IV/blob/main/Photo/7.png
![alt text]https://github.com/AmesisProject/FIS_color_Audi_A3_8L_Golf_IV/blob/main/Photo/8.png
![alt text]https://github.com/AmesisProject/FIS_color_Audi_A3_8L_Golf_IV/blob/main/Photo/9.png

Combiné d'instruments - jusqu'au millésime 1998/1999

Connexion à fiche multiple, 32 raccords, bleue
 
1 - -> Borne 15 
2 - Clignotant droit 
3 - Tachymètre, sortie 1 
4 - Non affecté 
5 - Transmetteur de niveau de carburant 
6 - Sac gonflable 
7 - Borne 31 (masse du détecteur) 
8 - Température du liquide de refroidissement 
9 - Borne 31 (masse de charge) 
10 - Contacteur de pression d'huile 
11 - Signal de régime 
12 - Borne 61 
13 - Témoin de préchauffage ou CAT 
14 - Correcteur d'assiette (non affecté à l'heure actuelle) 
15 - Borne 58d 
16 - Clignotants de remorque 
17 - feux de route 
18 - Clignotant gauche 
19 - ABS 
20 - Borne 58b 
21 - Contact de porte côté conducteur 
22 - Manque de liquide de refroidissement 
23 - Borne 30 
24 - Borne 31 (masse de charge) 
25 - Câble K 
26 - Feu de stationnement droit 
27 - Feu de stationnement gauche 
28 - Entrée du tachymètre 
29 - Niveau/pression du liquide de frein 
30 - Contact S 
31 - Serrure de ceinture 
32 - ASR 
 


Connexion à fiche multiple, 32 raccords, verte
 
1 - → Non affecté 
2 - Transpondeur 1 
3 - Non affecté 
4 - Non affecté 
5 - Câble W 
6 - Hayon (uniquement version Midline) 
7 - Garniture de frein (non affecté à l'heure actuelle) 
8 - Entrée destinée à l'activation du vibreur externe (non affecté à l'heure actuelle) 
9 - Entrée destinée à l'activation du gong externe (non affecté à l'heure actuelle) 
10 - Signal de sortie d'alerte du niveau de réservoir pour appareil de commande (non affecté à l'heure actuelle) 
11 - Edition du temps d'arrêt 
12 - Coupure climatiseur 
13 - → Frein de stationnement 
14 - Commande d'accélérateur électrique 
15 - Feu de position 
16 - Non affecté 
17 - Transpondeur 2 
18 - Non affecté 
19 - Non affecté 
20 - Non affecté 
21 - Non affecté 
22 - Non affecté 
23 - Non affecté 
24 - Non affecté 
25 - Non affecté 
26 - Non affecté 
27 - Non affecté 
28 - Non affecté 
29 - Borne 58d (variateur d'intensité externe) 
30 - Tachymètre, sortie 2 
31 - Non affecté 
32 - Non affecté 
 
 
 
Connexion à fiche multiple, 20 raccords, rouge
 
1 - → Signal de consommation 
2 - Non affecté 
3 - Non affecté 
4 - Non affecté 
5 - Température extérieure 
6 - Indication du levier sélecteur 
7 - Non affecté 
8 - Non affecté 
9 - Eau de lavage 
10 - Pression hydraulique 
11 - Ordinateur de bord, Reset 
12 - Signal Clock pour affichage de fréquences radio 
13 - Signal Data pour affichage de fréquences radio 
14 - Ordinateur de bord côté gauche 
15 - Signal Enable pour affichage de fréquences radio 
16 - Feu de recul/feu de croisement (contrôle des ampoules) 
17 - Relais de préchauffage (Pendant la période de préchauffage, le contrôle de sous-tension est inhibé dans le combiné d'instruments uniquement en cas de moteur diesel) 
18 - Ordinateur de bord côté droit 
19 - Feu de stop 
20 - Non affecté

Combiné d'instruments à partir du millésime 2000

Connexion à fiche multiple, 32 raccords, bleue
 
1 - → Borne 15 
2 - Usure des garnitures de frein 
3 - Sortie 1 tachymètre 
4 - Non affecté 
5 - Transmetteur de niveau de carburant 
6 - Alerte du niveau de réservoir OBD 2 
7 - Borne 31 (masse du détecteur) 
8 - Température du liquide de refroidissement 
9 - Borne 31 (masse de charge) 
10 - Pression d'huile 2 (élevée) 
11 - Signal de régime 
12 - Coupure climatiseur 
13 - Commande d'accélérateur électrique/contrôle préchauffage 
14 - Correcteur d'assiette 
15 - Borne 58d 
16 - Clignotants de remorque 
17 - → Feux de route 
18 - Clignotant gauche 
19 - Non affecté 
20 - Borne 58s 
21 - Contact de porte côté conducteur 
22 - Manque de liquide de refroidissement 
23 - Borne 30 
24 - Borne 31 (masse de charge) 
25 - Signal de consommation 
26 - Feu de stationnement droit 
27 - Feu de stationnement gauche 
28 - Entrée du tachymètre 
29 - Frein 
30 - Contact S 
31 - Sortie 2 tachymètre 
32 - ESP/ASR 
 
Connexion à fiche multiple, 32 raccords, verte
 
1 - → Contact de porte (toutes les portes) 
2 - Transpondeur 1 
3 - Non affecté 
4 - Non affecté 
5 - Câble W 
6 - Hayon 
7 - Clignotant droit 
8 - Vibreur externe 
9 - Gong externe 
10 - Sac gonflable 
11 - Edition du temps d'arrêt 
12 - Borne 61 
13 - Frein de stationnement/BRAKE 
14 - CONTRÔLE 
15 - Niveau d'huile/température d'huile 
16 - Non affecté 
17 - → Transpondeur 2 
18 - CAN high speed (propulsion) (high +) 
19 - CAN high speed (propulsion) (low -) 
20 - CAN high speed (propulsion) (screen) 
21 - ABS 
22 - CAN low speed (Confort) (high +) 
23 - CAN low speed (Confort) (low -) 
24 - CAN low speed (Confort) (screen) 
25 - Capot-moteur 
26 - Bouchon de réservoir 
27 - Boucle de ceinture 
28 - Câble K 
29 - Entrée température extérieure 
30 - Non affecté 
31 - Affichage de la gamme de vitesse 
32 - Non affecté 
 
 
Connexion à fiche multiple, 32 raccords, grise
 
1 - → Commande pour déroulement menu (Menue) 
2 - Commande pour déroulement menu (out A) 
3 - Commande pour déroulement menu (out B) 
4 - Commande pour déroulement menu (Enter) 
5 - Ecran CAN high speed (high +) 
6 - Ecran CAN high speed (low -) 
7 - Ecran CAN high speed (screen) 
8 - Contact de porte côté passager avant 
9 - Contact de porte arrière droite 
10 - Contact de porte arrière gauche 
11 - Enable 
12 - Clock 
13 - Data 
14 - Feu de stop 
15 - Niveau de lave-glace 
16 - Feu de recul/feu de croisement
17 - → Ordinateur de bord côté gauche 
18 - Ordinateur de bord côté droit 
19 - Ordinateur de bord, Reset 
20 - Non affecté 
21 - Non affecté 
22 - Non affecté 
23 - Embranchement touche 1, système de navigation 
24 - Embranchement touche 2, système de navigation 
25 - Embranchement touche télématique 
26 - Non affecté 
27 - Non affecté 
28 - Non affecté 
29 - Non affecté 
30 - Non affecté 
31 - Non affecté 
32 - Non affecté 
 
 
 
Connexion à fiche multiple à 4 raccords (noire) pour montre radiopilotée
 
1 - → Signal montre radiopilotée (données) 
2 - Montre radiopilotée 5 V 
3 - Non affecté 
4 - Masse montre radiopilotée 
