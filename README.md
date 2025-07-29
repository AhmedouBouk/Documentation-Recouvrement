# Documentation Technique - Système de Gestion des Dossiers de Recouvrement BNM

## 📋 Table des Matières

1. [Vue d'ensemble du Projet](#vue-densemble-du-projet)
2. [Architecture du Projet](#architecture-du-projet)
3. [Technologies Utilisées](#technologies-utilisées)
4. [Modélisation de la Base de Données](#modélisation-de-la-base-de-données)
5. [Système de Sécurité](#système-de-sécurité)
6. [API REST](#api-rest)
7. [Frontend Angular](#frontend-angular)
8. [Gestion des Erreurs](#gestion-des-erreurs)
9. [Tests](#tests)
10. [Guide d'Exécution](#guide-dexécution)
11. [Erreurs Fréquentes](#erreurs-fréquentes)
12. [Annexes](#annexes)

---

## 🎯 Vue d'ensemble du Projet

### Objectif
Le projet de développement de l'application de gestion des dossiers de recouvrement a pour but principal de **moderniser et d'optimiser le processus de recouvrement des créances au sein de la BNM**.

### Fonctionnalités Principales
- ✅ Gestion complète des dossiers de recouvrement
- ✅ Système d'authentification et d'autorisation basé sur les rôles
- ✅ Upload et gestion de fichiers multiples (PDF, images)
- ✅ Workflow de validation multi-étapes
- ✅ Notifications automatiques par email
- ✅ Import/Export CSV
- ✅ Génération de rapports PDF
- ✅ Historique des actions utilisateurs
- ✅ Interface responsive et moderne

---

## 🏗️ Architecture du Projet

### Architecture Générale
```
┌─────────────────┐    HTTP/REST    ┌─────────────────┐    JPA/Hibernate    ┌─────────────────┐
│                 │    (Port 4200)  │                 │    (Port 5432)      │                 │
│  Frontend       │◄───────────────►│  Backend        │◄───────────────────►│  PostgreSQL     │
│  Angular 15     │    JWT Auth     │  Spring Boot    │                     │  Database       │
│                 │                 │  Java 17        │                     │                 │
└─────────────────┘                 └─────────────────┘                     └─────────────────┘
```

### Pattern Architectural
- **Architecture 3-tiers** : Présentation (Angular) → Logique Métier (Spring Boot) → Données (PostgreSQL)
- **Séparation des responsabilités** : Frontend/Backend découplés
- **API REST** : Communication stateless avec authentification JWT
- **Microservices-ready** : Structure modulaire permettant l'évolution vers des microservices

### Relations entre Modules

#### Backend → Database
- **ORM** : JPA/Hibernate pour la persistance
- **Migrations** : Flyway pour la gestion des versions de schéma
- **Connexion** : Pool de connexions configuré via Spring Boot

#### Frontend → Backend
- **Protocole** : HTTP/HTTPS avec API REST
- **Authentification** : JWT Bearer Token
- **Format** : JSON pour les échanges de données
- **CORS** : Configuration pour autoriser localhost:4200

#### Protocoles de Communication
- **HTTP Methods** : GET, POST, PUT, DELETE, OPTIONS
- **Content-Type** : application/json, multipart/form-data (upload)
- **Headers** : Authorization (Bearer token), Content-Type
- **Status Codes** : 200, 201, 400, 401, 403, 404, 500

---

## 🛠️ Technologies Utilisées

### Backend (Spring Boot 3.3.5)
| Technologie | Version | Usage |
|-------------|---------|-------|
| **Java** | 17 | Langage principal |
| **Spring Boot** | 3.3.5 | Framework principal |
| **Spring Security** | 6.x | Authentification/Autorisation |
| **Spring Data JPA** | 3.x | Persistance des données |
| **PostgreSQL** | Latest | Base de données |
| **JWT (jjwt)** | 0.11.5 | Gestion des tokens |
| **Apache PDFBox** | 2.0.27 | Génération PDF |
| **iTextPDF** | 5.5.13.3 | Manipulation PDF |
| **OpenCSV** | 2.3 | Import/Export CSV |
| **Spring Mail** | 3.x | Envoi d'emails |
| **Lombok** | Latest | Réduction du code boilerplate |

### Frontend (Angular 15.2.0)
| Technologie | Version | Usage |
|-------------|---------|-------|
| **Angular** | 15.2.0 | Framework frontend |
| **TypeScript** | 4.9.0 | Langage de développement |
| **Angular Material** | 15.2.9 | Composants UI |
| **RxJS** | 7.8.0 | Programmation réactive |
| **FontAwesome** | 6.7.1 | Icônes |
| **jsPDF** | 3.0.1 | Génération PDF côté client |
| **html2pdf.js** | 0.10.3 | Conversion HTML vers PDF |
| **ng2-pdf-viewer** | 10.4.0 | Visualisation PDF |

### Base de Données
- **PostgreSQL** : Base de données principale
- **Host** : localhost:5432
- **Database** : testman
- **Credentials** : postgres/12345678

---

## 🗄️ Modélisation de la Base de Données

### Entités Principales

#### 1. DossierRecouvrement (Entité Centrale)
```sql
CREATE TABLE dossier_recouvrement (
    id BIGSERIAL PRIMARY KEY,
    account_number VARCHAR(255) NOT NULL,
    engagement_total DOUBLE PRECISION,
    montant_principal DOUBLE PRECISION,
    interet_contractuel DOUBLE PRECISION,
    interet_retard DOUBLE PRECISION,
    nature TEXT,
    agence_ouverture_compte VARCHAR(255),
    references_checks VARCHAR(255),
    references_credits VARCHAR(255),
    references_cautions VARCHAR(255),
    references_lc VARCHAR(255),
    provision DOUBLE PRECISION,
    interets_reserves DOUBLE PRECISION,
    status VARCHAR(50) DEFAULT 'EN_COURS',
    etat_validation VARCHAR(50) DEFAULT 'INITIALE',
    garanties_valeur VARCHAR(255),
    credits_file VARCHAR(255),
    cautions_file VARCHAR(255),
    lc_file VARCHAR(255),
    cheque_file VARCHAR(255),
    compte_id BIGINT,
    date_creation TIMESTAMP,
    date_archivage TIMESTAMP,
    FOREIGN KEY (compte_id) REFERENCES comptes(id)
);
```

#### 2. User (Gestion des Utilisateurs)
```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    first_name VARCHAR(255),
    last_name VARCHAR(255),
    user_type VARCHAR(50),
    first_login BOOLEAN DEFAULT TRUE,
    role_id BIGINT,
    agence_id BIGINT,
    FOREIGN KEY (role_id) REFERENCES roles(id),
    FOREIGN KEY (agence_id) REFERENCES agences(id)
);
```

#### 3. Role & Permission (Système d'Autorisation)
```sql
CREATE TABLE roles (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) UNIQUE NOT NULL,
    active BOOLEAN DEFAULT TRUE
);

CREATE TABLE permissions (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) UNIQUE NOT NULL,
    description TEXT
);

CREATE TABLE role_permissions (
    role_id BIGINT,
    permission_id BIGINT,
    PRIMARY KEY (role_id, permission_id),
    FOREIGN KEY (role_id) REFERENCES roles(id),
    FOREIGN KEY (permission_id) REFERENCES permissions(id)
);
```

### Diagramme ERD (Relations)
```
Users ──┐
         ├─► DossierRecouvrement ──┬─► ChequeFiles
         │                        ├─► CautionFiles  
Roles ───┤                        ├─► CreditFiles
         │                        ├─► LcFiles
Agences ─┘                        ├─► Garanties
                                  ├─► Comments
Clients ────► Comptes ────────────┘  └─► Notifications
```

### Statuts et Énumérations

#### Status du Dossier
```java
public enum Status {
    EN_COURS, ARCHIVEE, CLIENT_RELANCE, PROTOCOLE_SIGNE,
    RECOUVREMENT_AMIABLE, EN_JUSTICE, JUGEMENT_RENDU,
    ADJUDICATION_EN_COURS, CLOTURE, ARCHIVE, REJETE
}
```

#### État de Validation
```java
public enum EtatValidation {
    INITIALE, COMPLET, VALIDE, NON_VALIDE
}
```

---

## 🔐 Système de Sécurité

### Méthodes d'Authentification

#### 1. JWT (JSON Web Token)
- **Algorithme** : HS256 (HMAC SHA-256)
- **Secret Key** : `pFR/4gKqQWIdGIO+dE37DCthVlCbcI1bGtprGDW18+M=`
- **Expiration** : 10 heures (36000 secondes)
- **Claims inclus** :
  - `userId` : ID de l'utilisateur
  - `role` : Nom du rôle
  - `permissions` : Liste des permissions
  - `userType` : Type d'utilisateur

#### 2. Configuration Spring Security
```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) {
        return http
            .csrf(csrf -> csrf.disable())
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/auth/**").permitAll()
                .requestMatchers("/clients/**").hasAuthority("READ_CLIENT")
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }
}
```

### Système de Permissions

#### Permissions par Entité
```java
// CLIENTS
READ_CLIENT, CREATE_CLIENT, UPDATE_CLIENT, DELETE_CLIENT, IMPORT_CLIENT

// COMPTES  
READ_COMPTE, CREATE_COMPTE, UPDATE_COMPTE, DELETE_COMPTE

// CREDITS
READ_CREDIT, CREATE_CREDIT, UPDATE_CREDIT, DELETE_CREDIT

// DOSSIERS
READ_DOSSIER, CREATE_DOSSIER, UPDATE_DOSSIER, DELETE_DOSSIER

// FICHIERS
UPLOAD_CHEQUE_FILE, DOWNLOAD_CHEQUE_FILE, DELETE_CHEQUE_FILE
UPLOAD_CAUTION_FILE, DOWNLOAD_CAUTION_FILE, DELETE_CAUTION_FILE
UPLOAD_CREDIT_FILE, DOWNLOAD_CREDIT_FILE, DELETE_CREDIT_FILE
UPLOAD_LC_FILE, DOWNLOAD_LC_FILE, DELETE_LC_FILE

// ADMIN
CREATE_ROLE, READ_ROLE, UPDATE_ROLE, DELETE_ROLE
CREATE_USER, READ_USER, UPDATE_USER, DELETE_USER
```

### Politique de Sécurité

#### CORS (Cross-Origin Resource Sharing)
```java
@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration configuration = new CorsConfiguration();
    configuration.setAllowedOrigins(List.of("http://localhost:4200"));
    configuration.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));
    configuration.setAllowedHeaders(List.of("*"));
    configuration.setAllowCredentials(true);
    configuration.setExposedHeaders(List.of("Authorization"));
    return source -> configuration;
}
```

#### Protection CSRF
- **Désactivé** pour les API REST (stateless)
- **Justification** : Utilisation de JWT au lieu de cookies de session

#### Protection XSS
- **Validation** : Validation des entrées côté backend
- **Échappement** : Échappement automatique des données dans les templates Angular
- **Headers** : Content-Type strict

---

## 🌐 API REST

### Endpoints Principaux

#### 1. Authentification (`/auth`)
| Méthode | Endpoint | Description | Permissions |
|---------|----------|-------------|-------------|
| POST | `/auth/login` | Connexion utilisateur | Public |
| POST | `/auth/register` | Inscription utilisateur | Public |
| POST | `/auth/change-password` | Changement de mot de passe | Authentifié |

#### 2. Dossiers de Recouvrement (`/dossiers`)
| Méthode | Endpoint | Description | Permissions |
|---------|----------|-------------|-------------|
| GET | `/dossiers/affichage` | Liste des dossiers | READ_DOSSIER |
| GET | `/dossiers/{id}` | Détails d'un dossier | READ_DOSSIER |
| POST | `/dossiers/create` | Création d'un dossier | CREATE_DOSSIER |
| PUT | `/dossiers/update/{id}` | Modification d'un dossier | UPDATE_DOSSIER |
| DELETE | `/dossiers/delete/{id}` | Suppression d'un dossier | DELETE_DOSSIER |
| GET | `/dossiers/search` | Recherche de dossiers | READ_DOSSIER |
| POST | `/dossiers/{id}/fusionner-complet` | Fusion PDF complète | READ_DOSSIER |

#### 3. Clients (`/clients`)
| Méthode | Endpoint | Description | Permissions |
|---------|----------|-------------|-------------|
| GET | `/clients/Affichage` | Liste des clients | READ_CLIENT |
| GET | `/clients/{nni}` | Détails d'un client | READ_CLIENT |
| POST | `/clients/create` | Création d'un client | CREATE_CLIENT |
| POST | `/clients/import-client` | Import CSV | IMPORT_CLIENT |

#### 4. Administration (`/admin`)
| Méthode | Endpoint | Description | Permissions |
|---------|----------|-------------|-------------|
| GET | `/admin/roles` | Liste des rôles | READ_ROLE |
| POST | `/admin/roles` | Création d'un rôle | CREATE_ROLE |
| PUT | `/admin/roles/{id}` | Modification d'un rôle | UPDATE_ROLE |
| DELETE | `/admin/roles/{id}` | Suppression d'un rôle | DELETE_ROLE |

---

## 💻 Frontend Angular

### Structure des Composants

```
src/app/
├── auth/                    # Authentification
├── admin/                   # Administration
├── pages/                   # Pages métier
├── components/              # Composants réutilisables
├── shared/                  # Services et utilitaires
└── layouts/                 # Layouts
```

### Services Principaux

#### AuthService
```typescript
@Injectable({ providedIn: 'root' })
export class AuthService {
  private readonly apiUrl = `${environment.apiUrl}/auth`;
  
  login(credentials: LoginRequest): Observable<AuthResponse> {
    return this.http.post<AuthResponse>(`${this.apiUrl}/login`, credentials);
  }
  
  hasPermission(permission: string): boolean {
    const token = localStorage.getItem('token');
    if (!token) return false;
    const decoded = jwt_decode(token) as any;
    return decoded.permissions?.includes(permission) || false;
  }
}
```

### Guards de Sécurité

- **AuthGuard** : Vérification authentification
- **RoleGuard** : Vérification des rôles
- **NonAdminGuard** : Accès non-admin
- **FirstLoginGuard** : Changement mot de passe obligatoire

---

## ⚠️ Gestion des Erreurs

### Backend
- **GlobalExceptionHandler** : Gestion centralisée des exceptions
- **Validation** : Bean Validation avec messages personnalisés
- **Logging** : SLF4J pour traçabilité

### Frontend
- **HttpErrorInterceptor** : Interception des erreurs HTTP
- **ErrorHandlingService** : Service centralisé de gestion d'erreurs
- **User Feedback** : Messages d'erreur utilisateur-friendly

---

## 🧪 Tests

### Types de Tests Mis en Place

#### Tests Unitaires
- **Backend** : JUnit 5 + Mockito
- **Frontend** : Jasmine + Karma
- **Couverture** : Services et composants critiques

#### Tests d'Intégration
- **API Tests** : TestContainers avec PostgreSQL
- **End-to-End** : Cypress (à implémenter)

---

## 🚀 Guide d'Exécution

### Prérequis
- **Java 17+**
- **Node.js 16+**
- **PostgreSQL 12+**
- **Maven 3.6+**
- **Angular CLI 15+**

### Exécution Locale

#### 1. Base de Données
```bash
# Créer la base de données
createdb testman
```

#### 2. Backend
```bash
cd dossiers-recouvrement-backend
mvn spring-boot:run
# Serveur disponible sur http://localhost:8080
```

#### 3. Frontend
```bash
cd dossier_recouvrement_frontend
npm install
ng serve
# Application disponible sur http://localhost:4200
```



---

## 🔧 Erreurs Fréquentes

### 1. Erreur 403 Forbidden
**Cause** : Permissions insuffisantes ou token expiré
**Solution** :
- Vérifier les permissions utilisateur
- Renouveler le token JWT
- Vérifier la configuration CORS

### 2. Erreur Upload de Fichiers
**Cause** : Taille de fichier dépassée (limite 2MB)
**Solution** :
- Réduire la taille du fichier
- Modifier `spring.servlet.multipart.max-file-size`

### 3. Erreur de Connexion Base de Données
**Cause** : PostgreSQL non démarré ou mauvaises credentials
**Solution** :
- Vérifier le service PostgreSQL
- Contrôler `application.properties`

### 4. Erreur CORS
**Cause** : Frontend sur un port différent de 4200
**Solution** :
- Modifier `SecurityConfig.java`
- Ajouter le nouveau port dans `allowedOrigins`

---

## 📚 Annexes

### Références Techniques
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Angular Documentation](https://angular.io/docs)
- [JWT.io](https://jwt.io/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

### Contacts Équipe
- **Développeur Principal** : Mohamedou Bouk - 23083@esp.mr
- **Développeur** : Mariem Ejiwene - 23018@esp.mr
- **Développeur** : Babah  - 23064@esp.mr
- **Développeur** : selma - 23042@esp.mr

### Versions et Changelog
- **v1.0.0** : Version initiale avec fonctionnalités de base
- **v1.1.0** : Ajout gestion multi-fichiers
- **v1.2.0** : Workflow de validation amélioré

---

**Dernière mise à jour** : 29 Juillet 2025
**Version de la documentation** : 1.0.0
