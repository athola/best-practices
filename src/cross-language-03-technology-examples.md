# 8.4 Technology Stack Examples

Real-world cross-language integration requires careful consideration of technology stacks and architecture patterns. This chapter provides practical examples of how different languages can work together effectively in various scenarios.

## Microservices Architecture

### E-commerce Platform Example
**Multi-language microservices for scalability and specialization**

```
Frontend (React/TypeScript) → API Gateway (Node.js) →
├── User Service (Python/Django) - Authentication and user management
├── Product Service (Java/Spring Boot) - Product catalog and inventory
├── Order Service (Go) - High-performance order processing
├── Payment Service (Rust) - Secure payment processing
├── Recommendation Service (Python/ML) - Machine learning recommendations
└── Notification Service (Node.js) - Email and push notifications
```

#### Implementation Details

**API Gateway (Node.js)**
```javascript
// Express.js API Gateway with load balancing
const express = require('express');
const axios = require('axios');
const circuitBreaker = require('opossum');

const app = express();
app.use(express.json());

// Circuit breaker for each service
const userServiceBreaker = new circuitBreaker(axios.get, {
  timeout: 5000,
  errorThresholdPercentage: 50,
  resetTimeout: 30000
});

// Route to user service
app.get('/api/users/:id', async (req, res) => {
  try {
    const response = await userServiceBreaker.fire(
      `http://user-service:8000/users/${req.params.id}`
    );
    res.json(response.data);
  } catch (error) {
    res.status(503).json({ error: 'User service unavailable' });
  }
});

// Route to product service
app.get('/api/products', async (req, res) => {
  try {
    const response = await axios.get(
      `http://product-service:8080/products`,
      { params: req.query }
    );
    res.json(response.data);
  } catch (error) {
    res.status(503).json({ error: 'Product service unavailable' });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`API Gateway running on port ${PORT}`);
});
```

**User Service (Python/Django)**
```python
# Django REST Framework for user management
from django.contrib.auth.models import User
from rest_framework import serializers, viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
import jwt
from datetime import datetime, timedelta

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'first_name', 'last_name']

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    
    @action(detail=False, methods=['post'])
    def login(self, request):
        username = request.data.get('username')
        password = request.data.get('password')
        
        user = authenticate(username=username, password=password)
        if user:
            # Generate JWT token
            payload = {
                'user_id': user.id,
                'username': user.username,
                'exp': datetime.utcnow() + timedelta(hours=24)
            }
            token = jwt.encode(payload, 'your-secret-key', algorithm='HS256')
            
            return Response({
                'token': token,
                'user': UserSerializer(user).data
            })
        
        return Response(
            {'error': 'Invalid credentials'}, 
            status=status.HTTP_401_UNAUTHORIZED
        )

# Django settings for cross-service communication
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
    "http://api-gateway:3000"
]

# Redis for session management and caching
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://redis:6379/1',
    }
}
```

**Order Service (Go)**
```go
// High-performance order processing in Go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"
	"sync"
	"time"
	
	"github.com/gorilla/mux"
	"github.com/streadway/amqp"
)

type Order struct {
	ID          string    `json:"id"`
	UserID      string    `json:"user_id"`
	ProductID   string    `json:"product_id"`
	Quantity    int       `json:"quantity"`
	TotalAmount float64   `json:"total_amount"`
	Status      string    `json:"status"`
	CreatedAt   time.Time `json:"created_at"`
}

var orders = make(map[string]Order)
var ordersMutex sync.RWMutex

func main() {
	r := mux.NewRouter()
	
	// Order endpoints
	r.HandleFunc("/orders", createOrder).Methods("POST")
	r.HandleFunc("/orders/{id}", getOrder).Methods("GET")
	r.HandleFunc("/orders/{id}/status", updateOrderStatus).Methods("PUT")
	
	// Start message queue consumer
	go startMessageQueueConsumer()
	
	fmt.Println("Order Service starting on port 8080")
	log.Fatal(http.ListenAndServe(":8080", r))
}

