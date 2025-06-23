Consuming external APIs in a Spring Boot application involves making HTTP requests to third-party services and processing their responses. Below are the **best practices and methods** to consume external APIs efficiently:

---

## **1. Using `RestTemplate` (Synchronous, Blocking)**
**When to use?**  
- Simple, synchronous calls in non-reactive applications.  
- Legacy Spring applications (though being phased out in favor of `WebClient`).  

### **Example:**
```java
import org.springframework.web.client.RestTemplate;

@Service
public class WeatherService {
    
    private final RestTemplate restTemplate;
    private final String API_URL = "https://api.weatherapi.com/v1/current.json?key={key}&q={city}";

    public WeatherService(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder.build();
    }

    public Weather getWeather(String city, String apiKey) {
        String url = API_URL.replace("{city}", city).replace("{key}", apiKey);
        return restTemplate.getForObject(url, Weather.class);
    }
}
```

### **Pros & Cons:**
✅ Simple to use.  
❌ **Blocking** (not suitable for high concurrency).  
❌ Deprecated in newer Spring versions (replaced by `WebClient`).  

---

## **2. Using `WebClient` (Asynchronous, Non-Blocking)**
**When to use?**  
- Reactive applications (Spring WebFlux).  
- High-performance, non-blocking API calls.  

### **Example:**
```java
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

@Service
public class UserService {
    
    private final WebClient webClient;

    public UserService(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder.baseUrl("https://api.github.com").build();
    }

    public Mono<GitHubUser> fetchGitHubUser(String username) {
        return webClient.get()
                .uri("/users/{username}", username)
                .retrieve()
                .bodyToMono(GitHubUser.class);
    }
}
```

### **Pros & Cons:**
✅ **Non-blocking** (better performance under load).  
✅ Supports **reactive streams** (Mono/Flux).  
❌ Requires understanding of reactive programming.  

---

## **3. Using `Feign Client` (Declarative REST Client)**
**When to use?**  
- Simplifies REST calls with interfaces (like Retrofit).  
- Works well with Spring Cloud (load balancing, retries).  

### **Example:**
#### **Step 1: Add Dependency (`spring-cloud-starter-openfeign`)**  
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

#### **Step 2: Enable Feign Clients**
```java
@SpringBootApplication
@EnableFeignClients
public class MyApp { ... }
```

#### **Step 3: Define a Feign Client Interface**
```java
@FeignClient(name = "jsonplaceholder", url = "https://jsonplaceholder.typicode.com")
public interface JsonPlaceholderClient {
    
    @GetMapping("/posts/{id}")
    Post getPostById(@PathVariable Long id);
}
```

#### **Step 4: Use the Client**
```java
@Service
public class PostService {
    
    private final JsonPlaceholderClient client;

    public PostService(JsonPlaceholderClient client) {
        this.client = client;
    }

    public Post fetchPost(Long id) {
        return client.getPostById(id);
    }
}
```

### **Pros & Cons:**
✅ **Clean, declarative approach** (no manual HTTP calls).  
✅ Integrates with **Spring Cloud LoadBalancer** (for service discovery).  
❌ Requires **Spring Cloud** setup.  

---

## **4. Using `HttpClient` (Java 11+)**
**When to use?**  
- Low-level HTTP calls without Spring dependencies.  
- Maximum control over requests.  

### **Example:**
```java
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

public class CryptoPriceService {
    
    private static final HttpClient httpClient = HttpClient.newHttpClient();
    private static final String API_URL = "https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd";

    public String fetchBitcoinPrice() throws Exception {
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(API_URL))
                .GET()
                .build();

        HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());
        return response.body();
    }
}
```

### **Pros & Cons:**
✅ **No Spring dependency** (works in plain Java).  
✅ **Supports HTTP/2**.  
❌ More verbose than `WebClient`/`Feign`.  

---

## **5. Using `RestClient` (Spring 6.1+)**
**New in Spring 6.1**, a simpler alternative to `RestTemplate`.  

### **Example:**
```java
import org.springframework.web.client.RestClient;

@Service
public class TodoService {
    
    private final RestClient restClient;

    public TodoService() {
        this.restClient = RestClient.create("https://jsonplaceholder.typicode.com");
    }

    public Todo fetchTodo(Long id) {
        return restClient.get()
                .uri("/todos/{id}", id)
                .retrieve()
                .body(Todo.class);
    }
}
```

### **Pros & Cons:**
✅ **Modern replacement for `RestTemplate`**.  
✅ **Simpler than `WebClient`** for non-reactive apps.  
❌ Less mature than `WebClient`/`Feign`.  

---

## **🔹 Best Practices for Consuming External APIs**
1. **Use Retry Mechanisms**  
   - Spring Retry (`@Retryable`) or Resilience4j for transient failures.  
