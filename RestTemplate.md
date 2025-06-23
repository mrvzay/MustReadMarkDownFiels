# **Using `RestTemplate` in Spring Boot (Synchronous HTTP Client)**  

`RestTemplate` is a **synchronous, blocking HTTP client** provided by Spring for making REST API calls. Although it's being **deprecated** in favor of `WebClient` (reactive), it's still widely used in legacy applications.

---

## **🔹 1. How to Use `RestTemplate`**
### **Step 1: Add Dependency (Spring Web)**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
*(Note: In newer Spring Boot versions, you might need to manually define a `RestTemplate` bean.)*

### **Step 2: Create a `RestTemplate` Bean**
```java
@Configuration
public class AppConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```
*(For better control, use `RestTemplateBuilder`.)*

---

## **🔹 2. Common `RestTemplate` Methods**
| Method | Usage | Example |
|--------|-------|---------|
| **`getForObject()`** | GET request → returns POJO | `restTemplate.getForObject(url, User.class, params)` |
| **`getForEntity()`** | GET request → returns `ResponseEntity` | `restTemplate.getForEntity(url, User.class, params)` |
| **`postForObject()`** | POST request → returns POJO | `restTemplate.postForObject(url, requestObj, User.class)` |
| **`postForEntity()`** | POST request → returns `ResponseEntity` | `restTemplate.postForEntity(url, requestObj, User.class)` |
| **`exchange()`** | Custom HTTP method (PUT, DELETE, etc.) | `restTemplate.exchange(url, HttpMethod.PUT, requestEntity, User.class)` |
| **`delete()`** | DELETE request | `restTemplate.delete(url, userId)` |

---

## **🔹 3. Example: Consuming a REST API**
### **GET Request (Fetch Data)**
```java
@Service
public class UserService {

    @Autowired
    private RestTemplate restTemplate;

    public User fetchUser(Long userId) {
        String url = "https://jsonplaceholder.typicode.com/users/{id}";
        return restTemplate.getForObject(url, User.class, userId);
    }
}
```

### **POST Request (Send Data)**
```java
public User createUser(User user) {
    String url = "https://jsonplaceholder.typicode.com/users";
    return restTemplate.postForObject(url, user, User.class);
}
```

### **Using `ResponseEntity` for Headers/Status**
```java
public ResponseEntity<User> fetchUserWithDetails(Long userId) {
    String url = "https://jsonplaceholder.typicode.com/users/{id}";
    return restTemplate.getForEntity(url, User.class, userId);
}
```

---

## **🔹 4. Advanced Usage**
### **Adding Headers**
```java
HttpHeaders headers = new HttpHeaders();
headers.set("Authorization", "Bearer token123");
HttpEntity<String> requestEntity = new HttpEntity<>(headers);

ResponseEntity<User> response = restTemplate.exchange(
    url, 
    HttpMethod.GET, 
    requestEntity, 
    User.class
);
```

### **Error Handling**
```java
try {
    User user = restTemplate.getForObject(url, User.class, userId);
} catch (HttpClientErrorException e) {
    if (e.getStatusCode() == HttpStatus.NOT_FOUND) {
        throw new UserNotFoundException("User not found!");
    }
    throw e;
}
```

### **Using `RestTemplateBuilder` (Recommended)**
```java
@Bean
public RestTemplate restTemplate(RestTemplateBuilder builder) {
    return builder
            .setConnectTimeout(Duration.ofSeconds(5))
            .setReadTimeout(Duration.ofSeconds(5))
            .build();
}
```

---

## **🔹 5. Limitations & Best Practices**
### **❌ Problems with `RestTemplate`**
1. **Blocking** – Each request blocks the thread (bad for scalability).  
2. **Deprecated** – Spring recommends `WebClient` (reactive).  
3. **No Retry/Circuit Breaker** – Needs manual implementation.  

### **✅ Best Practices**
✔ **Use `RestTemplateBuilder`** (instead of `new RestTemplate()`).  
✔ **Set timeouts** to avoid hanging requests.  
✔ **Consider migrating to `WebClient`** for non-blocking calls.  

---