func createOrder(w http.ResponseWriter, r *http.Request) {
	var order Order
	if err := json.NewDecoder(r.Body).Decode(&order); err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}
	
	// Validate order
	if order.Quantity <= 0 {
		http.Error(w, "Quantity must be positive", http.StatusBadRequest)
		return
	}
	
	// Generate order ID
	order.ID = generateOrderID()
	order.Status = "pending"
	order.CreatedAt = time.Now()
	
	// Store order
	ordersMutex.Lock()
	orders[order.ID] = order
	ordersMutex.Unlock()
	
	// Send order to message queue for processing
	sendOrderToQueue(order)
	
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(order)
}

func sendOrderToQueue(order Order) {
	// Connect to RabbitMQ and send order for processing
	conn, err := amqp.Dial("amqp://guest:guest@rabbitmq:5672/")
	if err != nil {
		log.Printf("Failed to connect to RabbitMQ: %v", err)
		return
	}
	defer conn.Close()
	
	ch, err := conn.Channel()
	if err != nil {
		log.Printf("Failed to open a channel: %v", err)
		return
	}
	defer ch.Close()
	
	q, err := ch.QueueDeclare(
		"orders", // name
		true,    // durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	if err != nil {
		log.Printf("Failed to declare a queue: %v", err)
		return
	}
	
	body, err := json.Marshal(order)
	if err != nil {
		log.Printf("Failed to marshal order: %v", err)
		return
	}
	
	err = ch.Publish(
		"",     // exchange
		q.Name, // routing key
		false,  // mandatory
		false,  // immediate
		amqp.Publishing{
			ContentType: "application/json",
			Body:        body,
		})
	if err != nil {
		log.Printf("Failed to publish a message: %v", err)
	}
}
```

## Data Processing Pipeline

### Real-time Analytics Platform
**Multi-language data processing for different stages**

```
Data Sources → Message Queue (Kafka) →
├── Ingestion (Python) → Data validation and normalization
├── Stream Processing (Scala/Spark) → Real-time aggregations
├── Batch Processing (Python/Pandas) → Historical analysis
├── ML Training (Python/R) → Model training and evaluation
└── API Service (Go) → Low-latency query serving
```

#### Implementation Examples

**Data Ingestion (Python)**
```python
# Kafka producer for data ingestion
from kafka import KafkaProducer
from kafka.errors import KafkaError
import json
import logging
from datetime import datetime
import uuid

class DataIngestor:
    def __init__(self, bootstrap_servers='localhost:9092'):
        self.producer = KafkaProducer(
            bootstrap_servers=bootstrap_servers,
            value_serializer=lambda v: json.dumps(v).encode('utf-8'),
            key_serializer=lambda k: k.encode('utf-8') if k else None
        )
        self.logger = logging.getLogger(__name__)
    
    def ingest_user_event(self, user_id, event_type, event_data):
        """Ingest user events to Kafka"""
        try:
            event = {
                'event_id': str(uuid.uuid4()),
                'user_id': user_id,
                'event_type': event_type,
                'event_data': event_data,
                'timestamp': datetime.now().isoformat(),
                'ingestion_time': datetime.now().isoformat()
            }
            
            # Send to Kafka
            future = self.producer.send(
                'user-events',
                key=user_id,
                value=event
            )
            
            # Wait for send to complete
            record_metadata = future.get(timeout=10)
            
            self.logger.info(
                f"Event sent to topic {record_metadata.topic} "
                f"partition {record_metadata.partition} "
                f"offset {record_metadata.offset}"
            )
            
            return event['event_id']
            
        except KafkaError as e:
            self.logger.error(f"Failed to send event to Kafka: {e}")
            raise
        except Exception as e:
            self.logger.error(f"Unexpected error during ingestion: {e}")
            raise
    
    def close(self):
        """Close the producer"""
        self.producer.close()

# Usage example
if __name__ == "__main__":
    ingestor = DataIngestor()
    
    try:
        event_id = ingestor.ingest_user_event(
            user_id="user123",
            event_type="page_view",
            event_data={
                "page": "/products/123",
                "referrer": "https://google.com",
                "user_agent": "Mozilla/5.0..."
            }
        )
        print(f"Event ingested with ID: {event_id}")
    finally:
        ingestor.close()
