# Révision 2 : Values, references et pointers

[← Révision 1](./01_Functions_Files_Compiler.md) · [Sommaire](./README.md) · [Révision 3 →](./03_Const_Auto_Templates.md)

## Objectifs

- Distinguer une copy, une reference et un pointer.
- Prévoir si une function modifie la variable passée en argument.
- Distinguer l’address d'une variable et sa value.
- Utiliser un pointer valide et gérer le cas `nullptr`.
- Comprendre que les variables ont un scope et une lifetime.

**Prérequis :** Avoir revu les functions et les return values dans la [révision 1](01_Functions_Files_Compiler.md).

<details>
<summary>📑 Sommaire de la fiche</summary>

- [Vocabulaire](#vocabulaire)
- [1. Une value, une copy, une reference, un pointer](#1-une-value-une-copy-une-reference-un-pointer)
- [2. Parameter passing : value, reference et pointer](#2-parameter-passing--value-reference-et-pointer)
- [3. Return value ou modification d’une variable](#3-return-value-ou-modification-dune-variable)
- [4. Changer de cible : pointer et reference](#4-changer-de-cible--pointer-et-reference)
- [5. `nullptr` et validité](#5-nullptr-et-validité)
- [6. Scope et lifetime](#6-scope-et-lifetime)
- [7. Lire sans modifier : `const`](#7-lire-sans-modifier--const)
- [Exercices](#exercices)
- [À retenir](#à-retenir)
- [Avant de terminer](#avant-de-terminer)
- [Références](#références)

</details>

## Vocabulaire

| Terme technique | Sens en français |
|---|---|
| value / copy | Valeur / copie |
| reference / pointer | Référence / pointeur |
| address / dereference | Adresse / accès à la valeur désignée par un pointeur |
| parameter / argument | Paramètre de la fonction / expression fournie lors de l’appel |
| return value | Valeur renvoyée par la fonction |
| assignment | Affectation |
| scope / lifetime | Portée d’un nom / durée de vie d’un objet |
| dangling pointer | Pointeur vers un objet dont la durée de vie est terminée |
| undefined behavior (UB) | Comportement indéfini |

## 1. Une value, une copy, une reference, un pointer

```cpp
int life = 99;
int lifeCopy = life;
int& lifeRef = life;
int* lifePtr = &life;
```

| Expression | Signification |
|---|---|
| `life` | Un integer contenant `99`. |
| `lifeCopy` | Un autre integer, initialisé avec une copy de la value de `life`. |
| `lifeRef` | Une reference vers `life` : un autre nom pour le même integer. |
| `lifePtr` | Un pointer contenant l’address de `life`. |
| `&life` | L'address de `life`. |
| `*lifePtr` | L'integer désigné par le pointer, ici `life`. |

```cpp
lifeCopy = 10;  // Seule la copy change.
lifeRef = 80;   // life vaut maintenant 80.
*lifePtr = 70;  // life vaut maintenant 70.
```

Une reference n'est pas une copy de l'integer. Un pointer n'est pas l'integer vers lequel il pointe.

### Attention au contexte de `&` et `*`

| Code | Rôle du symbole |
|---|---|
| `int& lifeRef = life;` | `&` fait partie du type : reference vers un integer. |
| `int* lifePtr = &life;` | `*` fait partie du type : pointer vers un integer. `&life` obtient une address. |
| `*lifePtr = 50;` | `*` effectue un dereference du pointer pour accéder à l'integer désigné. |

📖 **Pour approfondir :** [References, Hacking C++ (EN)](https://hackingcpp.com/cpp/lang/references) montre les effets d’une reference sur l’objet d’origine. [Pointers, Hacking C++ (EN)](https://hackingcpp.com/cpp/lang/pointers) complète avec les addresses, le dereference et le changement de cible.

## 2. Parameter passing : value, reference et pointer

```cpp
#include <iostream>

void LoseLifeValue(int lifeValue)
{
    lifeValue--;
}

void LoseLifeReference(int& lifeReference)
{
    lifeReference--;
}

void LoseLifePointer(int* lifePtr)
{
    if (lifePtr == nullptr)
    {
        return;
    }

    (*lifePtr)--;
}

int main()
{
    int life = 99;
    int& lifeRef = life;
    int* lifePtr = &life;

    LoseLifeValue(life);
    std::cout << life << '\n'; // 99

    LoseLifeReference(lifeRef);
    std::cout << life << '\n'; // 98

    LoseLifePointer(lifePtr);
    std::cout << life << '\n'; // 97

    return 0;
}
```

| Parameter | Ce que reçoit la function | Effet du decrement |
|---|---|---|
| `int lifeValue` | Une copy de l'integer. | Seule la copy locale diminue. |
| `int& lifeReference` | Une reference vers l'integer d'origine. | L'integer d'origine diminue. |
| `int* lifePtr` | Une copy de l’address. | `(*lifePtr)--` diminue l'integer désigné. |

> [!IMPORTANT]
> **Le pointer est lui aussi passé par value.**
> L’address stockée est copiée, mais les deux pointers désignent le même integer. Modifier cet integer reste donc visible après l’appel.

`LoseLifeReference(life)` fonctionne aussi : il n'est pas nécessaire de créer `lifeRef` avant l'appel. Le type du parameter détermine le passage par reference (pass by reference).

### Pourquoi les parenthèses dans `(*lifePtr)--` ?

- `(*lifePtr)--` décrémente l'integer désigné.
- `lifePtr--` déplace le pointer, pas la value.
- `*lifePtr--` se lit `*(lifePtr--)` : il décrémente aussi le pointer.

Pour retirer une vie, on veut modifier **la value désignée**. Ne testez pas ces déplacements de pointer sur un integer isolé : ils peuvent produire un undefined behavior (UB).

## 3. Return value ou modification d’une variable

```cpp
int GetDamagedLife(int life, int damage)
{
    int result = life - damage;

    if (result < 0)
    {
        result = 0;
    }

    return result;
}
```

On suppose ici une vie entre `0` et `100` et des dégâts entre `0` et `100`.

```cpp
int life = 10;

GetDamagedLife(life, 3);         // Résultat ignoré : life reste à 10.
life = GetDamagedLife(life, 3);  // Assignment du résultat : life vaut 7.
```

`return` renvoie un résultat au caller. Il ne remplace pas automatiquement la variable utilisée comme argument. Une function `void` peut modifier une variable par reference ou par pointer sans renvoyer de return value.

## 4. Changer de cible : pointer et reference

```cpp
int aliceLife = 10;
int bobLife = 20;

int* target = &aliceLife;
*target = 5;          // Alice a 5 points de vie.
target = &bobLife;   // Le pointer désigne désormais Bob.
*target = 15;        // Bob a 15 points de vie.
```

Une reference doit être initialisée et ne peut pas changer d’objet désigné ensuite (rebinding interdit) :

```cpp
int& selectedLife = aliceLife;
selectedLife = bobLife;
```

La dernière ligne **copie la value de Bob dans la vie d'Alice**. `selectedLife` fait toujours reference à `aliceLife`.

### Réaffecter un pointer parameter

```cpp
void ForgetTarget(int* target)
{
    target = nullptr;
}
```

Cette function change seulement sa copy locale du pointer. Après `ForgetTarget(lifePtr)`, le pointer du caller n'a pas changé.

## 5. `nullptr` et validité

```cpp
int* target = nullptr;
```

`nullptr` représente l'absence de cible. Il ne faut pas effectuer un dereference de ce pointer.

```cpp
if (target != nullptr)
{
    *target = 50;
}
```

> [!WARNING]
> **Non nul ne signifie pas forcément valide.**
> Un pointer peut encore contenir l’address d’une variable qui n’existe plus. Le test contre `nullptr` ne détecte pas ce problème.

## 6. Scope et lifetime

Le scope indique où un nom est utilisable. La lifetime indique pendant quelle période l'object existe.

```cpp
int* target = nullptr;

{
    int temporaryLife = 10;
    target = &temporaryLife;
    *target = 8; // Valide : temporaryLife existe.
}

// temporaryLife n'existe plus.
// target est un dangling pointer : ne pas utiliser *target.
target = nullptr;
```

Pour la même raison, ne renvoyez pas une reference ou un pointer vers une variable locale ordinaire : elle est détruite à la sortie de la function. Renvoyer sa **value** est possible.

## 7. Lire sans modifier : `const`

```cpp
void DisplayLife(const int& life)
{
    std::cout << life << '\n';
    // life--; // Erreur : modification interdite par cette reference.
}
```

`const int&` permet de lire l'object sans le copier et sans le modifier par cette reference. Pour un simple `int`, un passage par value reste généralement le choix le plus simple. L'intérêt de la const reference devient plus visible avec des objects plus volumineux.

Avec un pointer :

| Type | Modifier l'integer par ce pointer ? | Changer la cible du pointer ? |
|---|---|---|
| `int*` | Oui | Oui |
| `const int*` | Non | Oui |
| `int* const` | Oui | Non |

Ces droits concernent cet accès à l'object. Une const reference ne rend pas nécessairement l'object d'origine constant.

🖼️ **Comparer les écritures de `const` :** [East const vs. const West, Hacking C++ (EN)](https://hackingcpp.com/cpp/design/east_vs_west_const) met côte à côte `const int*`, `int const*` et `int* const` pour distinguer ce qui est constant.

---

## Exercices

Pour chaque exercice :

1. Écrire une prédiction avant d'exécuter.
2. Tester le programme.
3. Expliquer tout écart entre prédiction et résultat.

### Exercice 1 : suivre les values • 10 min

Reprendre les trois functions `LoseLife...` de la fiche. Indiquer la value de `life` après chaque statement :

```cpp
int life = 10;
int copy = life;
int& lifeRef = life;
int* lifePtr = &life;

LoseLifeValue(life);
LoseLifeReference(life);
LoseLifePointer(lifePtr);
copy = 50;
lifeRef = 20;
*lifePtr = 30;
```

Questions :

- Quelles lignes modifient `life` ?
- Quelles sont les values finales de `life` et `copy` ?
- Que contiennent respectivement `lifePtr` et `*lifePtr` ?

### Exercice 2 : soigner un joueur • 15 à 20 min

Écrire :

```cpp
int HealValue(int life, int amount);
void HealReference(int& life, int amount);
```

Contraintes :

- `HealValue` renvoie la nouvelle vie sans modifier la variable d'origine.
- `HealReference` modifie directement cette variable.
- La vie ne dépasse jamais `100`.
- Les entrées `life` et `amount` sont comprises entre `0` et `100`.

Tester les trois statements suivants **séparément, en repartant chaque fois de `life = 90`** :

```cpp
HealValue(life, 20);
life = HealValue(life, 20);
HealReference(life, 20);
```

Ajouter les cas : soin nul, vie nulle, vie déjà à `100`. Expliquer pourquoi le premier appel ne modifie pas `life`.

### Exercice 3 : changer de cible • 15 min

Prédire les vies d'Alice et de Bob après chaque ligne :

```cpp
int aliceLife = 10;
int bobLife = 20;

int* target = &aliceLife;
*target -= 3;
target = &bobLife;
*target -= 5;

int& selectedLife = aliceLife;
selectedLife = bobLife;
selectedLife = 1;
```

Expliquer la différence entre `target = &bobLife;` et `*target = bobLife;`.

### Exercice 4 : une cible peut être absente • 15 à 20 min

Écrire :

```cpp
void Damage(int* life, int amount);
```

Contraintes :

- Si le pointer vaut `nullptr`, ne rien faire.
- Sinon, retirer `amount` points de vie sans descendre sous `0`.
- La vie initiale et les dégâts sont compris entre `0` et `100`.

Tests attendus :

```cpp
int life = 10;

Damage(&life, 3);    // 7
Damage(&life, 20);   // 0
Damage(nullptr, 5); // Aucun accès invalide.
```

Ajouter un test avec des dégâts nuls. Expliquer pourquoi on teste le pointer avant d'utiliser `*life`.

### Exercice 5 : permuter deux vies • 20 min

Écrire sans `std::swap` :

```cpp
void SwapReference(int& first, int& second);
void SwapPointer(int* first, int* second);
```

Contraintes :

- Échanger les values des deux integers d'origine.
- Dans la version pointer, ne rien faire si l'un des pointers vaut `nullptr`.
- Tester deux values différentes, deux values égales et deux accès au même integer.

Question : échanger seulement les addresses contenues dans les parameters locaux suffit-il à échanger les vies ?

### Exercice 6 : trouver les problèmes • 15 min

Les extraits sont indépendants. Les analyser sans les exécuter, expliquer le problème et proposer une correction.

**A.**

```cpp
int* target = nullptr;
*target = 10;
```

**B.**

```cpp
int& GetLife()
{
    int life = 100;
    return life;
}
```

**C.** On souhaite remettre le pointer du caller à `nullptr` :

```cpp
void ClearTarget(int* target)
{
    target = nullptr;
}
```

Pour C, expliquer d'abord pourquoi le résultat attendu n'est pas obtenu. En bonus, proposer un parameter par reference vers un pointer : `int*&`.

### Bonus : organiser les files

Répartir les functions dans `Life.h` et `Life.cpp`. Conserver les appels dans `main.cpp`. Mettre une include guard ou un `#pragma once` dans le header et vérifier que les declarations correspondent aux definitions.

---

## À retenir

> [!TIP]
> **Les repères à garder**
>
> - Une value passée par copy est indépendante de la variable d’origine.
> - Une reference donne accès au même objet et ne peut pas changer de cible.
> - Un pointer contient une address ; `*pointer` accède à l’objet désigné.
> - Un pointer passé par value est lui aussi copié.
> - `nullptr` signale l’absence de cible ; un pointer non nul peut quand même être invalide.
> - Le scope d’un nom et la lifetime d’un objet sont deux notions différentes.

## Avant de terminer

- [ ] Je peux expliquer la différence entre `life`, `&life`, `lifePtr` et `*lifePtr`.
- [ ] Je peux prévoir si une function modifie la variable d'origine.
- [ ] Je distingue modifier la cible et modifier la value désignée.
- [ ] Je sais qu'une reference ne peut pas changer d’objet désigné après son initialization (rebinding interdit).
- [ ] Je ne fais pas de dereference sur un null pointer ou un dangling pointer.
- [ ] Je sais récupérer une return value et tester les cas limites.

---

## Références

- [References : donner un autre nom à un objet](https://learn.microsoft.com/fr-fr/cpp/cpp/references-cpp?view=msvc-170) : L’initialization d’une reference et les différences avec un pointer, notamment l’impossibilité de changer de cible.
- [Pointers : stocker une address et accéder à un objet](https://learn.microsoft.com/fr-fr/cpp/cpp/pointers-cpp?view=msvc-170) : Les bases des pointers, leur declaration et leur utilisation pour accéder à des données.
- [Scope : où un nom est-il utilisable ?](https://learn.microsoft.com/en-us/cpp/cpp/scope-visual-cpp?view=msvc-170) : Les différentes portées et les règles de visibilité des noms dans le programme.

---

[← Révision 1](./01_Functions_Files_Compiler.md) · [Sommaire](./README.md) · [Révision 3 →](./03_Const_Auto_Templates.md)
