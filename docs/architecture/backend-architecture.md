# Backend Architecture

This section details the Spring Boot backend application structure, service layer design, security configuration, and key implementation patterns.

## Project Structure

```
backend/
├── src/
│   ├── main/
│   │   ├── java/com/pawpatrol/
│   │   │   ├── PawPatrolApplication.java    # Spring Boot entry point
│   │   │   ├── config/                       # Configuration classes
│   │   │   │   ├── SecurityConfig.java
│   │   │   │   ├── JwtConfig.java
│   │   │   │   ├── CorsConfig.java
│   │   │   │   └── OpenApiConfig.java
│   │   │   ├── controller/                   # REST controllers
│   │   │   │   ├── AuthController.java
│   │   │   │   ├── ColonyController.java
│   │   │   │   ├── CatController.java
│   │   │   │   ├── ActivityController.java
│   │   │   │   └── AdminController.java
│   │   │   ├── service/                      # Business logic
│   │   │   │   ├── AuthService.java
│   │   │   │   ├── ColonyService.java
│   │   │   │   ├── CatService.java
│   │   │   │   ├── ActivityService.java
│   │   │   │   ├── AdminService.java
│   │   │   │   ├── AuditService.java
│   │   │   │   ├── GeocodingService.java
│   │   │   │   └── FileStorageService.java
│   │   │   ├── repository/                   # Data access
│   │   │   │   ├── UserRepository.java
│   │   │   │   ├── ColonyRepository.java
│   │   │   │   ├── CatRepository.java
│   │   │   │   ├── ActivityRepository.java
│   │   │   │   └── AuditEventRepository.java
│   │   │   ├── entity/                       # JPA entities
│   │   │   │   ├── User.java
│   │   │   │   ├── Colony.java
│   │   │   │   ├── Cat.java
│   │   │   │   ├── Activity.java
│   │   │   │   └── AuditEvent.java
│   │   │   ├── dto/                          # Data transfer objects
│   │   │   │   ├── request/
│   │   │   │   │   ├── LoginRequest.java
│   │   │   │   │   ├── RegisterRequest.java
│   │   │   │   │   ├── CreateColonyRequest.java
│   │   │   │   │   ├── CreateCatRequest.java
│   │   │   │   │   └── LogActivityRequest.java
│   │   │   │   └── response/
│   │   │   │       ├── LoginResponse.java
│   │   │   │       ├── ColonyDTO.java
│   │   │   │       ├── CatDTO.java
│   │   │   │       ├── ActivityDTO.java
│   │   │   │       └── ErrorResponse.java
│   │   │   ├── security/                     # Security components
│   │   │   │   ├── JwtTokenProvider.java
│   │   │   │   ├── JwtAuthenticationFilter.java
│   │   │   │   ├── UserDetailsServiceImpl.java
│   │   │   │   └── CustomAuthenticationEntryPoint.java
│   │   │   ├── exception/                    # Custom exceptions
│   │   │   │   ├── GlobalExceptionHandler.java
│   │   │   │   ├── NotFoundException.java
│   │   │   │   ├── DuplicateResourceException.java
│   │   │   │   ├── UnauthorizedException.java
│   │   │   │   └── ValidationException.java
│   │   │   └── util/                         # Utility classes
│   │   │       ├── DateUtil.java
│   │   │       └── DistanceCalculator.java
│   │   └── resources/
│   │       ├── application.properties         # Main configuration
│   │       ├── application-dev.properties     # Dev profile
│   │       ├── application-prod.properties    # Prod profile
│   │       └── db/migration/                  # Flyway migrations
│   │           └── V1__Initial_schema.sql
│   └── test/
│       └── java/com/pawpatrol/
│           ├── controller/                    # Controller tests
│           ├── service/                       # Service tests
│           └── repository/                    # Repository tests
├── Dockerfile                                 # Docker build instructions
├── pom.xml                                    # Maven dependencies
└── README.md
```

---

## Service Layer Design

**Service Layer Principles:**
- Single Responsibility: Each service handles one domain entity
- Transaction Management: `@Transactional` at service method level
- No direct controller → repository access (always through service)
- Services coordinate between repositories, never repositories between services

**Example Service Implementation:**

