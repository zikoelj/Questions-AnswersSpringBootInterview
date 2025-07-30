# 📚 FAQ Spring Boot - Questions/Réponses

## 🔹 Spring Boot – Cœur & Configuration

### 1. Qu'est-ce que Spring Boot et quelle est la différence avec le Spring Framework ?
**Réponse :**  
Spring Boot est une surcouche de Spring Framework qui simplifie le développement en fournissant :  

✅ **Auto-configuration** (plus besoin de configurer manuellement Tomcat, les beans, etc.)  
✅ **Serveur embarqué** (Tomcat/Jetty inclus par défaut)  
✅ **Starters** (dépendances prédéfinies pour éviter les conflits de versions)  
✅ **Réduction du boilerplate** (conventions over configuration)  

**Différence claire :**  
- **Spring** → Configuration manuelle nécessaire (XML, JavaConfig)  
- **Spring Boot** → Configuration automatique basée sur les dépendances  

---

### 2. Qu'est-ce que l'auto-configuration et comment cela fonctionne-t-il en interne ?
**Réponse :**  
**Définition :**  
Mécanisme qui configure automatiquement les composants Spring (DataSource, MVC, JPA, etc.) en se basant sur les dépendances du projet.  

**Exemple :**  
Si vous ajoutez `spring-boot-starter-data-jpa`, Spring Boot configure automatiquement :
- Hibernate  
- Une DataSource par défaut  

