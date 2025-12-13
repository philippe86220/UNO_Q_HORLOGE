# Horloge LED 13×8 sur Arduino UNO Q 
### 1. Objectif du projet  
Ce projet affiche l’heure (HH:MM) sur la matrice LED 13×8 de l’Arduino UNO Q, en utilisant directement  
l’API bas niveau :
```cpp
matrixBegin();
matrixWrite(uint32_t frame[4]);
```
les chiffres sont décrits sous forme de **glyphes binaires 3×5**,   
puis directement traduits en bits dans un buffer de **4 mots de 32 bits**.

---

### 2. Principe général du rendu
Le mécanisme est volontairement simple :    
1. L’heure est reçue depuis le cœur Linux via RPC (`updateTime`)
2. Les chiffres sont découpés en dizaines/unités
3. Chaque chiffre est dessiné à une position (`xOffset`, `yOffset`)
4. Chaque LED allumée est convertie en un bit dans `frame[4]`
5. Le buffer est transmis à la matrice via `matrixWrite()`    

**Il n’existe qu’une seule représentation finale** : le buffer `uint32_t frame[4]`.

---

### 3. Représentation des chiffres (glyphes 3×5)
Les chiffres sont définis dans le tableau suivant :  
```cpp
const uint8_t DIGITS[10][5] = {
    // 0
    { 0b111, 0b101, 0b101, 0b101, 0b111 }, 
    // 1
    { 0b001, 0b001, 0b001, 0b001, 0b001 },
    // 2
    { 0b111, 0b001, 0b111, 0b100, 0b111 },
    // 3
    { 0b111, 0b001, 0b111, 0b001, 0b111 },
    // 4
    { 0b101, 0b101, 0b111, 0b001, 0b001 },
    // 5
    { 0b111, 0b100, 0b111, 0b001, 0b111 },
    // 6
    { 0b111, 0b100, 0b111, 0b101, 0b111 },
    // 7
    { 0b111, 0b001, 0b001, 0b001, 0b001 },
    // 8
    { 0b111, 0b101, 0b111, 0b101, 0b111 },
    // 9
    { 0b111, 0b101, 0b111, 0b001, 0b111 }
};
```
Chaque chiffre est composé de 5 lignes, chacune codée sur **3 bits** :
1. bit 2 → colonne gauche
2. bit 1 → colonne centrale
3. bit 0 → colonne droite  


Exemple pour le chiffre 0 :
```cpp
DIGITS[0] = {
  0b111,
  0b101,
  0b101,
  0b101,
  0b111
};
```
Cela correspond visuellement à :
```
###
# #
# #
# #
###
```
---

### 4. Exemple détaillé : affichage du chiffre 0 (dizaine d’heures)
```cpp
// Hours
   if (hour >= 10) drawDigit(hTens, 0, frame);
```  
> Remarque : 
>   
> L’exemple détaillé du chiffre **0** (dizaine d’heures), présenté dans ce README, n’est pas codé spécifiquement dans le programme.
> **J'ai exclu le `0` des dizaine (je l'avais mis au début puis je l'ai enlevé) d'heures mais chacun pourra le remettre si bon lui semble**.  
> Il s’agit d’un **exemple pédagogique volontairement isolé**, choisi parce qu’il permet de dérouler simplement et complètement le cheminement :
>  
> glyphe 3×5 → coordonnées (x, y) → index linéaire → bit dans le buffer 32 bits.  
>  
> **Ce raisonnement est strictement identique pour tous les autres chiffres**.


Hypothèses explicites de l'exemple :
- `digit = 0`
- chiffre des **dizaines d’heures**
- donc `xOffset = 0`
- `yOffset = 1`
- `DIGITS[0] = { 0b111, 0b101, 0b101, 0b101, 0b111 }`

  ---

### 4.1 Parcours du glyphe
La fonction suivante est utilisée :  

```cpp
  void drawDigit(int digit, int xOffset, uint32_t frame[4]) {
    if (digit < 0 || digit > 9) return;

    const int yOffset = 1; // digits drawn from rows 1..5

    for (int row = 0; row < 5; row++) {
        uint8_t pattern = DIGITS[digit][row];

        for (int col = 0; col < 3; col++) {
            if (pattern & (1u << (2 - col))) {
                setPixelBit(frame, xOffset + col, yOffset + row);
            }
        }
    }
}
```
Pour chaque ligne (`row = 0 .. 4`) :  
- on lit un motif binaire (`pattern`)
- on teste chaque colonne (`col = 0 .. 2`)
- si le bit correspondant vaut 1, une LED est allumée  

Le test clé est :

```cpp
if (pattern & (1u << (2 - col)))
```
Cette écriture permet :  
- de lire les bits du glyphe de **gauche à droite**
- tout en parcourant les colonnes `col = 0 → 2`

---




### 4.2 Parcours du glyphe Ligne par ligne 
**row = 0**  
`pattern = 0b111` 


