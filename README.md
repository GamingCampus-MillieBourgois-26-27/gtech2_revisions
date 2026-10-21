# GTech 2 — Révisions C++

**Gaming Campus · GTech 2 · C++20**

Des fiches pour reprendre les notions vues en cours, comprendre les exemples et s’entraîner à écrire du code.

Les explications sont en français et les termes techniques restent en anglais : ce sont ceux utilisés dans le code, la documentation et les messages du compiler.

## Sommaire

| Jour | Fiche | Notions principales |
| :---: | --- | --- |
| **01** | [Functions, files et compiler](./01_Functions_Files_Compiler.md) | Parameters, return values, declarations et definitions, headers, preprocessor, compiler, linker. |
| **02** | [Values, references et pointers](./02_Values_References_Pointers.md) | Copies, addresses, parameter passing, `nullptr`, scope, lifetime, dangling pointers. |
| **03** | [const, auto et templates](./03_Const_Auto_Templates.md) | Const correctness, type deduction, default arguments, class templates, premiers repères dans la standard library. |

**Commencer par la [révision 1 →](./01_Functions_Files_Compiler.md)**

## Comment utiliser ces fiches

Chaque fiche suit la même organisation : objectifs, rappels avec exemples, exercices, points à retenir, checklist et références.

1. Lire les rappels et prévoir le comportement des exemples avant de les lancer.
2. Écrire et tester les exercices dans un projet C++.
3. Expliquer les écarts entre la prédiction et le résultat.
4. Utiliser la checklist de fin pour repérer les notions à retravailler.

> [!TIP]
> Pour les exercices de diagnostic, lire le message d’erreur et identifier l’étape concernée : preprocessing, compilation, linking ou runtime.

## Compiler les exemples

- Utiliser un compiler configuré en **C++20**, par exemple MSVC, GCC ou Clang.
- Les exemples d’affichage utilisent `std::cout` avec `<iostream>`.
- Les blocs de code sont parfois des extraits ou des versions successives : lire le contexte avant de les assembler.
- Quand plusieurs files sont présentés, conserver leurs noms et inclure tous les `.cpp` nécessaires dans le projet.

> [!NOTE]
> Certains extraits servent à analyser une erreur de compilation, de linking ou un accès invalide. Ceux à analyser sans exécution sont signalés dans les consignes. Un programme qui compile peut encore contenir des erreurs de logique ou un undefined behavior.

## Repères de vocabulaire

| Terme technique | Sens dans les fiches |
| --- | --- |
| Function / parameter / argument | Fonction / paramètre déclaré / expression fournie à l’appel. |
| Return value | Valeur transmise au code appelant par une function. |
| Declaration / definition | Annoncer une entité / fournir sa définition, notamment le body d’une function. |
| Header / source file | Fichier d’en-tête, ici `.h` / fichier source, ici `.cpp`. |
| Translation unit | Un `.cpp` et le contenu de ses inclusions, après preprocessing. |
| Compiler / linker | Vérifier et compiler chaque translation unit / réunir les object files et les libraries. |
| Value / copy / reference / pointer | Valeur / copie / autre nom pour un objet / variable contenant une adresse. |
| Scope / lifetime | Portée d’un nom / durée de vie d’un objet. |
| Template / type deduction | Modèle paramétré / déduction d’un type par le compiler. |
| Build / runtime | Construction du programme / période d’exécution du programme. |

## Progression

- [ ] **Jour 1** — Je sais organiser une function entre un header et un source file, puis identifier une erreur du compiler ou du linker.
- [ ] **Jour 2** — Je sais prévoir les effets d’une copy, d’une reference et d’un pointer sur la variable d’origine.
- [ ] **Jour 3** — Je sais lire les `const`, prévoir les types déduits et utiliser un template avec ou sans default argument.

Les références pour approfondir se trouvent à la fin de chaque fiche.