**Fonctionnement interne :**  
Utilise des annotations conditionnelles comme :  
- `@ConditionalOnClass` (si une classe est présente dans le classpath)  
- `@ConditionalOnMissingBean` (si aucun bean équivalent n'est déjà défini)  

Les configurations sont définies dans :  
`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`  

**Exemple concret :**  
Avec H2 (BDD embarquée) + JPA, Spring Boot :  
1. Détecte ces dépendances  
2. Configure automatiquement :  
   - Une `DataSource` en mémoire  
   - Un `EntityManager`  
   - La création des tables via `spring.jpa.hibernate.ddl-auto=update`  

---

### 3. À quoi sert l'annotation @SpringBootApplication ?
**Réponse :**  
`@SpringBootApplication` combine 3 annotations en une :  

1. **`@Configuration`** → Déclare des beans  
2. **`@EnableAutoConfiguration`** → Active la config automatique (ex: Tomcat si `spring-boot-starter-web` est présent)  
3. **`@ComponentScan`** → Scanne les composants (`@Service`, `@Controller`, etc.) dans le package courant  

**Résultat :**  
Simplifie le démarrage d'une application Spring Boot sans configuration manuelle.  

---

### 4. Que sont les Spring Boot starters ?
**Réponse :**  
Les Spring Boot Starters sont des ensembles de dépendances pré-configurées pour des fonctionnalités spécifiques (ex : web, data, security).  

**Key Points :**  
- **Packages de dépendances** : Groupent toutes les librairies nécessaires pour un usage particulier (ex: `spring-boot-starter-web` inclut Tomcat + Spring MVC)  
- **Évite les conflits de versions** : Les dépendances sont testées pour être compatibles entre elles  
- **Simplifie le pom.xml/build.gradle** : Une seule dépendance starter remplace plusieurs déclarations manuelles  

**Exemple :**  
`spring-boot-starter-data-jpa` = Hibernate + Spring Data JPA + DataSource  

---

### 5. Différence entre application.properties et application.yml ?
**Réponse :**  
**Même rôle :** Configuration centralisée (BDD, ports, logs, etc.)  

**Différence clé :**  
- **`.properties`** : Format clé-valeur simple  
  ```properties
  server.port=8080
  spring.datasource.url=jdbc:h2:mem:testdb
  ```
- **`.yml`**: Structure hiérarchique (indentation), plus lisible pour les configs complexes
	```yml
	server:
	  port: 8080
	spring:
	  datasource:
		url: jdbc:h2:mem:testdb
	```
**Avantage de YAML :**
	- Évite la répétition de préfixes (ex: spring.datasource)
	- Supporte les listes/objets imbriqués (utile pour Spring Cloud, Kubernetes)

---

### 6. Comment Spring Boot réduit-il le code boilerplate ?
**Réponse :**  
Spring Boot réduit le boilerplate (code répétitif) grâce à :

**1. Auto-configuration :**  
- Configure automatiquement les composants standards (ex: DataSource, MVC, JPA) en détectant les dépendances  
- *Exemple* : Pas besoin de configurer Tomcat manuellement avec `spring-boot-starter-web`  

**2. Annotations "magiques" :**  
- `@SpringBootApplication` → Combine 3 annotations en une  
- `@RestController` → Intègre `@Controller` + `@ResponseBody`  
- `@Data` (Lombok) → Génère getters/setters/constructeurs automatiquement  

**3. Starters :**  
- Éliminent la gestion manuelle des dépendances  
- *Exemple* : `spring-boot-starter-data-jpa` inclut Hibernate + Spring Data  

**4. Serveur embarqué :**  
- Plus besoin de déployer sur un Tomcat externe  

---

## 🔹 Injection de dépendances & gestion des beans

### 7. Qu'est-ce que l'injection de dépendances dans Spring Boot ?
**Réponse :**  
L'injection de dépendances (DI) dans Spring Boot est un mécanisme où :  

**Fonctionnement :**  
- Le conteneur Spring (IoC Container) crée et gère les objets (beans)  
- Injection automatique des dépendances entre beans (au lieu de `new` manuel)  

**Exemple :**  
```java
@Service
public class UserService {
    @Autowired  // Injection automatique
    private UserRepository userRepo;
    // Plus besoin de 'new UserRepository()'
}
```
**Points clés :**

**1. Principe IoC (Inversion of Control) :**  
- Spring gère les dépendances à votre place  

**2. Types d'injection :**  
- Par champ (`@Autowired` sur attribut)  
- Par constructeur (✔️ recommandé)  
- Par setter  

**3. Avantages :**  
- Réduit le couplage entre classes  
- Facilite les tests (mocks injectables)  

---

### 8. Différence entre @Component, @Service et @Repository ?
**Réponse :**  
`@Component`, `@Service` et `@Repository` sont des stéréotypes Spring pour déclarer des beans, mais avec des rôles sémantiques distincts :

#### 1. `@Component`
- Annotation générique pour tout bean géré par Spring
- **Exemple** (classe utilitaire) :
```java
@Component
public class LoggerUtil { ... }
```
#### 2 `@Service`
- Spécialisation de `@Component` pour la couche métier
- **Exemple** (business logic) :
```java
@Service
public class UserService {
    // Logique métier (calculs, règles de gestion)
}
```
#### 3 `@Repository`
- Spécialisation de `@Component` pour la couche DAO
- Bonus : Traduction automatique des exceptions SQL en DataAccessException
- **Exemple** :
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> { ... }
```
**En pratique :**
- Techniquement interchangeables (toutes créent des beans)
- Utilisez-les pour clarifier l'intention de votre code

---

### 9. Comment les beans sont-ils gérés par le conteneur Spring ?
**Réponse :**  
Les beans sont gérés par le conteneur Spring (IoC Container) via un cycle de vie précis :

1. **Création**  
   - Instanciation via :  
     - Annotations (`@Component`, `@Service`, etc.)  
     - Fichiers de config (`@Bean` dans une classe `@Configuration`)  
     - Appel du constructeur  

2. **Injection des dépendances**  
   - Le conteneur injecte les beans nécessaires (via `@Autowired`, constructeur, ou setter)  

3. **Initialisation**  
   - Si le bean implémente InitializingBean ou utilise `@PostConstruct`, une méthode d’initialisation est appelée.

4. **Utilisation**  
   - Le bean est disponible pour d'autres composants  

5. **Destruction**  
   - Si le bean implémente DisposableBean ou utilise `@PreDestroy`, une méthode de nettoyage est appelée avant la destruction.  
   - Le conteneur libère le bean lors de l'arrêt de l'application  
   
- **Exemple concret** :
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepo; // Injection

    @PostConstruct
    public void init() { System.out.println("Bean prêt !"); }

    @PreDestroy
    public void cleanup() { System.out.println("Nettoyage..."); }
}

```

### 10. Différence entre @Bean et @Component ?
**Réponse :**  

| **@Bean**                          | **@Component**                     |
|------------------------------------|-----------------------------------|
| Utilisé dans une classe annotée `@Configuration` | Utilisé directement sur une classe |
| Déclare un bean via une méthode (ex : configurer un bean externe comme `DataSource`) | Transforme la classe elle-même en bean (ex : `@Service`, `@Repository`) |
| Flexible : Permet de personnaliser l'instanciation (ex : `@Bean public DataSource ds() { ... }`) | Automatique : Spring crée le bean via le constructeur par défaut |
| Cas d'usage : Intégration de librairies externes (ex : Redis, Kafka) | Cas d'usage : Composants métiers (services, repositories) |

**Exemple :**
```java
@Configuration  
public class AppConfig {  
    @Bean // Bean personnalisé  
    public DataSource dataSource() {  
        return new HikariDataSource();  
    }  
}  

@Component // Bean automatique  
public class UserService { ... }
```
**Résumé :**

- `@Bean` → Configuration manuelle dans une classe @Configuration.
- `@Component` → Détection automatique par Spring.

---

### 11. À quoi servent @Autowired et @Qualifier ?
**Réponse :**

#### @Autowired
- Permet l'injection automatique de dépendances par Spring
- Peut être utilisé sur :
  - Les champs (field injection)
  - Les constructeurs (✔️ recommandé)
  - Les setters

#### @Qualifier
- Résout les ambiguïtés quand plusieurs beans du même type existent
- Permet de spécifier quel bean exactement doit être injecté

#### Exemple combiné :
```java
@Component
@Qualifier("pdfReport")
public class PdfReportGenerator implements ReportGenerator {...}

@Component
@Qualifier("excelReport")
public class ExcelReportGenerator implements ReportGenerator {...}

@Service
public class ReportService {
    @Autowired
    @Qualifier("pdfReport")  // Injection spécifique
    private ReportGenerator reportGenerator;
}
```
**Bonnes pratiques :**
- Préférez l'injection par constructeur pour les dépendances obligatoires
- Utilisez @Qualifier pour les cas d'ambiguïté
- Pour les cas complexes, envisagez @Primary à la place

---

### 12. Quels sont les différents scopes dans Spring ?
**Réponse :**

#### Scopes Spring Principaux

**Singleton (par défaut) :**
- Une seule instance partagée dans tout l'application
- Idéal pour :
  - Services sans état (stateless)
  - Composants coûteux à instancier
- Exemple : `@Service` classique

**Prototype :**
- Nouvelle instance à chaque injection/demande
- Idéal pour :
  - Objets avec état (stateful)
  - Quand la mutabilité est nécessaire
- Exemple : `@Scope("prototype")`

#### Scopes Web (disponibles dans les applications web) :
- **Request** : Une instance par requête HTTP
- **Session** : Une instance par session utilisateur
- **Application** : Une instance par contexte Servlet
- **WebSocket** : Une instance par connexion WebSocket

#### Exemple d'utilisation :
```java
@Service
@Scope("prototype") // Crée une nouvelle instance à chaque injection
public class PrototypeService { ... }

@RestController
public class MyController {
    @Autowired // Toujours la même instance
    private SingletonService singletonService;
    
    @Autowired // Nouvelle instance à chaque injection
    private PrototypeService prototypeService;
}
```
**Bonnes Pratiques :**
- Le singleton est le scope le plus performant (moins d'instances)
- Le prototype est plus sûr pour les objets mutables
- Pour les applications web, privilégiez les scopes web quand approprié

---

## 🔹 API REST & couche Web

### 13. Différences entre @RestController et @Controller ?
**Réponse :**

| **@Controller**                     | **@RestController**                |
|-------------------------------------|------------------------------------|
| Pour applications MVC traditionnelles (backend + templates) | Pour applications REST/API (backend séparé du front) |
| Retourne des noms de vues (HTML)    | Retourne directement des données (JSON/XML) |
| Utilise des moteurs de templates : Thymeleaf, JSP, Freemarker | Communique via HTTP avec React/Angular/autres clients |
| Nécessite `@ResponseBody` pour retourner du JSON | Combine `@Controller` + `@ResponseBody` automatiquement |

#### Exemples concrets :
**Avec @Controller (MVC) :**
```java
@Controller
public class WebController {
    @GetMapping("/home")
    public String home(Model model) {
        model.addAttribute("message", "Bonjour");
        return "home"; // Renvoie home.html (Thymeleaf/JSP)
    }
}
```
***Avec @RestController (API) :***
```java
@RestController
public class ApiController {
    @GetMapping("/api/data")
    public Data getData() {
        return new Data(...); // Renvoyé directement en JSON
    }
}
```
**Bon à savoir :**

`@RestController` = `@Controller` + `@ResponseBody` sur chaque méthode

Pour une application hybride (à la fois MVC et API), vous pouvez utiliser les deux annotations dans le même projet

---

### 14. Quand utiliser @RequestMapping, @GetMapping, @PostMapping, etc. ?
**Réponse :**

| Annotation       | Equivalent à                     | Usage typique                              | Méthode HTTP |
|------------------|----------------------------------|--------------------------------------------|-------------|
| `@RequestMapping` | La plus générique                | `@RequestMapping("/path")`                 | Toutes (GET par défaut) |
| `@GetMapping`    | `@RequestMapping(method=GET)`    | Lire des données                           | GET         |
| `@PostMapping`   | `@RequestMapping(method=POST)`   | Créer des ressources                       | POST        |
| `@PutMapping`    | `@RequestMapping(method=PUT)`    | Mettre à jour complètement une ressource   | PUT         |
| `@DeleteMapping` | `@RequestMapping(method=DELETE)` | Supprimer une ressource                    | DELETE      |
| `@PatchMapping`  | `@RequestMapping(method=PATCH)`  | Mettre à jour partiellement                | PATCH       |

#### Bonnes pratiques :
**Préférez les annotations spécifiques** (`@GetMapping`, `@PostMapping`) pour plus de clarté  
**Utilisez `@RequestMapping` seulement pour :**
   - Définir un préfixe commun à plusieurs méthodes  
   - Des cas nécessitant plusieurs méthodes HTTP  

**Exemple d'usage :**
```java
@RestController
@RequestMapping("/api/users")  // Préfixe commun
public class UserController {

    @GetMapping  // GET /api/users
    public List<User> getAllUsers() { ... }

    @PostMapping // POST /api/users
    public User createUser(@RequestBody User user) { ... }
	
	@RequestMapping(value = "/special", method = {GET, POST}) // Cas mixte
    public String specialHandler() { ... }
}

```

### 15. Comment gérer les paramètres de requête et les variables de chemin ?
**Réponse :**

### 1. Variables de chemin (Path Variables)
```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    // Ex: /users/42 → id = 42
}
```
### 2. Paramètres de requête (Query Parameters)
```java
@GetMapping("/search")
public List<User> searchUsers(@RequestParam String name) {
    // Ex: /search?name=John → name = "John"
}
```
### 3. Corps de requête (Request Body)

```java
@PostMapping("/users")
public User createUser(@RequestBody User user) {
    // Reçoit un objet JSON dans le body
}
```
***Règles clés :***

- `@RequestBody` : Uniquement pour le corps des requêtes (POST/PUT)
- `@RequestParam` : Paramètres après le ? dans l'URL
- `@PathVariable` : Segments du chemin d'URL

***Exemple complet :***

```java
@GetMapping("/products/{category}")
public List<Product> getProducts(
    @PathVariable String category,      // /products/electronics
    @RequestParam(required = false) String sort, // ?sort=price
    @RequestParam(defaultValue = "10") int limit // ?limit=20
) { ... }
```
**Bonnes pratiques :**
- `required = false` pour les paramètres optionnels
- `defaultValue` pour les valeurs par défaut
- Validez avec `@Valid` + annotations de validation (ex: `@Size`, `@NotNull`)

**Annotations de validation utiles :**

- `@NotNull`, `@Size(min=1, max=10)`

- `@Email`, `@Pattern(regexp = "...")`

- `@Min`, `@Max` pour les nombres

---

### 16. Comment retourner des données JSON avec Spring Boot ?
**Réponse :**

#### Fonctionnement automatique :
- Grâce à **Jackson** (intégré par défaut via `spring-boot-starter-web`)
- Conversion automatique des objets Java ↔ JSON
- Nécessite soit :
  - Un contrôleur annoté `@RestController` (contient `@ResponseBody` implicitement)
  - Ou l'annotation `@ResponseBody` sur une méthode

#### Exemple de base :
```java
@RestController
public class UserController {
    
    @GetMapping("/user")
    public User getUser() {
        return new User("John", 30); // → {"name":"John","age":30}
    }
}
```
#### Personnalisation du JSON (annotations Jackson) :
```java
public class User {
    @JsonProperty("user_name")  // Renomme le champ dans le JSON
    private String name;
    
    @JsonIgnore  // Exclut le champ du JSON
    private String password;
    
    @JsonFormat(pattern="dd/MM/yyyy")  // Format de date
    private LocalDate birthDate;
}
```
#### Bon à savoir :
- Spring Boot supporte aussi XML (si activé via HttpMessageConverter)
- La conversion se fait via HttpMessageConverters (Jackson par défaut pour JSON)
- Configuration possible dans application.properties :
```java
spring.jackson.date-format=yyyy-MM-dd
spring.jackson.serialization.WRITE_DATES_AS_TIMESTAMPS=false
```
#### Gestion avancée :
- Créer des DTOs spécifiques pour le JSON
- Utiliser @JsonView pour contrôler les champs exposés
- Gérer les collections : List<User> → tableau JSON automatique

---

### 17. Validation des données dans Spring Boot
#### Annotation clé : `@Valid`

- Active la validation sur un objet reçu dans le corps de la requête

#### Exemple :

```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
    // Le user est validé automatiquement avant d'arriver ici
}
```
- Annotations de validation standard (jakarta.validation)

```java
public class User {
    @NotBlank
    private String name;
    
    @Email
    private String email;
    
    @Min(18)
    private int age;
}
```
***Gestion des erreurs***

- Spring retourne automatiquement un 400 Bad Request si la validation échoue
- Format d'erreur standard :

```json
{
  "timestamp": "2023-05-20T12:34:56",
  "status": 400,
  "errors": ["Le champ email doit être une adresse valide"]
}
```
***Bonnes pratiques :***

- Combinez avec `@RestControllerAdvice` pour personnaliser les messages d'erreur
- Utilisez des DTOs spécifiques plutôt que vos entités JPA pour la validation
- Pour les validations complexes, implémentez `ConstraintValidator`

---
## 🔹 Gestion des exceptions

### 18. Utilisation de @ControllerAdvice pour la gestion globale des exceptions
**Rôle principal :**

- Centralise le traitement des exceptions pour tous les contrôleurs
- Évite la duplication du code de gestion d'erreurs 

I**mplémentation de base :**

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse("NOT_FOUND", ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleAllExceptions(Exception ex) {
        ErrorResponse error = new ErrorResponse("INTERNAL_ERROR", "Une erreur est survenue");
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```
**Composants clés :**

- `@ExceptionHandler` : Définit quelle méthode gère quel type d'exception
- `ResponseEntity` : Permet de contrôler le statut HTTP et le corps de la réponse
- Classe d'erreur personnalisée (ici ErrorResponse)

**Avantages :**

- Retourne des réponses d'erreur cohérentes dans tout l'API
- Séparation claire entre la logique métier et la gestion des erreurs
- Personnalisation fine des codes HTTP et messages d'erreur

**Exemple de réponse JSON :**
```json
{
  "code": "NOT_FOUND",
  "message": "L'utilisateur demandé n'existe pas"
}
```

---

### 19. Comment utiliser @ControllerAdvice pour la gestion globale des exceptions ?

- L’annotation `@ExceptionHandler` est utilisée dans Spring pour gérer des exceptions spécifiques de manière centralisée ou locale.
- Elle peut être placée :
	- dans un contrôleur (`@RestController`) pour capturer les exceptions levées par ses propres méthodes, ou
	- dans une classe annotée avec `@ControllerAdvice` ou `@RestControllerAdvice` pour une gestion globale des exceptions dans toute l’application.

- Par exemple :
```java
@ExceptionHandler(ResourceNotFoundException.class)
public ResponseEntity<String> handleResourceNotFound(ResourceNotFoundException ex) {
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
}
```
- Quand une `ResourceNotFoundException` est levée, Spring va chercher automatiquement une méthode avec `@ExceptionHandler(ResourceNotFoundException.class)` pour gérer cette exception.

**Points importants à dire à l’oral :**
- Permet de mapper une exception vers une réponse HTTP personnalisée.
- Peut être utilisée dans une méthode d’un contrôleur ou dans une classe globale annotée avec `@RestControllerAdvice`.
- La méthode peut retourner un `ResponseEntity`, une vue, ou même un objet JSON selon les besoins.

---

### 20. Quelles sont les bonnes pratiques pour créer des exceptions personnalisées dans une application Spring Boot ?

- Pour créer des exceptions personnalisées dans une application Spring Boot, voici les bonnes pratiques :

1. Créer une classe d’exception dédiée, qui hérite de `RuntimeException` ou d’une autre exception adaptée.

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```
2. Organiser ces classes dans un package spécifique, comme `com.example.exception`, pour plus de clarté.

3. Gérer ces exceptions avec `@ExceptionHandler`, soit :

	- localement dans un contrôleur ;

	- soit globalement dans une classe annotée avec `@RestControllerAdvice`.

4. Toujours retourner une réponse claire (statut HTTP + message d’erreur) pour améliorer l’expérience client/API.

----