| col | bit testé | condition | x | y | appel                      |
| --: | --------- | --------- | - | - | -------------------------- |
|   0 | `1<<2`    | vrai      | 0 | 1 | `setPixelBit(frame, 0, 1)` |
|   1 | `1<<1`    | vrai      | 1 | 1 | `setPixelBit(frame, 1, 1)` |
|   2 | `1<<0`    | vrai      | 2 | 1 | `setPixelBit(frame, 2, 1)` |


---
**row = 1**  
`pattern = 0b101`

| col | bit testé | condition | x | y | appel                      |
| --: | --------- | --------- | - | - | -------------------------- |
|   0 | `1<<2`    | vrai      | 0 | 2 | `setPixelBit(frame, 0, 2)` |
|   1 | `1<<1`    | faux      | — | — | —                          |
|   2 | `1<<0`    | vrai      | 2 | 2 | `setPixelBit(frame, 2, 2)` |

---
**row = 2**  
`pattern = 0b101`

| col | bit testé | condition | x | y | appel                      |
| --: | --------- | --------- | - | - | -------------------------- |
|   0 | `1<<2`    | vrai      | 0 | 3 | `setPixelBit(frame, 0, 3)` |
|   1 | `1<<1`    | faux      | — | — | —                          |
|   2 | `1<<0`    | vrai      | 2 | 3 | `setPixelBit(frame, 2, 3)` |

---

**row = 3**  
`pattern = 0b101`

| col | bit testé | condition | x | y | appel                      |
| --: | --------- | --------- | - | - | -------------------------- |
|   0 | `1<<2`    | vrai      | 0 | 4 | `setPixelBit(frame, 0, 4)` |
|   1 | `1<<1`    | faux      | — | — | —                          |
|   2 | `1<<0`    | vrai      | 2 | 4 | `setPixelBit(frame, 2, 4)` |

---

**row = 4**  
pattern = 0b111

| col | bit testé | condition | x | y | appel                      |
| --: | --------- | --------- | - | - | -------------------------- |
|   0 | `1<<2`    | vrai      | 0 | 5 | `setPixelBit(frame, 0, 5)` |
|   1 | `1<<1`    | vrai      | 1 | 5 | `setPixelBit(frame, 1, 5)` |
|   2 | `1<<0`    | vrai      | 2 | 5 | `setPixelBit(frame, 2, 5)` |

---

### 5. Passage des coordonnées (x, y) au buffer binaire
```cpp
setPixelBit(frame, 0, 1);
setPixelBit(frame, 1, 1);
setPixelBit(frame, 2, 1);

setPixelBit(frame, 0, 2);
setPixelBit(frame, 2, 2);

setPixelBit(frame, 0, 3);
setPixelBit(frame, 2, 3);

setPixelBit(frame, 0, 4);
setPixelBit(frame, 2, 4);

setPixelBit(frame, 0, 5);
setPixelBit(frame, 1, 5);
setPixelBit(frame, 2, 5);
```


Donc lorsqu’une LED doit être allumée, on appelle :
```cpp
setPixelBit(frame, x, y);
```
À l’intérieur de cette fonction :
```cpp
int index = y * MATRIX_WIDTH + x;  // 0..103
int word  = index / 32;            // 0..3
int bit   = index % 32;            // 0..31

frame[word] |= (1u << bit);
```
Ainsi :
- la matrice 13×8 est linéarisée en indices de **0 à 103**
- chaque LED correspond à **un bit unique** du buffer
- le buffer final contient **128 bits**, dont seuls 104 sont utilisés



### Ce que cela représente visuellement
Un **0 en 3×5**, parfaitement centré verticalement entre `y = 1` et `y = 5`, avec :
- une barre pleine en haut,
- deux colonnes verticales,
- une barre pleine en bas.

Et surtout :  
👉 **chaque appel correspond exactement à une LED allumée**, sans aucune ambiguïté.

---

### 6. Exemple concret de LEDs allumées (chiffre 0)
Pour le chiffre `0` à `xOffset = 0`, les appels suivants sont générés :
```
(0,1) (1,1) (2,1)
(0,2)       (2,2)
(0,3)       (2,3)
(0,4)       (2,4)
(0,5) (1,5) (2,5)
```
Ces coordonnées correspondent exactement au glyphe 3×5 positionné dans la matrice.  

Un tableau récapitulatif (fourni en pièce jointe) permet de visualiser :
- les coordonnées (x, y)
- l’index linéaire y*13 + x
- le mot 32 bits et le bit correspondant

  ---

 ### 7. Gestion du séparateur « : »

 Le séparateur des heures et minutes est affiché à la colonne `x = 6` :
```cpp
setPixelBit(frame, 6, 2);
setPixelBit(frame, 6, 4);
```

Il clignote une seconde sur deux en fonction de `second % 2`.

---

### 8. Conclusion
Ce projet montre comment :  
- représenter des chiffres sous forme de glyphes binaires,
- les positionner précisément sur une matrice LED,
- et convertir ces positions en bits dans un buffer bas niveau.

L’objectif n’est pas la performance maximale, mais la compréhension complète du cheminement, du chiffre abstrait jusqu’au bit allumé sur la matrice.

## Credits

- Projet, conception et implémentation : philippe86220
- Accompagnement pédagogique, explications détaillées et aide à la compréhension du code : ChatGPT (OpenAI)

