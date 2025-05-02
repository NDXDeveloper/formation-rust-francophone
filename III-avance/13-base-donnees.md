# 13\. **Base de données** - Connexion et manipulation de bases de données

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction
L'interaction avec les bases de données reste une composante essentielle des applications modernes. Pour la Rust 2024 Edition, les bibliothèques de base de données ont évolué avec de nouvelles fonctionnalités et améliorations. Voici une mise à jour du tutoriel original adapté aux dernières versions des bibliothèques.
## Bibliothèques populaires pour les bases de données en Rust
### 1. SQLx
SQLx a évolué depuis la version 0.7 mentionnée dans le tutoriel original.
#### Installation
``` toml
[dependencies]
sqlx = { version = "0.7.4", features = ["runtime-tokio", "postgres", "sqlite", "mysql", "time"] }
tokio = { version = "1.44", features = ["full"] }
```
#### Exemple avec PostgreSQL
``` rust
use sqlx::{postgres::PgPoolOptions, Row};
use std::error::Error;

#[derive(Debug)]
struct Utilisateur {
    id: i32,
    nom: String,
    email: String,
    age: Option<i32>,
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn Error>> {
    // Établir une connexion à la base de données avec des options plus modernes
    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect("postgres://utilisateur:motdepasse@localhost/mabasededonnees")
        .await?;

    // Créer une table
    sqlx::query(
        r#"
        CREATE TABLE IF NOT EXISTS utilisateurs (
            id SERIAL PRIMARY KEY,
            nom TEXT NOT NULL,
            email TEXT NOT NULL UNIQUE,
            age INTEGER
        )
        "#,
    )
    .execute(&pool)
    .await?;

    // Insérer des données - Notez l'utilisation de .try_map() pour gérer les erreurs
    let nouvel_utilisateur = sqlx::query(
        r#"
        INSERT INTO utilisateurs (nom, email, age)
        VALUES ($1, $2, $3)
        RETURNING id, nom, email, age
        "#,
    )
    .bind("Marie Dupont")
    .bind("marie@exemple.com")
    .bind(32)
    .fetch_one(&pool)
    .await?;

    // Créer une instance à partir des résultats
    let utilisateur = Utilisateur {
        id: nouvel_utilisateur.get("id"),
        nom: nouvel_utilisateur.get("nom"),
        email: nouvel_utilisateur.get("email"),
        age: nouvel_utilisateur.get("age"),
    };

    println!("Nouvel utilisateur créé: {:?}", utilisateur);

    // Requête avec paramètres - Avec gestion d'erreur améliorée
    let utilisateurs = sqlx::query_as!(
        Utilisateur,
        "SELECT id, nom, email, age FROM utilisateurs WHERE age > $1",
        25
    )
    .fetch_all(&pool)
    .await?;

    println!("Utilisateurs de plus de 25 ans:");
    for u in utilisateurs {
        println!("- {} ({}) - {}", u.nom, u.email, u.age.unwrap_or(0));
    }

    Ok(())
}
```
### 2. Diesel
Diesel a également évolué vers une version plus récente avec des améliorations.
#### Installation
``` toml
[dependencies]
diesel = { version = "2.1.5", features = ["postgres", "r2d2"] }
dotenvy = "0.15.7"
```
#### Exemple avec PostgreSQL
``` rust
use diesel::prelude::*;
use diesel::pg::PgConnection;
use dotenvy::dotenv;
use std::env;

// Définition du schéma
mod schema {
    diesel::table! {
        articles (id) {
            id -> Int4,
            titre -> Varchar,
            contenu -> Text,
            publie -> Bool,
        }
    }
}

use schema::articles;

// Définition des modèles avec #[derive] amélioré pour la Rust 2024
#[derive(Queryable, Selectable, Debug)]
#[diesel(table_name = articles)]
#[diesel(check_for_backend(diesel::pg::Pg))]
struct Article {
    id: i32,
    titre: String,
    contenu: String,
    publie: bool,
}

#[derive(Insertable)]
#[diesel(table_name = articles)]
struct NouvelArticle<'a> {
    titre: &'a str,
    contenu: &'a str,
    publie: bool,
}

fn établir_connexion() -> PgConnection {
    dotenv().ok();

    let database_url = env::var("DATABASE_URL").expect("DATABASE_URL doit être définie");
    PgConnection::establish(&database_url)
        .unwrap_or_else(|_| panic!("Erreur de connexion à {}", database_url))
}

fn créer_article(conn: &mut PgConnection, titre: &str, contenu: &str) -> Article {
    let nouvel_article = NouvelArticle {
        titre,
        contenu,
        publie: false,
    };

    diesel::insert_into(articles::table)
        .values(&nouvel_article)
        .returning(Article::as_returning())
        .get_result(conn)
        .expect("Erreur lors de la création du nouvel article")
}

fn main() {
    let mut conn = établir_connexion();

    let article = créer_article(
        &mut conn,
        "Le langage Rust",
        "Rust est un langage de programmation compilé multi-paradigme développé par Mozilla Research."
    );

    println!("Article créé avec succès: {:?}", article);

    // Récupérer tous les articles avec une syntaxe légèrement améliorée dans Diesel 2.1
    let résultats = articles::table
        .filter(articles::publie.eq(false))
        .limit(5)
        .select(Article::as_select())
        .load(&mut conn)
        .expect("Erreur lors de la récupération des articles");

    println!("Articles non publiés trouvés: {}", résultats.len());
    for article in résultats {
        println!("{}: {}", article.titre, article.contenu);
    }
}
```
### 3. Rusqlite (pour SQLite)
La bibliothèque Rusqlite a également reçu des mises à jour importantes.
#### Installation
``` toml
[dependencies]
rusqlite = { version = "0.31.0", features = ["bundled"] }
```
#### Exemple avec SQLite
``` rust
use rusqlite::{params, Connection, Result};
use std::path::Path;

#[derive(Debug)]
struct Personne {
    id: i32,
    nom: String,
    données: Option<Vec<u8>>,
}

fn main() -> Result<()> {
    let chemin_db = Path::new("ma_base.db3");
    let existe_déjà = chemin_db.exists();

    // Connexion améliorée avec meilleure gestion des options
    let conn = Connection::open(chemin_db)?;

    // Créer la table si elle n'existe pas
    if !existe_déjà {
        conn.execute(
            "CREATE TABLE personnes (
                id    INTEGER PRIMARY KEY,
                nom   TEXT NOT NULL,
                données  BLOB
            )",
            [],
        )?;
    }

    // Insérer une nouvelle personne
    let jean = Personne {
        id: 0, // Sera ignoré et généré automatiquement
        nom: "Jean Dupont".to_string(),
        données: Some(vec![1, 2, 3, 4]),
    };

    conn.execute(
        "INSERT INTO personnes (nom, données) VALUES (?1, ?2)",
        params![jean.nom, jean.données],
    )?;

    // Récupérer le dernier ID inséré
    let id = conn.last_insert_rowid();
    println!("Personne insérée avec l'ID: {}", id);

    // Requête pour récupérer des données - API améliorée avec gestion des erreurs
    let mut stmt = conn.prepare("SELECT id, nom, données FROM personnes")?;
    let iter_personnes = stmt.query_map([], |row| {
        Ok(Personne {
            id: row.get(0)?,
            nom: row.get(1)?,
            données: row.get(2)?,
        })
    })?;

    for personne in iter_personnes {
        println!("Trouvé personne {:?}", personne?);
    }

    // Utilisation d'une transaction avec API améliorée
    let tx = conn.transaction()?;

    tx.execute(
        "INSERT INTO personnes (nom) VALUES (?1)",
        params!["Marie Martin"],
    )?;

    tx.execute(
        "INSERT INTO personnes (nom) VALUES (?1)",
        params!["Sophie Bernard"],
    )?;

    tx.commit()?;

    println!("Personnes dans la base de données:");
    let mut stmt = conn.prepare("SELECT id, nom FROM personnes ORDER BY id")?;
    let iter_noms = stmt.query_map([], |row| {
        let id: i32 = row.get(0)?;
        let nom: String = row.get(1)?;
        Ok(format!("#{}: {}", id, nom))
    })?;

    for nom in iter_noms {
        println!("{}", nom?);
    }

    Ok(())
}
```
### 4. MongoDB avec MongoDB Rust Driver
Le pilote MongoDB pour Rust a subi plusieurs mises à jour importantes.
#### Installation
``` toml
[dependencies]
mongodb = "2.8.1"
tokio = { version = "1.44", features = ["full"] }
serde = { version = "1.0.219", features = ["derive"] }
futures = "0.3.31"
```
#### Exemple avec MongoDB
``` rust
use mongodb::{Client, options::ClientOptions};
use mongodb::bson::{doc, Document};
use serde::{Deserialize, Serialize};
use futures::stream::TryStreamExt;

#[derive(Debug, Serialize, Deserialize)]
struct Produit {
    #[serde(rename = "_id", skip_serializing_if = "Option::is_none")]
    id: Option<mongodb::bson::oid::ObjectId>,
    nom: String,
    prix: f64,
    stock: i32,
    catégorie: String,
}

#[tokio::main]
async fn main() -> mongodb::error::Result<()> {
    // Connexion à MongoDB avec options améliorées
    let client_options = ClientOptions::parse("mongodb://localhost:27017").await?;
    let client = Client::with_options(client_options)?;

    // Accès à la base de données et à la collection
    let db = client.database("boutique");
    let collection = db.collection::<Produit>("produits");

    // Insertion d'un document
    let produit = Produit {
        id: None,
        nom: "Ordinateur portable".to_string(),
        prix: 1299.99,
        stock: 10,
        catégorie: "Électronique".to_string(),
    };

    let résultat_insertion = collection.insert_one(produit, None).await?;
    println!("Produit inséré avec l'ID: {}", résultat_insertion.inserted_id);

    // Insertion de plusieurs documents
    let produits = vec![
        Produit {
            id: None,
            nom: "Smartphone".to_string(),
            prix: 899.99,
            stock: 15,
            catégorie: "Électronique".to_string(),
        },
        Produit {
            id: None,
            nom: "Casque sans fil".to_string(),
            prix: 199.99,
            stock: 20,
            catégorie: "Accessoires".to_string(),
        }
    ];

    let résultat_insertion_multi = collection.insert_many(produits, None).await?;
    println!("Nombre de produits insérés: {}", résultat_insertion_multi.inserted_ids.len());

    // Recherche avec API améliorée
    let filtre = doc! { "catégorie": "Électronique" };
    let mut cursor = collection.find(filtre, None).await?;

    println!("Produits électroniques:");
    while let Some(produit) = cursor.try_next().await? {
        println!(" - {} ({}€) - {} en stock", produit.nom, produit.prix, produit.stock);
    }

    // Mise à jour avec méthodes améliorées
    let filtre = doc! { "nom": "Smartphone" };
    let mise_à_jour = doc! { "$set": { "stock": 12 } };
    let résultat_mise_à_jour = collection.update_one(filtre, mise_à_jour, None).await?;

    println!(
        "Mise à jour effectuée: {} document(s) modifié(s)",
        résultat_mise_à_jour.modified_count
    );

    // Agrégation avec pipeline
    let pipeline = vec![
        doc! { "$group": { "_id": "$catégorie", "prix_moyen": { "$avg": "$prix" } } }
    ];

    let mut cursor = collection.aggregate(pipeline, None).await?;

    println!("Prix moyen par catégorie:");
    while let Some(résultat) = cursor.try_next().await? {
        let doc: Document = résultat;
        println!(" - {}: {}€", doc.get_str("_id")?, doc.get_f64("prix_moyen")?);
    }

    Ok(())
}
```
## Abstractions multi-bases de données
### SeaORM
SeaORM a également connu des améliorations notables.
#### Installation
``` toml
[dependencies]
sea-orm = { version = "0.12.15", features = ["sqlx-postgres", "runtime-tokio-native-tls", "macros"] }
tokio = { version = "1.44", features = ["full"] }
```
#### Exemple avec SeaORM
``` rust
use sea_orm::{Database, EntityTrait, Set, ActiveModelTrait, DatabaseConnection};
use sea_orm::entity::prelude::*;

// Définition de l'entité avec améliorations pour Rust 2024
#[derive(Clone, Debug, PartialEq, DeriveEntityModel)]
#[sea_orm(table_name = "livres")]
pub struct Model {
    #[sea_orm(primary_key)]
    pub id: i32,
    pub titre: String,
    pub auteur: String,
    #[sea_orm(column_type = "Text", nullable)]
    pub description: Option<String>,
    pub année_publication: i32,
    pub disponible: bool,
}

#[derive(Copy, Clone, Debug, EnumIter, DeriveRelation)]
pub enum Relation {}

impl ActiveModelBehavior for ActiveModel {}

pub struct Livre;
impl EntityName for Livre {
    fn table_name(&self) -> &str {
        "livres"
    }
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Connexion à la base de données avec options améliorées
    let db: DatabaseConnection = Database::connect("postgres://utilisateur:motdepasse@localhost/bibliotheque").await?;

    // Création de la table si elle n'existe pas
    let schema = Schema::new(DbBackend::Postgres);
    let create_table_stmt = schema.create_table_from_entity(Entity);
    db.execute(db.get_database_backend().build(&create_table_stmt)).await?;

    // Création d'un nouveau livre (ActiveModel)
    let nouveau_livre = ActiveModel {
        titre: Set("1984".to_owned()),
        auteur: Set("George Orwell".to_owned()),
        description: Set(Some("Un roman dystopique sur la surveillance de masse.".to_owned())),
        année_publication: Set(1949),
        disponible: Set(true),
        ..Default::default()
    };

    // Insertion dans la base de données avec API améliorée
    let livre_inséré: Model = nouveau_livre.insert(&db).await?;
    println!("Livre inséré: {:?}", livre_inséré);

    // Recherche de livres avec API de requête améliorée
    let livres = Entity::find()
        .filter(Column::Année_publication.gt(1900))
        .order_by_asc(Column::Titre)
        .all(&db)
        .await?;

    println!("Livres trouvés: {}", livres.len());
    for livre in livres {
        println!("{} par {} ({})", livre.titre, livre.auteur, livre.année_publication);
    }

    Ok(())
}
```
## Exemple d'application complète mise à jour : Gestionnaire de tâches
Voici une version mise à jour de l'application CLI de gestion de tâches utilisant SQLite.
``` rust
use rusqlite::{params, Connection, Result};
use std::io::{self, Write};
use std::path::Path;

#[derive(Debug)]
struct Tâche {
    id: Option<i32>,
    titre: String,
    terminée: bool,
}

fn créer_base_de_données() -> Result<Connection> {
    let chemin_db = Path::new("tâches.db3");
    let conn = Connection::open(chemin_db)?;

    conn.execute(
        "CREATE TABLE IF NOT EXISTS tâches (
            id INTEGER PRIMARY KEY,
            titre TEXT NOT NULL,
            terminée BOOLEAN NOT NULL DEFAULT 0
        )",
        [],
    )?;

    Ok(conn)
}

fn ajouter_tâche(conn: &Connection, titre: &str) -> Result<i32> {
    conn.execute(
        "INSERT INTO tâches (titre, terminée) VALUES (?1, 0)",
        params![titre],
    )?;

    Ok(conn.last_insert_rowid() as i32)
}

fn marquer_comme_terminée(conn: &Connection, id: i32) -> Result<usize> {
    conn.execute(
        "UPDATE tâches SET terminée = 1 WHERE id = ?1",
        params![id],
    )
}

fn supprimer_tâche(conn: &Connection, id: i32) -> Result<usize> {
    conn.execute(
        "DELETE FROM tâches WHERE id = ?1",
        params![id],
    )
}

fn lister_tâches(conn: &Connection) -> Result<Vec<Tâche>> {
    let mut stmt = conn.prepare("SELECT id, titre, terminée FROM tâches ORDER BY id")?;

    let iter_tâches = stmt.query_map([], |row| {
        Ok(Tâche {
            id: Some(row.get(0)?),
            titre: row.get(1)?,
            terminée: row.get(2)?,
        })
    })?;

    // Utilisation de collect pour simplifier le code
    iter_tâches.collect()
}

fn afficher_menu() {
    println!("\n--- Gestionnaire de Tâches ---");
    println!("1. Ajouter une tâche");
    println!("2. Lister les tâches");
    println!("3. Marquer une tâche comme terminée");
    println!("4. Supprimer une tâche");
    println!("5. Quitter");
    print!("Choisissez une option: ");
    io::stdout().flush().unwrap();
}

fn main() -> Result<()> {
    let conn = créer_base_de_données()?;

    loop {
        afficher_menu();

        let mut choix = String::new();
        io::stdin().read_line(&mut choix).unwrap();

        match choix.trim() {
            "1" => {
                print!("Titre de la tâche: ");
                io::stdout().flush().unwrap();

                let mut titre = String::new();
                io::stdin().read_line(&mut titre).unwrap();

                let id = ajouter_tâche(&conn, titre.trim())?;
                println!("Tâche ajoutée avec l'ID: {}", id);
            },
            "2" => {
                let tâches = lister_tâches(&conn)?;

                if tâches.is_empty() {
                    println!("Aucune tâche trouvée.");
                } else {
                    println!("\nListe des tâches:");
                    for tâche in tâches {
                        let statut = if tâche.terminée { "[✓]" } else { "[ ]" };
                        println!("{}. {} {}", tâche.id.unwrap(), statut, tâche.titre);
                    }
                }
            },
            "3" => {
                print!("ID de la tâche à marquer comme terminée: ");
                io::stdout().flush().unwrap();

                let mut id_str = String::new();
                io::stdin().read_line(&mut id_str).unwrap();

                if let Ok(id) = id_str.trim().parse::<i32>() {
                    match marquer_comme_terminée(&conn, id) {
                        Ok(n) if n > 0 => println!("Tâche marquée comme terminée."),
                        _ => println!("Tâche non trouvée."),
                    }
                } else {
                    println!("ID invalide.");
                }
            },
            "4" => {
                print!("ID de la tâche à supprimer: ");
                io::stdout().flush().unwrap();

                let mut id_str = String::new();
                io::stdin().read_line(&mut id_str).unwrap();

                if let Ok(id) = id_str.trim().parse::<i32>() {
                    match supprimer_tâche(&conn, id) {
                        Ok(n) if n > 0 => println!("Tâche supprimée."),
                        _ => println!("Tâche non trouvée."),
                    }
                } else {
                    println!("ID invalide.");
                }
            },
            "5" => {
                println!("Au revoir!");
                break;
            },
            _ => println!("Option invalide, veuillez réessayer."),
        }
    }

    Ok(())
}
```
## Conclusion
L'écosystème de base de données Rust continue de mûrir avec la Rust 2024 Edition, offrant des améliorations significatives en termes de performances, de sécurité et d'ergonomie. Les bibliothèques ont été mises à jour pour exploiter pleinement les nouvelles fonctionnalités du langage, tout en maintenant les garanties de sécurité mémoire de Rust.
Les principales améliorations comprennent:
1. Des APIs plus ergonomiques pour travailler avec les résultats de requêtes
2. Une meilleure gestion des erreurs
3. Des optimisations de performances
4. Une intégration plus fluide avec l'écosystème Rust moderne

Que vous préfériez une approche de bas niveau avec des requêtes SQL brutes, ou une abstraction de haut niveau avec un ORM, l'écosystème Rust 2024 pour les bases de données offre des solutions robustes et performantes.
Comme toujours, choisissez la bibliothèque qui convient le mieux à votre cas d'utilisation spécifique, en tenant compte de facteurs tels que les performances, la complexité du schéma et le type de base de données que vous utilisez.

⏭️ [Génération de bindings](/III-avance/14-generation-bindings.md) - Création d'interfaces pour d'autres langages
