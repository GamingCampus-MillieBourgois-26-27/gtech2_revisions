# Révision 3 : const, auto et templates

[← Révision 2](./02_Values_References_Pointers.md) · [Sommaire](./README.md) · [Révision 4 →](./04_Design_Method.md)

## Objectifs

- Lire les différents usages de `const` dans `Player`.
- Prévoir le type déduit par `auto`, `auto&` et `const auto&`.
- Ajouter des default arguments aux functions et aux templates.
- Comprendre un class template en remplaçant son type parameter.
- Exprimer des intentions dans le code et faire vérifier certaines contraintes par le compiler.

**Prérequis :** Avoir revu les copies, references, pointers et lifetimes dans la [révision 2](02_Values_References_Pointers.md).

<details>
<summary>📑 Sommaire de la fiche</summary>

- [1. Donner du sens au code](#1-donner-du-sens-au-code)
- [2. Notre exemple : Player](#2-notre-exemple--player)
- [3. Comprendre les trois usages de `const`](#3-comprendre-les-trois-usages-de-const)
- [4. `auto` déduit un type précis](#4-auto-déduit-un-type-précis)
- [5. Default arguments d’une function](#5-default-arguments-dune-function)
- [6. Un class template : `Vector<T>`](#6-un-class-template--vectort)
- [7. Default template arguments](#7-default-template-arguments)
- [8. Reconnaître les templates de la standard library](#8-reconnaître-les-templates-de-la-standard-library)
- [Exercices](#exercices)
- [À retenir](#à-retenir)
- [Avant de terminer](#avant-de-terminer)
- [Références](#références)

</details>

## 1. Donner du sens au code

Les types permettent d’exprimer des intentions et de détecter certaines erreurs à la compilation.

- `const` limite les modifications autorisées par un accès.
- `auto` demande au compiler de déduire un type.
- Un template décrit une famille de types ou de functions.
- Un default argument permet d’omettre un argument lorsqu’un choix par défaut convient.

Ces outils ne promettent pas une amélioration des performances. Leur intérêt est aussi de rendre le code plus expressif et de faire vérifier certaines contraintes par le compiler.

Un programme qui compile peut toujours contenir des erreurs de logique.

## 2. Notre exemple : Player

### `Player.h`

```cpp
#pragma once

#include <string>

class Player
{
public:
    const std::string& GetName() const
    {
        return name;
    }

    void SetName(const std::string& newName)
    {
        name = newName;
    }

private:
    int life = 99;
    std::string name = "Default Name";
};
```

### `main.cpp`

```cpp
#include <iostream>
#include <string>

#include "Player.h"

int main()
{
    Player player;

    std::string newPlayerName = "Millie";
    player.SetName(newPlayerName);

    auto& playerName = player.GetName();

    std::cout << playerName << '\n';

    return 0;
}
```

## 3. Comprendre les trois usages de `const`

### Le parameter : `const std::string&`

```cpp
void SetName(const std::string& newName)
{
    name = newName;
}
```

- `&` permet de recevoir une reference vers la string existante.
- `const` interdit de modifier cette string par `newName`.
- Le body peut modifier le member `name`, qui est une autre variable.

Passer une string existante par reference évite une copy pour construire le parameter. En revanche, `name = newName` copie ensuite son contenu dans le member.

> [!IMPORTANT]
> **Distinguer les deux rôles**
> La reference évite ici la copy du parameter. `const` interdit de modifier la string par cet accès.

### Le return type : `const std::string&`

```cpp
const std::string& GetName() const
{
    return name;
}
```

Le getter renvoie une reference vers le member existant.

- Le return ne crée pas une nouvelle string.
- Le caller ne peut pas modifier `name` par cette const reference.
- La reference dépend de la lifetime du `Player`.

La reference n’est pas une photographie du texte : si le même `Player` change de nom, elle permet de lire le nouveau nom.

### Le `const` après les parenthèses

Dans `GetName() const`, ce dernier `const` concerne la member function.

Elle ne peut pas modifier les data members ordinaires à travers `this`. Elle peut aussi être appelée sur une instance const.

```cpp
const Player player;

player.GetName();         // Autorisé.
// player.SetName("Bob"); // Erreur : setter non const.
```

Les deux `const` de `GetName()` n’ont donc pas le même rôle.

📖 **Pour approfondir :** [Basic Custom Types / Classes, Hacking C++ (EN)](https://hackingcpp.com/cpp/lang/custom_type_basics), section **const Member Functions**, illustre les appels autorisés sur un objet const et les modifications refusées dans une member function const.

## 4. `auto` déduit un type précis

```cpp
auto life = 99;      // int
auto speed = 3.5f;   // float
auto distance = 3.5; // double
```

Le type est déterminé à la compilation. Il ne change pas ensuite.

`auto` ne rend pas automatiquement une variable const et ne signifie pas « sans copy ».

### `auto`, `auto&` et `const auto&`

Les declarations suivantes sont indépendantes :

| Declaration | Type réel | Conséquence |
|---|---|---|
| `auto name = player.GetName();` | `std::string` | Copy modifiable. |
| `const auto name = player.GetName();` | `const std::string` | Copy non modifiable. |
| `auto& name = player.GetName();` | `const std::string&` | Reference non modifiable. |
| `const auto& name = player.GetName();` | `const std::string&` | Reference non modifiable. |

Avec `auto` seul, la reference du return type n’est pas conservée.

Avec `auto&`, le `const` de la string désignée est conservé.

```cpp
Player player;
player.SetName("Millie");

auto nameCopy = player.GetName();
auto& nameRef = player.GetName();

nameCopy = "Alice"; // Modifie seulement la copy.
// nameRef = "Bob"; // Erreur : const reference.

player.SetName("Chris");
// nameCopy contient toujours "Alice".
// nameRef permet maintenant de lire "Chris".
```

📖 **Déplier la type deduction :** [Type deduction with pointers, references, and const, LearnCpp (EN)](https://www.learncpp.com/cpp-tutorial/type-deduction-with-pointers-references-and-const/) détaille ce que `auto` conserve ou retire et ce que change `auto&`.

## 5. Default arguments d’une function

Notre setter initial exige un argument :

```cpp
player.SetName("Millie");
// player.SetName(); // Erreur avec le setter initial.
```

On peut lui donner un default argument :

```cpp
void SetName(const std::string& newName = "Default Name")
{
    name = newName;
}
```

Les deux appels deviennent valides :

```cpp
player.SetName("Millie"); // Utilise l’argument fourni.
player.SetName();        // Utilise le default argument.
```

Il s’agit toujours de la même function.

Le default argument doit être visible à l’endroit de l’appel. Si declaration et definition sont séparées, on le place généralement dans la declaration du header, sans le répéter dans le `.cpp`.

Pour une function ordinaire, les parameters qui suivent un parameter avec default argument doivent aussi disposer d’un default argument.

```cpp
void SpawnPlayer(int life = 100, int level = 1);

// Incorrect : level n’a pas de default argument.
// void SpawnPlayer(int life = 100, int level);
```

📖 **Exemples et règles :** [Default arguments, LearnCpp (EN)](https://www.learncpp.com/cpp-tutorial/default-arguments/) explique les arguments omis, leur ordre et leur emplacement dans une declaration ou un header.

## 6. Un class template : `Vector<T>`

### `Vector.h`

```cpp
#pragma once

template<typename T>
class Vector
{
public:
    T* elements = nullptr;
    int count = 0;
};
```

`T` est un type parameter. On choisit un type lors de l’utilisation :

```cpp
Vector<int> integers;
Vector<float> floats;
Vector<Player> players;
```

`Vector<int>` et `Vector<float>` sont deux types distincts.

Pour comprendre un template, commencer par remplacer son type parameter :

| Utilisation | Type de `elements` |
|---|---|
| `Vector<int>` | `int*` |
| `Vector<Player>` | `Player*` |
| `Vector<const float>` | `const float*` |
| `Vector<const float*>` | `const float**` |

Dans le dernier cas, `T` est déjà un pointer type. Le `*` de `T*` ajoute donc un niveau de pointer.

### Pourquoi `Vector<Player&>` échoue ici

Avec `T = Player&`, le member `T*` essaierait de former un pointer vers une reference, ce qui est interdit.

Cela ne signifie pas que les references sont interdites comme template arguments en général. Le problème vient de la manière dont notre classe utilise `T`.

### Ce `Vector` est un exemple pédagogique

> [!NOTE]
> **Un modèle volontairement minimal**
> Ce `Vector` ne réserve aucune mémoire et ne libère rien. Il ne fournit pas les fonctionnalités de `std::vector`.

```cpp
int values[] = { 10, 20, 30 };

Vector<int> numbers;
numbers.elements = values;
numbers.count = 3;

numbers.elements[1] = 99; // Modifie values[1].
```

Ici :

- L’array doit rester vivant pendant les accès.
- Les indices doivent rester dans les limites.
- `count` n’est pas calculé ni mis à jour automatiquement.
- Les initializations `nullptr` et `0` évitent de laisser les members avec des values indéterminées.

🖼️ **Pour une autre présentation :** [Templates, Hacking C++ (EN)](https://hackingcpp.com/cpp/lang/templates) propose une introduction en slides à la programmation avec des templates.

## 7. Default template arguments

On peut choisir un type par défaut :

```cpp
template<typename T = int>
class Vector
{
public:
    T* elements = nullptr;
    int count = 0;
};
```

On peut alors écrire :

```cpp
Vector<> integers;    // Vector<int>
Vector<float> floats;
```

Depuis C++17, dans cet exemple, cette forme fonctionne également :

```cpp
Vector integers; // Vector<int>
```

Cette omission des chevrons utilise la class template argument deduction (CTAD), disponible depuis C++17. Ici, le default template argument fournit le type `int`.

Sans default template argument, notre declaration `Vector vector;` ne fournit aucun moyen de choisir `T`.

À distinguer :

| Default argument | Ce qui est choisi |
|---|---|
| `int life = 100` dans une function | Une value pour un argument omis. |
| `typename T = int` dans un template | Un type pour un template argument omis. |

## 8. Reconnaître les templates de la standard library

```cpp
#include <memory>
#include <string>
#include <vector>

std::vector<int> scores;
std::basic_string<char> text;
std::unique_ptr<Player> player;
```

- `std::vector<int>` est un container d’entiers.
- `std::basic_string<char>` est le type dont `std::string` est un alias.
- `std::unique_ptr<Player>` est un smart pointer pouvant posséder un `Player`.

Le template argument joue un rôle différent selon le template. Tous les types ne conviennent pas à tous les templates.

### Instance directe ou smart pointer

Ces deux exemples sont indépendants :

```cpp
Player player;
player.SetName("Millie");
```

```cpp
auto player = std::make_unique<Player>();
player->SetName("Millie");
```

Attention :

```cpp
std::unique_ptr<Player> player;
```

> [!WARNING]
> **Un smart pointer vide ne contient pas de Player.**
> Cette ligne ne crée pas de `Player`. Remplacer `.` par `->` ne suffit donc pas : il faut aussi une instance valide avant l’accès.

---

## Exercices

Pour chaque exercice : prévoir le résultat, tester, puis expliquer les différences.

### Exercice 1 : lire les `const`

Dans `Player`, expliquer les trois usages de `const`.

Ajouter temporairement `life--;` dans `GetName()` :

- Pourquoi le compiler le refuse-t-il ?
- Pourquoi `SetName` peut-il modifier `name` malgré son parameter const ?
- Quels appels sont possibles sur un `const Player` ?

### Exercice 2 : copy ou reference ?

```cpp
Player player;
player.SetName("Millie");

auto first = player.GetName();
auto& second = player.GetName();
const auto third = player.GetName();

player.SetName("Alice");
first = "Bob";
```

1. Donner les types exacts de `first`, `second` et `third`.
2. Prédire leur contenu final et celui du member `name`.
3. Vérifier en affichant les quatre textes.
4. Expliquer pourquoi modifier `second` ou `third` serait refusé.

### Exercice 3 : ajouter des default arguments

Adapter le setter pour que :

```cpp
player.SetName();
```

remette le nom à `"Default Name"`.

Puis écrire une function :

```cpp
int ComputeDamage(int baseDamage, int bonus = 0);
```

Elle renvoie la somme des deux parameters.

Tester un appel avec un seul argument, puis avec deux arguments. Expliquer pourquoi `ComputeDamage()` sans argument reste incorrect.

### Exercice 4 : déplier les types

Avec notre `Vector<T>`, donner le type de `elements` pour :

```cpp
Vector<double> a;
Vector<Player*> b;
Vector<const Player> c;
Vector<const Player*> d;
// Vector<int&> e;
```

Expliquer pourquoi le dernier cas échoue.

Ajouter ensuite `int` comme default template argument et vérifier :

```cpp
Vector<> first;
Vector second;
Vector<float> third;
```

Quels sont les trois types obtenus ?

### Exercice 5 : utiliser `Vector<int>`

Créer un array contenant `5`, `10` et `15`, puis un `Vector<int>` pointant vers cet array.

1. Initialiser `elements` et `count`.
2. Afficher les éléments avec une boucle.
3. Ajouter `2` à chaque élément.
4. Vérifier les nouvelles values dans l’array d’origine.

Bonus : utiliser un `Vector<const int>` vers le même array. Vérifier que la lecture fonctionne mais qu’une modification par ce pointer est refusée.

### Exercice 6 : compiler ne signifie pas fonctionner

Analyser cet extrait sans l’exécuter :

```cpp
std::unique_ptr<Player> player;
player->SetName("Millie");
```

- Pourquoi peut-il compiler ?
- Pourquoi l’accès reste-t-il invalide ?
- Proposer une correction avec `std::make_unique`.
- Proposer une autre version avec une instance directe.

---

## À retenir

> [!TIP]
> **Les repères à garder**
>
> - `const` exprime une restriction, pas une promesse de performance.
> - `auto` déduit un type précis.
> - `auto` seul et `auto&` n’ont pas les mêmes effets sur les copies.
> - Un template décrit une famille de types ou de functions.
> - Un default argument fournit un choix lorsqu’un argument est omis.
> - Le compiler détecte certaines erreurs ; les résultats, limites et lifetimes restent à vérifier.

## Avant de terminer

- [ ] Je peux expliquer le rôle de chaque `const` dans `Player`.
- [ ] Je peux donner le type déduit par `auto`, `auto&` et `const auto&`.
- [ ] Je distingue une copy d’une reference vers un member existant.
- [ ] Je sais ajouter un default argument à une function et à un template.
- [ ] Je peux remplacer `T` pour retrouver le type de `elements`.
- [ ] Je sais pourquoi un smart pointer vide ne permet pas d’accéder à un `Player`.

---

## Références

- [const : restreindre les modifications](https://learn.microsoft.com/fr-fr/cpp/cpp/const-cpp?view=msvc-170) : Les usages de const sur les variables, les pointers et les member functions.
- [auto : déduire un type depuis une initialization](https://learn.microsoft.com/fr-fr/cpp/cpp/auto-cpp?view=msvc-170) : Les règles de type deduction et des exemples avec const, references et pointers.
- [Default arguments : permettre des arguments omis](https://learn.microsoft.com/fr-fr/cpp/cpp/default-arguments?view=msvc-170) : La syntaxe, la visibilité et les règles d’utilisation des arguments par défaut dans les functions.
- [Templates : écrire du code paramétré par des types](https://learn.microsoft.com/en-us/cpp/cpp/templates-cpp?view=msvc-170) : La définition et l’utilisation des function templates et class templates, avec des paramètres par défaut.
- [CTAD : comment les arguments d’un class template sont déduits](https://eel.is/c++draft/over.match.class.deduct) : Les règles formelles des deduction guides dans le C++ working draft. Lecture avancée pour approfondir l’exemple Vector.

---

[← Révision 2](./02_Values_References_Pointers.md) · [Sommaire](./README.md) · [Révision 4 →](./04_Design_Method.md)