```

**Stream Processing (Scala/Spark)**
```scala
// Spark Streaming for real-time processing
import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.spark.streaming.kafka010._
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.functions._
import org.apache.spark.sql.types._

object RealTimeAnalytics {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder()
      .appName("RealTimeAnalytics")
      .getOrCreate()
    
    import spark.implicits._
    
    // Define schema for user events
    val userEventSchema = StructType(Array(
      StructField("event_id", StringType, nullable = false),
      StructField("user_id", StringType, nullable = false),
      StructField("event_type", StringType, nullable = false),
      StructField("event_data", MapType(StringType, StringType), nullable = true),
      StructField("timestamp", TimestampType, nullable = false),
      StructField("ingestion_time", TimestampType, nullable = false)
    ))
    
    // Read from Kafka
    val df = spark.readStream
      .format("kafka")
      .option("kafka.bootstrap.servers", "localhost:9092")
      .option("subscribe", "user-events")
      .option("startingOffsets", "latest")
      .load()
    
    // Parse JSON data
    val eventsDF = df
      .selectExpr("CAST(value AS STRING) as json")
      .select(from_json($"json", userEventSchema).as("data"))
      .select("data.*")
    
    // Real-time aggregations
    val pageViewCounts = eventsDF
      .filter($"event_type" === "page_view")
      .groupBy(
        window($"timestamp", "1 minute"),
        $"event_data.page"
      )
      .count()
      .orderBy($"window")
    
    // User activity summary
    val userActivity = eventsDF
      .groupBy(
        window($"timestamp", "5 minutes"),
        $"user_id"
      )
      .agg(
        count("*").as("total_events"),
        countDistinct("event_type").as("unique_event_types"),
        collect_set("event_type").as("event_types")
      )
    
    // Write results to console (in production, write to database or another Kafka topic)
    val pageViewQuery = pageViewCounts.writeStream
      .outputMode("complete")
      .format("console")
      .option("truncate", "false")
      .start()
    
    val userActivityQuery = userActivity.writeStream
      .outputMode("complete")
      .format("console")
      .option("truncate", "false")
      .start()
    
    pageViewQuery.awaitTermination()
    userActivityQuery.awaitTermination()
  }
}
```

**API Service (Go)**
```go
// High-performance API for serving analytics data
package main

import (
	"database/sql"
	"encoding/json"
	"fmt"
	"log"
	"net/http"
	"time"
	
	_ "github.com/lib/pq"
	"github.com/gorilla/mux"
	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

type AnalyticsService struct {
	db *sql.DB
}

type PageViewStats struct {
	Page      string    `json:"page"`
	Count     int       `json:"count"`
	Window    time.Time `json:"window"`
	Duration  string    `json:"duration"`
}

type UserActivity struct {
	UserID           string   `json:"user_id"`
	TotalEvents      int      `json:"total_events"`
	UniqueEventTypes int      `json:"unique_event_types"`
	EventTypes       []string `json:"event_types"`
	Window           time.Time `json:"window"`
}

var (
	requestDuration = prometheus.NewHistogramVec(
		prometheus.HistogramOpts{
			Name: "analytics_request_duration_seconds",
			Help: "Duration of analytics requests",
		},
		[]string{"endpoint"},
	)
	
	requestCount = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "analytics_requests_total",
			Help: "Total number of analytics requests",
		},
		[]string{"endpoint", "status"},
	)
)

func init() {
	prometheus.MustRegister(requestDuration)
	prometheus.MustRegister(requestCount)
}

