## 1\. Les macros procédurales (ou proc-macros)

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

Les macros procédurales (ou proc-macros) sont des outils puissants dans l'écosystème Rust qui permettent de générer, analyser et transformer du code pendant la compilation. Contrairement aux macros déclaratives (`macro_rules!`), les proc-macros peuvent manipuler l'arbre syntaxique directement, ce qui leur confère une grande flexibilité.

### Introduction aux proc-macros

Vous avez probablement déjà utilisé des proc-macros sans le savoir. L'exemple le plus courant est l'attribut `#[derive(...)]` :

``` rust
#[derive(Debug, Clone, PartialEq)]
pub struct Point {
    x: f64,
    y: f64,
}
```

Dans cet exemple, `#[derive(Debug, Clone, PartialEq)]` est une proc-macro qui génère automatiquement les implémentations des traits `Debug`, `Clone` et `PartialEq` pour notre structure `Point`.

### Types de proc-macros

Il existe trois types de proc-macros en Rust :

1.  **Les proc-macros fonction** (function-like macros) : ressemblent aux macros déclaratives dans leur utilisation (`nom_macro!(...)`)
2.  **Les macros dérivées** (derive macros) : utilisées avec l'attribut `#[derive(MacroName)]`
3.  **Les macros attributs** (attribute macros) : utilisées comme des attributs sur des items (`#[nom_attribut]`)

### Configuration du projet

Pour créer une proc-macro, vous devez établir un projet spécifique de type bibliothèque. Voici comment configurer votre `Cargo.toml` :

``` toml
[package]
name = "ma_proc_macro"
version = "0.1.0"
edition = "2021"

[lib]
proc-macro = true

[dependencies]
syn = { version = "2.0", features = ["full"] }
quote = "1.0"
proc-macro2 = "1.0"
```

L'option `proc-macro = true` est essentielle car elle indique à Rust que cette bibliothèque contient des macros procédurales et donne accès à la crate `proc_macro` qui est nécessaire pour leur développement.

Les dépendances `syn`, `quote` et `proc-macro2` sont presque indispensables pour développer des proc-macros modernes :

- `syn` : analyse syntaxique du code Rust
- `quote` : génération facile de code Rust
- `proc-macro2` : version plus flexible de `proc_macro`

### Function-like macro

Commençons par un exemple simple de macro de type fonction :
``` toml
[package]
name = "ma_lib"
version = "0.1.0"
edition = "2024"

[lib]
proc-macro = true

[dependencies]

```

``` rust
// ma_lib/src/lib.rs
use proc_macro::TokenStream;

#[proc_macro]
pub fn log_vars(input: TokenStream) -> TokenStream {
    let input_str = input.to_string();

    let vars: Vec<&str> = input_str.split(',')
        .map(|s| s.trim())
        .collect();

    let mut output = String::new();
    output.push_str("{\n");

    for var in vars {
        output.push_str(&format!("    println!(\"{}={{:?}}\", {});\n", var, var));
    }

    output.push_str("}");

    output.parse().unwrap()
}
```

Cette macro peut être utilisée comme suit :
``` toml
[dependencies]
ma_lib = { path = "../ma_lib" }
```

``` rust
// main_project/src/main.rs
use ma_lib::log_vars;

fn main() {
    let nombre = 42;
    let message = "Bonjour, monde!";
    let tableau = [1, 2, 3, 4, 5];

    log_vars!(nombre, message, tableau);
}
```

À la compilation, cela générera :

``` rust
{
    println!("x={:?}", x);
    println!("nom={:?}", nom);
    println!("liste={:?}", liste);
}
```

et en sortie :
``` bash
nombre=42
message="Bonjour, monde!"
tableau=[1, 2, 3, 4, 5]

Process finished with exit code 0

```

### Derive macro avec syn et quote

Les macros dérivées sont plus complexes mais aussi plus puissantes. Améliorons l'exemple de la macro `GetSet` en utilisant `syn` et `quote` pour générer des getters et setters avec plus d'options :
``` toml
[dependencies]
quote = "1.0.40"
syn = "2.0.100"
```

