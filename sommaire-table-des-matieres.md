# Sommaire

## I. Les bases de la programmation en Rust

1.  [Présentation de Rust](# "#1-pr%C3%A9sentation-de-rust")
2.  [Mise en place des outils](#2-mise-en-place-des-outils)
3.  [Premier programme](#3-premier-programme)
4.  [Variables](#4-variables)
5.  [Conditions et pattern matching](#5-conditions-et-pattern-matching)
6.  [Les fonctions](#6-les-fonctions)
7.  [Les expressions](#7-les-expressions)
8.  [Les boucles](#8-les-boucles)
9.  [Les enums](#9-les-enums)
10. [Les structures](#10-les-structures)
11. [if let / while let](#11-if-let--while-let)
12. [Gestion des erreurs](#12-gestion-des-erreurs)
13. [Cargo](#13-cargo)
14. [Utiliser des bibliothèques externes](# "#14-utiliser-des-biblioth%C3%A8ques-externes")
15. [Jeu de devinette de mots](#15-jeu-de-devinette-de-mots)

## II. Spécificités de Rust

1.  [Le formatage des flux](#1-le-formatage-des-flux)
2.  [Les traits](#2-les-traits)
3.  [Les attributs](#3-les-attributs)
4.  [Généricité](# "#4-g%C3%A9n%C3%A9ricit%C3%A9")
5.  [Propriété (ou ownership)](# "#5-propri%C3%A9t%C3%A9-ou-ownership")
6.  [Durée de vie (ou lifetime)](# "#6-dur%C3%A9e-de-vie-ou-lifetime")
7.  [Déréférencement](# "#7-d%C3%A9r%C3%A9f%C3%A9rencement")
8.  [Sized et String vs str](#8-sized-et-string-vs-str)
9.  [Unsafe](#9-unsafe)
10. [Les unions](#10-les-unions)
11. [Closure](#11-closure)
12. [Multi-fichier](#12-multi-fichier)
13. [Les macros](#13-les-macros)
14. [Box](#14-box)
15. [Les itérateurs](# "#15-les-it%C3%A9rateurs")
16. [Les traits objets et dynamic dispatch](#16-les-traits-objets-et-dynamic-dispatch) - Approfondissement sur `dyn Trait`
17. [Pattern matching avancé](# "#17-pattern-matching-avanc%C3%A9") - Patterns plus complexes, guards, etc.
18. [Async/Await et programmation asynchrone](#18-asyncawait-et-programmation-asynchrone) - Introduction aux futures et à la programmation asynchrone
19. [Pinning et futures](#19-pinning-et-futures) - Concept de pinning pour les types non déplaçables
20. [Les traits avancés](# "#20-les-traits-avanc%C3%A9s") - Auto Traits, Marker Traits, Trait Bounds complexes
21. [Le système de types avancé](# "#21-le-syst%C3%A8me-de-types-avanc%C3%A9") - Associated Types, GAT (Generic Associated Types), HRTB
22. [Les atomics et la mémoire ordonnée](# "#22-les-atomics-et-la-m%C3%A9moire-ordonn%C3%A9e") - `std::sync::atomic` et modèles de mémoire

## III. Aller plus loin

1.  [Les macros procédurales (ou proc-macros)](# "#1-les-macros-proc%C3%A9durales-ou-proc-macros")
2.  [La compilation conditionnelle](#2-la-compilation-conditionnelle)
3.  [Utiliser du code compilé en C](# "#3-utiliser-du-code-compil%C3%A9-en-c")
4.  [Documentation et rustdoc](#4-documentation-et-rustdoc)
5.  [Ajouter des tests](#5-ajouter-des-tests)
6.  [Rc et RefCell](#6-rc-et-refcell)
7.  [Le multi-threading](#7-le-multi-threading)
8.  [Le réseau](# "#8-le-r%C3%A9seau")
9.  [Optimisation des performances](#9-optimisation-des-performances) - Profiling, benchmarking avec Criterion
10. [Rust embarqué](# "#10-rust-embarqu%C3%A9") - Introduction à la programmation sur microcontrôleurs
11. [WebAssembly avec Rust](#11-webassembly-avec-rust) - Compilation vers WASM et interopérabilité avec JavaScript
12. [Interface graphique](#12-interface-graphique) - Introduction aux frameworks GUI disponibles (GTK, Egui, etc.)
13. [Base de données](# "#13-base-de-donn%C3%A9es") - Connexion et manipulation de bases de données
14. [Génération de bindings](# "#14-g%C3%A9n%C3%A9ration-de-bindings") - Création d'interfaces pour d'autres langages
15. [Écosystème Rust pour la cryptographie](# "#15-%C3%A9cosyst%C3%A8me-rust-pour-la-cryptographie") - Utilisation des crates de hachage et cryptographie (sha2, sha1, md5)
16. [Internationalisation et localisation](#16-internationalisation-et-localisation) - Gestion des langues et formats internationaux
17. [Sérialisation et désérialisation](# "#17-s%C3%A9rialisation-et-d%C3%A9s%C3%A9rialisation") - Utilisation de serde pour JSON, YAML, etc.
18. [Création de crates et publication](# "#18-cr%C3%A9ation-de-crates-et-publication") - Comment publier sur crates.io
19. [Les attributs de compilation et optimisation](#19-les-attributs-de-compilation-et-optimisation) - Flags de compilation, optimisations spécifiques
20. [Rust pour le domaine scientifique](#20-rust-pour-le-domaine-scientifique) - Calcul numérique, interfaçage avec Python, etc.
21. [Zero-cost abstractions avancées](# "#21-zero-cost-abstractions-avanc%C3%A9es") - Exemples et techniques pour créer des abstractions sans coût à l'exécution

## Conclusion du tutoriel Rust