2. **Circuit Breakers**  
   - Use Resilience4j or Spring Cloud Circuit Breaker to avoid cascading failures.  
3. **Timeouts**  
   - Always set timeouts (`connectTimeout`, `readTimeout`).  
4. **Error Handling**  
   - Use `ResponseEntity`, `onStatus()` in `WebClient`, or Feign error decoders.  
5. **Caching**  
   - Cache responses with `@Cacheable` (if the API allows it).  
6. **Rate Limiting**  
   - Use `Bucket4j` or API Gateway throttling.  

---

## **🔹 Comparison of HTTP Clients in Spring**
| Method          | Blocking | Async | Best For | Example Use Case |
|----------------|---------|-------|----------|------------------|
| **RestTemplate** | ✅ | ❌ | Legacy apps | Simple REST calls |
| **WebClient**  | ❌ | ✅ | Reactive apps | High-performance APIs |
| **Feign Client** | ✅ | ❌ | Microservices | Declarative REST |
| **HttpClient** | ✅ | ✅ (CompletableFuture) | Plain Java apps | Low-level control |
| **RestClient** | ✅ | ❌ | Spring 6.1+ | Modern `RestTemplate` alternative |

---

### **Final Recommendations**
- **For most Spring Boot apps** → Use **`WebClient`** (non-blocking) or **`Feign`** (declarative).  
- **For legacy apps** → Use **`RestTemplate`** (but migrate to `RestClient`).  
- **For maximum control** → Use **Java `HttpClient`**.  

Would you like a deeper dive into **error handling with `WebClient`** or **Feign + Spring Cloud integration**? 🚀

-----

To **consume external APIs** in a **Spring Boot** application (or any Java microservice), you typically use one of the following clients depending on your use case, preference, and whether the call should be **synchronous** or **asynchronous**.

---

## ✅ 1. **Using `RestTemplate` (Traditional, Blocking)**

> Easy to use, but considered legacy since Spring 5.

```java
@RestController
public class WeatherController {

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/weather")
    public String getWeather() {
        String url = "https://api.weatherapi.com/v1/current.json?key=API_KEY&q=London";
        return restTemplate.getForObject(url, String.class);
    }
}

@Configuration
class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

---

## ✅ 2. **Using `WebClient` (Reactive, Non-blocking - Preferred in Spring 5+)**

```java
@RestController
public class WeatherController {

    private final WebClient webClient = WebClient.create();

    @GetMapping("/weather")
    public Mono<String> getWeather() {
        return webClient.get()
                .uri("https://api.weatherapi.com/v1/current.json?key=API_KEY&q=London")
                .retrieve()
                .bodyToMono(String.class);
    }
}
```

---

## ✅ 3. **Using `FeignClient` (Declarative HTTP Client, best with service discovery)**

> Works great in microservices with Spring Cloud. Less boilerplate.

### Step 1: Add dependency (Spring Cloud OpenFeign)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

### Step 2: Enable Feign

```java
@SpringBootApplication
@EnableFeignClients
public class MyApp { }
```

### Step 3: Create a Feign interface

```java
@FeignClient(name = "weatherClient", url = "https://api.weatherapi.com/v1")
public interface WeatherClient {

    @GetMapping("/current.json?key=API_KEY&q={city}")
    String getCurrentWeather(@PathVariable("city") String city);
}
```

### Step 4: Use it

```java
@RestController
public class WeatherController {

    @Autowired
    private WeatherClient weatherClient;

    @GetMapping("/weather/{city}")
    public String getWeather(@PathVariable String city) {
        return weatherClient.getCurrentWeather(city);
    }
}
```

---

## 🔐 Consuming APIs with Authentication

| Auth Type          | How to use                                                |
| ------------------ | --------------------------------------------------------- |
| **API Key**        | Pass in header or query param                             |
| **Basic Auth**     | Use `HttpHeaders` with `BasicAuthenticationInterceptor`   |
| **OAuth2 / JWT**   | Use token-based headers (`Authorization: Bearer <token>`) |
| **Custom Headers** | Add using `HttpHeaders` in `RestTemplate` or `WebClient`  |

---

## 🧪 Best Practices

* Use **DTOs** instead of `String.class` to parse structured JSON
* Add **exception handling** (`try-catch` or `.onStatus()` in `WebClient`)
* Use **timeout settings**
* Add **retry** mechanism (e.g., `Resilience4j`)

---

## 📌 Summary

| Client         | Type           | When to Use                                                 |
| -------------- | -------------- | ----------------------------------------------------------- |
| `RestTemplate` | Synchronous    | Simple, legacy systems                                      |
| `WebClient`    | Async/Reactive | Preferred for new apps (non-blocking)                       |
| `FeignClient`  | Declarative    | Clean and maintainable, great for internal or external APIs |

---

Let me know if you'd like a working Spring Boot example of any of these!