```java
@Service
@RequiredArgsConstructor
@Transactional
public class ColonyService {
    
    private final ColonyRepository colonyRepository;
    private final CatRepository catRepository;
    private final ActivityRepository activityRepository;
    private final AuditService auditService;
    private final GeocodingService geocodingService;
    
    private static final double DUPLICATE_THRESHOLD_METERS = 100.0;
    
    public ColonyDTO getColonyById(UUID id) {
        Colony colony = colonyRepository.findById(id)
            .orElseThrow(() -> new NotFoundException("Colony not found: " + id));
        
        return buildColonyDTO(colony);
    }
    
    public ColonyDTO createColony(CreateColonyRequest request, UUID creatorId) {
        // Check for duplicate within 100m
        List<Colony> nearbyColonies = colonyRepository.findNearbyColonies(
            request.getLatitude(),
            request.getLongitude(),
            DUPLICATE_THRESHOLD_METERS
        );
        
        if (!nearbyColonies.isEmpty()) {
            throw new DuplicateResourceException(
                "Colony exists within 100m",
                nearbyColonies.get(0).getId()
            );
        }
        
        // Geocode if address provided
        String address = request.getAddress();
        if (address == null || address.isBlank()) {
            address = geocodingService.reverseGeocode(
                request.getLatitude(),
                request.getLongitude()
            );
        }
        
        Colony colony = Colony.builder()
            .name(request.getName())
            .latitude(request.getLatitude())
            .longitude(request.getLongitude())
            .address(address)
            .creator(userRepository.getReferenceById(creatorId))
            .build();
        
        Colony saved = colonyRepository.save(colony);
        auditService.logEvent(AuditEventType.COLONY_CREATED, creatorId, saved.getId(), null);
        
        return buildColonyDTO(saved);
    }
    
    private ColonyDTO buildColonyDTO(Colony colony) {
        // Calculate computed fields
        int catCount = catRepository.countByColonyIdAndStatusNot(
            colony.getId(),
            CatStatus.DECEASED
        );
        
        Instant lastFedAt = activityRepository.findLastFeedingTime(colony.getId())
            .orElse(null);
        
        TnrProgress tnrProgress = calculateTnrProgress(colony.getId());
        
        return ColonyDTO.fromEntity(colony, catCount, lastFedAt, tnrProgress);
    }
    
    private TnrProgress calculateTnrProgress(UUID colonyId) {
        int total = catRepository.countByColonyIdAndStatusNot(colonyId, CatStatus.DECEASED);
        int neutered = catRepository.countByColonyIdAndIsNeutered(colonyId, true);
        int notNeutered = catRepository.countByColonyIdAndIsNeutered(colonyId, false);
        int unknown = catRepository.countByColonyIdAndIsNeuteredIsNull(colonyId);
        
        double percentComplete = total > 0 ? (neutered * 100.0 / total) : 0.0;
        
        return TnrProgress.builder()
            .total(total)
            .neutered(neutered)
            .notNeutered(notNeutered)
            .unknown(unknown)
            .percentComplete(Math.round(percentComplete * 10) / 10.0)
            .build();
    }
}
```

---

## Security Configuration

**Spring Security Setup:**

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {
    
    @Autowired
    private JwtAuthenticationFilter jwtAuthenticationFilter;
    
    @Autowired
    private CustomAuthenticationEntryPoint authenticationEntryPoint;
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .cors(Customizer.withDefaults())
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/admin/**").hasAnyRole("ADMIN", "SUPER_ADMIN")
                .requestMatchers("/api/**").hasRole("USER")
                .anyRequest().authenticated()
            )
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(authenticationEntryPoint))
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(10);
    }
    
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }
}
```

**JWT Token Provider:**

```java
@Component
public class JwtTokenProvider {
    
    @Value("${jwt.secret}")
    private String jwtSecret;
    
    @Value("${jwt.expiration}")
    private long jwtExpiration; // 7 days in milliseconds
    
    @Value("${jwt.refresh-expiration}")
    private long refreshExpiration; // 30 days
    
    public String generateToken(User user) {
        return Jwts.builder()
            .setSubject(user.getId().toString())
            .claim("email", user.getEmail())
            .claim("role", user.getRole().name())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + jwtExpiration))
            .signWith(SignatureAlgorithm.HS512, jwtSecret)
            .compact();
    }
    
    public String generateRefreshToken(User user) {
        return Jwts.builder()
            .setSubject(user.getId().toString())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + refreshExpiration))
            .signWith(SignatureAlgorithm.HS512, jwtSecret)
            .compact();
    }
    
    public UUID getUserIdFromToken(String token) {
        Claims claims = Jwts.parser()
            .setSigningKey(jwtSecret)
            .parseClaimsJws(token)
            .getBody();
        return UUID.fromString(claims.getSubject());
    }
    
    public boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            return false;
        }
    }
}
```

---