``` rust
use proc_macro::TokenStream;
use quote::{quote, format_ident};
use syn::{parse_macro_input, DeriveInput, Data, Fields, FieldsNamed, Type, Visibility, Ident};

#[proc_macro_derive(GetSet, attributes(skip_getter, skip_setter, rename))]
pub fn derive_getters_setters(input: TokenStream) -> TokenStream {
    // Analyser l'entrée
    let input = parse_macro_input!(input as DeriveInput);

    // Récupérer le nom de la structure et ses génériques
    let struct_name = &input.ident;
    let generics = &input.generics;
    let (impl_generics, ty_generics, where_clause) = generics.split_for_impl();

    // Traiter uniquement les structures avec des champs nommés
    let fields = match &input.data {
        Data::Struct(data_struct) => {
            match &data_struct.fields {
                Fields::Named(fields_named) => fields_named,
                _ => return TokenStream::from(quote! {
                    compile_error!("GetSet ne supporte que les structures avec des champs nommés");
                }),
            }
        },
        _ => return TokenStream::from(quote! {
            compile_error!("GetSet ne peut être utilisé que sur des structures");
        }),
    };

    // Générer les getters et setters
    let mut getters = Vec::new();
    let mut setters = Vec::new();

    for field in &fields.named {
        let field_name = field.ident.as_ref().unwrap();
        let field_type = &field.ty;
        let field_vis = &field.vis;

        // Vérifier les attributs spéciaux
        let mut skip_getter = false;
        let mut skip_setter = false;
        let mut custom_name = None;

        for attr in &field.attrs {
            if attr.path().is_ident("skip_getter") {
                skip_getter = true;
            } else if attr.path().is_ident("skip_setter") {
                skip_setter = true;
            } else if attr.path().is_ident("rename") {
                // Extraire le nom personnalisé
                custom_name = attr.parse_args::<Ident>().ok();
            }
        }

        // Générer le getter si nécessaire
        if !skip_getter {
            let getter_name = match &custom_name {
                Some(name) => format_ident!("get_{}", name),
                None => format_ident!("get_{}", field_name),
            };

            getters.push(quote! {
                #field_vis fn #getter_name(&self) -> &#field_type {
                    &self.#field_name
                }
            });
        }

        // Générer le setter si nécessaire
        if !skip_setter {
            let setter_name = match &custom_name {
                Some(name) => format_ident!("set_{}", name),
                None => format_ident!("set_{}", field_name),
            };

            setters.push(quote! {
                #field_vis fn #setter_name(&mut self, value: #field_type) {
                    self.#field_name = value;
                }
            });
        }
    }

    // Assembler le résultat final
    let expanded = quote! {
        impl #impl_generics #struct_name #ty_generics #where_clause {
            #(#getters)*

            #(#setters)*
        }
    };

    TokenStream::from(expanded)
}
```

Cette version améliorée permet :

- De sauter la génération de getters ou setters avec `#[skip_getter]` et `#[skip_setter]`
- De renommer les méthodes avec `#[rename(autre_nom)]`
- De conserver la visibilité des champs
- De gérer correctement les génériques

Exemple d'utilisation :

``` rust
// main_project/src/main.rs
use ma_lib::GetSet;

#[derive(GetSet)]
pub struct Personne<T> {
    #[skip_setter]
    id: u32,

    pub name: String,

    #[rename(email_address)]
    pub(crate) email: String,

    #[skip_getter]
    mot_de_passe: String,

    metadata: T,
}

fn main() {
    let mut p = Personne {
        id: 1,
        name: "Jean".to_string(),
        email: "jean@example.com".to_string(),
        mot_de_passe: "secret".to_string(),
        metadata: 42,
    };

    // Utiliser les méthodes générées
    println!("ID: {}", p.get_id());
    println!("Nom: {}", p.get_name());
    println!("Email: {}", p.get_email_address());

    p.set_name("Pierre".to_string());
    p.set_email_address("pierre@example.com".to_string());
    p.set_metadata(100);

    // N'existe pas car on a utilisé skip_getter
    // p.get_mot_de_passe();

    // N'existe pas car on a utilisé skip_setter
    // p.set_id(2);
}
```

``` bash
ID: 1
Nom: Jean
Email: jean@example.com

```
### Macro attribut

Les macros attributs peuvent transformer complètement un item Rust. Voici un exemple de macro attribut qui transforme une fonction en ajoutant du logging automatique des appels et du temps d'exécution :
``` toml
[lib]
proc-macro = true

[dependencies]
quote = "1.0.40"
syn = { version = "2.0.100", features = ["full"] }
```

