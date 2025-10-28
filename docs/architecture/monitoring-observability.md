# Monitoring & Observability

## Spring Boot Actuator

**Enabled Endpoints:**

```properties
# application-prod.properties
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=always
management.metrics.export.prometheus.enabled=true
```

**Health Check URL:** `https://paw-patrol.fly.dev/actuator/health`

**Metrics URL:** `https://paw-patrol.fly.dev/actuator/metrics`

---

## Logging Strategy

**Log Levels (Production):**

```properties
logging.level.root=INFO
logging.level.com.pawpatrol=INFO
logging.level.org.springframework.web=WARN
logging.level.org.hibernate.SQL=WARN
```

**Structured Logging (JSON format):**

```xml
<!-- logback-spring.xml -->
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

**View Logs:**

```bash
# Real-time logs
flyctl logs

# Recent logs
flyctl logs --tail=100
```

---
