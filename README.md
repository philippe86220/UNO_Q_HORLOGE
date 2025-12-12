# UNO_Q_HORLOGE

Hypothèses explicites (celles de votre horloge) :
- `digit = 0`
- chiffre des dizaines d’heures
- donc `xOffset = 0`
- `yOffset = 1`
- `DIGITS[0] = { 0b111, 0b101, 0b101, 0b101, 0b111 }`

  ---

 ### Rappel du code testé

```cpp
  void drawDigitDirect(int digit, int xOffset, uint32_t frame[4]) {
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
### Ligne par ligne (valeurs exactes transmises)  
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

### Liste finale exhaustive des appels à `setPixelBit`
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

### Ce que cela représente visuellement
Un **0 en 3×5**, parfaitement centré verticalement entre `y = 1` et `y = 5`, avec :
- une barre pleine en haut,
- deux colonnes verticales,
- une barre pleine en bas.

Et surtout :  
👉 **chaque appel correspond exactement à une LED allumée**, sans aucune ambiguïté.