``` rust
use proc_macro::TokenStream;
use quote::{quote, format_ident};
use syn::{parse_macro_input, ItemFn, parse_str, Expr, Attribute, Lit, Meta};
use syn::parse::Parser;
use syn::punctuated::Punctuated;
use syn::token::Comma;
use syn::meta::ParseNestedMeta;

#[proc_macro_attribute]
pub fn log_function(args: TokenStream, input: TokenStream) -> TokenStream {
    // Analyser la fonction
    let mut item_fn = parse_macro_input!(input as ItemFn);
    let fn_name = &item_fn.sig.ident;
    let fn_vis = &item_fn.vis;
    let fn_block = &item_fn.block;

    // Valeur par défaut pour le niveau de log
    let mut log_target = quote!(log::info);

    // Utilisation de la nouvelle API de syn pour parser les attributs
    let mut parser = syn::meta::parser(|meta| {
        if meta.path.is_ident("level") {
            let value = meta.value()?;
            let level_str: syn::LitStr = value.parse()?;
            let level = level_str.value();

            match level.as_str() {
                "trace" => log_target = quote!(log::trace),
                "debug" => log_target = quote!(log::debug),
                "info" => log_target = quote!(log::info),
                "warn" => log_target = quote!(log::warn),
                "error" => log_target = quote!(log::error),
                _ => return Err(meta.error("Niveau de log invalide, utilisez: trace, debug, info, warn, error")),
            }

            Ok(())
        } else {
            Err(meta.error("Attribut non reconnu"))
        }
    });

    // Parser les arguments de l'attribut
    if let Err(err) = parser.parse(args.into()) {
        return TokenStream::from(err.to_compile_error());
    }

    // Générer le nouveau corps de fonction avec logging
    let inputs = &item_fn.sig.inputs;
    let output = &item_fn.sig.output;

    let expanded = quote! {
        #fn_vis fn #fn_name(#inputs) #output {
            #log_target!("Début de l'exécution de la fonction {}", stringify!(#fn_name));

            let start_time = std::time::Instant::now();

            let result = {
                #fn_block
            };

            let duration = start_time.elapsed();
            #log_target!("Fin de l'exécution de {} en {:?}", stringify!(#fn_name), duration);

            result
        }
    };

    TokenStream::from(expanded)
}
```

Exemple d'utilisation :
``` toml
[dependencies]
ma_lib = { path = "../ma_lib" }
env_logger = "0.11.8"
log = "0.4"
```

``` rust
use ma_proc_macro::log_function;

// Utilisez la macro avec le niveau de log par défaut (info)
#[log_function]
fn calcul_complexe(iterations: u32) -> u64 {
    let mut somme = 0;
    for i in 0..iterations {
        somme += i as u64 * i as u64;
        std::thread::sleep(std::time::Duration::from_millis(1));
    }
    somme
}

// Utilisez la macro avec un niveau de log spécifique
#[log_function(level = "debug")]
fn autre_fonction() {
    println!("Exécution d'une autre fonction");
}

fn main() {
    // Configurez l'environnement de logging
    env_logger::init();

    calcul_complexe(1000);
    autre_fonction();
}
```

### Debugging des proc-macros

Le débogage des proc-macros peut être difficile car elles s'exécutent pendant la compilation. Voici quelques techniques utiles :

1.  **Affichage pendant la compilation** :

``` rust
#[proc_macro_derive(MyDerive)]
pub fn my_derive(input: TokenStream) -> TokenStream {
    eprintln!("Input: {}", input);
    // ...
}
```

2.  **Panic avec messages informatifs** :

``` rust
if condition_problématique {
    panic!("Erreur dans la macro: {:?}", problème);
}
```

3.  **Utilisation du module `proc_macro2::TokenStream::from_string`** pour tester vos macros sans compiler :

``` rust
#[test]
fn test_ma_macro() {
    let input = proc_macro2::TokenStream::from_str("struct Test { field: u32 }").unwrap();
    let output = ma_macro_interne(input);
    assert!(output.to_string().contains("get_field"));
}
```

4.  **L'attribut `#[proc_macro_error]`** de la crate `proc-macro-error` pour de meilleurs messages d'erreur.

### Bonnes pratiques

1.  **Validation des entrées** : Toujours vérifier que les entrées correspondent à ce que votre macro attend.
2.  **Messages d'erreur clairs** : Utilisez `compile_error!` pour indiquer précisément les problèmes.
3.  **Documentation** : Documentez votre macro en détail, notamment les attributs spéciaux qu'elle supporte.
4.  **Tests** : Utilisez des tests unitaires pour vérifier que votre macro génère le code attendu.
5.  **Modularité** : Séparez l'analyse et la génération de code en fonctions distinctes.

### Conclusion

Les macros procédurales sont un outil puissant pour la métaprogrammation en Rust. Elles permettent d'automatiser les tâches répétitives, de générer du code à partir de spécifications, et d'étendre le langage avec de nouvelles fonctionnalités.

Contrairement aux macros déclaratives qui fonctionnent par pattern-matching, les proc-macros manipulent directement l'arbre syntaxique, ce qui leur donne beaucoup plus de flexibilité. Cette puissance s'accompagne d'une complexité accrue, mais les bibliothèques comme `syn` et `quote` rendent leur développement beaucoup plus accessible.

Pour aller plus loin, vous pouvez explorer des macros bien connues comme `serde_derive`, `thiserror` ou `async-trait` dont le code source est disponible publiquement et qui démontrent des utilisations avancées des proc-macros.

⏭️ [La compilation conditionnelle](/III-avance/02-compilation-conditionnelle.md)
