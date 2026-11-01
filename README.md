# Commande Express

Projet réalisé dans le cadre du cours J2EE avancé et Spring Boot (EMSI), enseigné par M. KHALIL Salim.

Commande Express est une mini plateforme e-commerce basée sur une architecture microservices. Le client passe une commande, le stock est décrémenté automatiquement à la confirmation, et les produits dont le stock passe sous un seuil d'alerte sont signalés « en rupture proche » dans le catalogue.

L'unité de traitement est la **commande**, dont le cycle de vie est : `EN_ATTENTE_STOCK` → `CONFIRMEE` → `ANNULEE`.

L'application est composée de trois services métier :

- **service-catalogue** : référentiel des produits à la vente
- **service-commande** : passage et cycle de vie des commandes (service principal)
- **service-stock** : suivi des niveaux de stock

Technologies : Java 21, Spring Boot, Maven, Spring Cloud (Eureka, Gateway, OpenFeign), Resilience4j, Kafka.

## Diagramme de classes

```mermaid
classDiagram
    direction LR

    namespace service_catalogue {
        class Product {
            Long id
            String nom
            BigDecimal prix
            String description
            String categorie
            StatutProduit statut
        }
        class StatutProduit {
            <<enumeration>>
            DISPONIBLE
            RUPTURE_PROCHE
        }
    }

    namespace service_commande {
        class Order {
            Long id
            String clientEmail
            LocalDateTime date
            StatutCommande statut
            BigDecimal montantTotal
        }
        class OrderLine {
            Long id
            Long orderId
            Long productId
            int quantite
            BigDecimal prixUnitaire
        }
        class StatutCommande {
            <<enumeration>>
            EN_ATTENTE_STOCK
            CONFIRMEE
            ANNULEE
        }
    }

    namespace service_stock {
        class Stock {
            Long id
            Long productId
            int quantite
            int seuilAlerte
        }
    }

    Order "1" *-- "1..*" OrderLine : contient
    Order --> StatutCommande : statut
    Product --> StatutProduit : statut

    OrderLine ..> Product : référence par productId
    Stock ..> Product : référence par productId
    Order ..> Stock : dépend de
```
## Architecture

```mermaid
flowchart LR
    FE["Frontend"] <--> GW["API Gateway<br/>:8080"]
    GW <--> EU["Service Registry<br/>Eureka Server<br/>:8761"]

    GW --> CAT["service-catalogue<br/>:8081"]
    GW --> STK["service-stock<br/>:8082"]
    GW --> CMD["service-commande<br/>:8083"]

    CAT --- DB1[("PostgreSQL<br/>catalogue_db")]
    STK --- DB2[("PostgreSQL<br/>stock_db")]
    CMD --- DB3[("PostgreSQL<br/>commande_db")]
```

| Application | Rôle | Port |
|---|---|---|
| api-gateway | Point d'entrée unique, routage | 8080 |
| eureka-server | Registre des services | 8761 |
| service-catalogue | Référentiel des produits | 8081 |
| service-stock | Niveaux de stock | 8082 |
| service-commande | Cycle de vie des commandes | 8083 |