func main() {
	// Initialize database connection
	db, err := sql.Open("postgres", "postgres://user:password@localhost:5432/analytics?sslmode=disable")
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()
	
	service := &AnalyticsService{db: db}
	
	r := mux.NewRouter()
	
	// Analytics endpoints
	r.HandleFunc("/api/analytics/page-views", service.getPageViewStats).Methods("GET")
	r.HandleFunc("/api/analytics/user-activity/{user_id}", service.getUserActivity).Methods("GET")
	
	// Health check
	r.HandleFunc("/health", service.healthCheck).Methods("GET")
	
	// Metrics endpoint
	r.Handle("/metrics", promhttp.Handler())
	
	fmt.Println("Analytics API Service starting on port 8080")
	log.Fatal(http.ListenAndServe(":8080", r))
}

func (s *AnalyticsService) getPageViewStats(w http.ResponseWriter, r *http.Request) {
	start := time.Now()
	
	// Parse query parameters
	page := r.URL.Query().Get("page")
	duration := r.URL.Query().Get("duration")
	if duration == "" {
		duration = "1h"
	}
	
	// Query database
	query := `
		SELECT page, COUNT(*) as count, 
		       date_trunc('hour', timestamp) as window
		FROM user_events 
		WHERE event_type = 'page_view'
		AND timestamp >= NOW() - INTERVAL '1 hour'
	`
	
	if page != "" {
		query += " AND event_data->>'page' = $1"
	}
	
	query += " GROUP BY page, date_trunc('hour', timestamp) ORDER BY window"
	
	var rows *sql.Rows
	var err error
	
	if page != "" {
		rows, err = s.db.Query(query, page)
	} else {
		rows, err = s.db.Query(query)
	}
	
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		requestCount.WithLabelValues("/api/analytics/page-views", "500").Inc()
		return
	}
	defer rows.Close()
	
	var stats []PageViewStats
	for rows.Next() {
		var stat PageViewStats
		err := rows.Scan(&stat.Page, &stat.Count, &stat.Window)
		if err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)
			requestCount.WithLabelValues("/api/analytics/page-views", "500").Inc()
			return
		}
		stat.Duration = duration
		stats = append(stats, stat)
	}
	
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(stats)
	
	// Record metrics
	duration := time.Since(start).Seconds()
	requestDuration.WithLabelValues("/api/analytics/page-views").Observe(duration)
	requestCount.WithLabelValues("/api/analytics/page-views", "200").Inc()
}

func (s *AnalyticsService) getUserActivity(w http.ResponseWriter, r *http.Request) {
	start := time.Now()
	
	vars := mux.Vars(r)
	userID := vars["user_id"]
	
	query := `
		SELECT user_id, COUNT(*) as total_events,
		       COUNT(DISTINCT event_type) as unique_event_types,
		       ARRAY_AGG(DISTINCT event_type) as event_types,
		       date_trunc('5 minutes', timestamp) as window
		FROM user_events 
		WHERE user_id = $1
		AND timestamp >= NOW() - INTERVAL '1 day'
		GROUP BY user_id, date_trunc('5 minutes', timestamp)
		ORDER BY window
	`
	
	rows, err := s.db.Query(query, userID)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		requestCount.WithLabelValues("/api/analytics/user-activity", "500").Inc()
		return
	}
	defer rows.Close()
	
	var activities []UserActivity
	for rows.Next() {
		var activity UserActivity
		var eventTypesArray []string
		
		err := rows.Scan(
			&activity.UserID,
			&activity.TotalEvents,
			&activity.UniqueEventTypes,
			&eventTypesArray,
			&activity.Window,
		)
		if err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)
			requestCount.WithLabelValues("/api/analytics/user-activity", "500").Inc()
			return
		}
		activity.EventTypes = eventTypesArray
		activities = append(activities, activity)
	}
	
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(activities)
	
	// Record metrics
	duration := time.Since(start).Seconds()
	requestDuration.WithLabelValues("/api/analytics/user-activity").Observe(duration)
	requestCount.WithLabelValues("/api/analytics/user-activity", "200").Inc()
}