## **🔹 6. Comparison: `RestTemplate` vs `WebClient`**
| Feature | `RestTemplate` | `WebClient` |
|---------|--------------|------------|
| **Type** | Blocking | Non-blocking (Reactive) |
| **Performance** | Slower (thread-per-request) | Faster (event-loop) |
| **Spring 5+** | Deprecated | Recommended |
| **Error Handling** | Try-catch | Reactive operators (`onErrorResume`) |
| **Use Case** | Legacy apps | Modern microservices |

---

## **🔹 Final Verdict**
- **If you're on Spring Boot 2.x/3.x**, prefer **`WebClient`** for better scalability.  
- **For simple, blocking calls**, `RestTemplate` still works but is **not future-proof**.  

Would you like a **migration guide from `RestTemplate` to `WebClient`**? 🚀


---------------


### ✅ **`RestTemplate` – Consuming External APIs in Spring Boot**

`RestTemplate` is a synchronous client to perform HTTP requests, exposing methods to interact with RESTful services. It’s widely used but considered **legacy** (Spring now recommends `WebClient` for non-blocking I/O), though still supported.

---

## 🔧 **1. Add `RestTemplate` Bean**

```java
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

---

## 🧪 **2. Basic Example – Consuming a Public API**

```java
@RestController
@RequestMapping("/api")
public class WeatherController {

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/weather")
    public String getWeather() {
        String url = "https://api.weatherapi.com/v1/current.json?key=YOUR_API_KEY&q=London";
        return restTemplate.getForObject(url, String.class);
    }
}
```

---

## 📦 **3. Using a Custom DTO for Response**

```java
public class WeatherResponse {
    private Location location;
    private Current current;
    // getters and setters

    public static class Location {
        private String name;
        private String country;
        // getters and setters
    }

    public static class Current {
        private double temp_c;
        // getters and setters
    }
}
```

```java
@GetMapping("/weather/dto")
public WeatherResponse getWeatherDto() {
    String url = "https://api.weatherapi.com/v1/current.json?key=YOUR_API_KEY&q=London";
    return restTemplate.getForObject(url, WeatherResponse.class);
}
```

---

## 🔐 **4. With Headers (e.g., Authorization / API Key)**

```java
@GetMapping("/secure-data")
public ResponseEntity<String> getSecureData() {
    HttpHeaders headers = new HttpHeaders();
    headers.set("Authorization", "Bearer YOUR_TOKEN_HERE");

    HttpEntity<String> entity = new HttpEntity<>(headers);
    String url = "https://external-api.com/data";

    return restTemplate.exchange(url, HttpMethod.GET, entity, String.class);
}
```

---

## ⚠️ **5. Error Handling with `try-catch`**

```java
try {
    String response = restTemplate.getForObject(url, String.class);
    return response;
} catch (HttpClientErrorException e) {
    return "Client error: " + e.getStatusCode();
} catch (HttpServerErrorException e) {
    return "Server error: " + e.getStatusCode();
}
```

---

## 🔄 **6. POST Request Example**

```java
@PostMapping("/create")
public String createUser() {
    String url = "https://external-api.com/users";
    Map<String, String> request = new HashMap<>();
    request.put("name", "Vijay");

    return restTemplate.postForObject(url, request, String.class);
}
```

---

## 📌 Summary

| Method            | Description                                    |
| ----------------- | ---------------------------------------------- |
| `getForObject()`  | HTTP GET, returns object                       |
| `postForObject()` | HTTP POST, returns object                      |
| `exchange()`      | Full control (method, headers, body)           |
| `getForEntity()`  | Like `getForObject` but with response metadata |

---

### 🧼 RestTemplate vs WebClient (Modern Approach)

| Feature        | `RestTemplate`     | `WebClient` (Preferred) |
| -------------- | ------------------ | ----------------------- |
| Threading      | Blocking           | Non-blocking (Reactive) |
| Flexibility    | Limited            | Higher                  |
| Performance    | Lower under load   | Scales better           |
| Spring Support | Supported (legacy) | Preferred (Spring 5+)   |

---

Let me know if you want a **complete working Spring Boot project** using `RestTemplate` for GET/POST requests.