func (s *AnalyticsService) healthCheck(w http.ResponseWriter, r *http.Request) {
	err := s.db.Ping()
	if err != nil {
		http.Error(w, "Database connection failed", http.StatusServiceUnavailable)
		return
	}
	
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(map[string]string{"status": "healthy"})
}
```

## Legacy System Integration

### Modernizing Legacy Systems
**Integrating new microservices with legacy monoliths**

```
Legacy System (COBOL/Mainframe) → Integration Layer (Java) →
├── Modern API Gateway (Node.js) → REST/GraphQL APIs
├── New Services (Python/Go/Rust) → Modern functionality
├── Data Sync Service (Python) → Database synchronization
└── Monitoring Service (Go) → Cross-system monitoring
```

#### Implementation Examples

**Integration Layer (Java)**
```java
// Spring Boot integration layer for legacy system
package com.example.legacyintegration;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;
import javax.jms.Queue;
import java.util.Map;
import java.util.HashMap;
import com.fasterxml.jackson.databind.ObjectMapper;

@SpringBootApplication
@RestController
@RequestMapping("/api/legacy")
public class LegacyIntegrationApplication {
    
    @Autowired
    private LegacySystemService legacySystemService;
    
    @Autowired
    private JmsTemplate jmsTemplate;
    
    @Autowired
    private Queue legacyRequestQueue;
    
    private final ObjectMapper objectMapper = new ObjectMapper();
    
    public static void main(String[] args) {
        SpringApplication.run(LegacyIntegrationApplication.class, args);
    }
    
    @PostMapping("/customers")
    public Map<String, Object> createCustomer(@RequestBody Map<String, Object> customerData) {
        try {
            // Transform modern JSON to legacy format
            LegacyCustomer legacyCustomer = transformToLegacyFormat(customerData);
            
            // Call legacy system
            String legacyResponse = legacySystemService.createCustomer(legacyCustomer);
            
            // Transform response back to modern format
            Map<String, Object> response = transformToModernFormat(legacyResponse);
            
            // Send to message queue for async processing
            jmsTemplate.convertAndSend(legacyRequestQueue, customerData);
            
            return response;
            
        } catch (Exception e) {
            Map<String, Object> error = new HashMap<>();
            error.put("error", "Failed to create customer");
            error.put("details", e.getMessage());
            return error;
        }
    }
    
    @GetMapping("/customers/{id}")
    public Map<String, Object> getCustomer(@PathVariable String id) {
        try {
            String legacyResponse = legacySystemService.getCustomer(id);
            return transformToModernFormat(legacyResponse);
        } catch (Exception e) {
            Map<String, Object> error = new HashMap<>();
            error.put("error", "Customer not found");
            error.put("details", e.getMessage());
            return error;
        }
    }
    
    private LegacyCustomer transformToLegacyFormat(Map<String, Object> modernData) {
        LegacyCustomer legacy = new LegacyCustomer();
        legacy.setCustomerId((String) modernData.get("id"));
        legacy.setCustomerName((String) modernData.get("name"));
        legacy.setCustomerAddress((String) modernData.get("address"));
        // Additional transformations...
        return legacy;
    }
    
    private Map<String, Object> transformToModernFormat(String legacyResponse) {
        // Parse legacy response and transform to modern JSON
        Map<String, Object> modern = new HashMap<>();
        // Transformation logic...
        return modern;
    }
}

// Legacy system service interface
interface LegacySystemService {
    String createCustomer(LegacyCustomer customer);
    String getCustomer(String customerId);
    String updateCustomer(String customerId, LegacyCustomer customer);
    String deleteCustomer(String customerId);
}

// Legacy customer data structure
class LegacyCustomer {
    private String customerId;
    private String customerName;
    private String customerAddress;
    // Additional legacy fields...
    
    // Getters and setters...
}
```

## Conclusion

These technology stack examples demonstrate how different programming languages can work together effectively in real-world scenarios. The key takeaways are:

1. **Choose the right language for each task**: Use languages based on their strengths
2. **Establish clear communication patterns**: APIs, message queues, or shared memory
3. **Implement proper error handling**: Consistent error handling across all services
4. **Monitor cross-language performance**: Track performance across language boundaries
5. **Plan for scalability**: Design systems that can grow and evolve

By following these patterns and examples, organizations can build robust, scalable systems that leverage the best features of multiple programming languages while maintaining maintainability and performance.