# Integration Patterns

Cross-language integration can be achieved through various patterns, each with its own strengths, weaknesses, and appropriate use cases. Understanding these patterns is crucial for designing effective multi-language systems.

## 1. API-Based Integration

### REST/GraphQL APIs
**Language-agnostic communication through HTTP-based APIs**

#### Advantages
- **Universal Compatibility**: Works with any language that can make HTTP requests
- **Well-Understood**: Extensive tooling and community knowledge
- **Scalable**: Easy to scale horizontally with load balancers
- **Debuggable**: Standard HTTP debugging tools available

#### Implementation Examples

**Python Flask API**
```python
from flask import Flask, jsonify, request
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

@app.route('/api/users', methods=['GET'])
def get_users():
    users = [
        {"id": 1, "name": "Alice", "email": "alice@example.com"},
        {"id": 2, "name": "Bob", "email": "bob@example.com"}
    ]
    return jsonify(users)

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json()
    # Process user creation
    return jsonify({"message": "User created", "user": data}), 201

if __name__ == '__main__':
    app.run(debug=True, port=5000)
```

**Node.js Consumer**
```javascript
const axios = require('axios');

async function fetchUsers() {
    try {
        const response = await axios.get('http://localhost:5000/api/users');
        console.log('Users:', response.data);
        return response.data;
    } catch (error) {
        console.error('Error fetching users:', error.message);
    }
}

async function createUser(userData) {
    try {
        const response = await axios.post(
            'http://localhost:5000/api/users', 
            userData
        );
        console.log('Created user:', response.data);
        return response.data;
    } catch (error) {
        console.error('Error creating user:', error.message);
    }
}
```

#### Best Practices
- Use OpenAPI/Swagger for API documentation
- Implement proper authentication and authorization
- Use HTTPS for all communications
- Implement rate limiting and throttling
- Provide comprehensive error responses

### gRPC
**High-performance RPC framework using Protocol Buffers**

#### Advantages
- **Performance**: Binary serialization for faster communication
- **Strong Typing**: Schema-defined service contracts
- **Streaming Support**: Bidirectional streaming capabilities
- **Code Generation**: Automatic client/server code generation

#### Example

**Protocol Definition (.proto file)**
```protobuf
syntax = "proto3";

package user_service;

service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc ListUsers(ListUsersRequest) returns (ListUsersResponse);
  rpc CreateUser(CreateUserRequest) returns (User);
}

message GetUserRequest {
  int32 user_id = 1;
}

message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}

message ListUsersRequest {
  int32 page = 1;
  int32 limit = 2;
}

message ListUsersResponse {
  repeated User users = 1;
  int32 total = 2;
}

message CreateUserRequest {
  string name = 1;
  string email = 2;
}
```

## 2. Message Queue Integration

### Asynchronous Communication
**Decoupled services through message queues**

#### Popular Technologies
- **RabbitMQ**: Feature-rich message broker
- **Apache Kafka**: Distributed streaming platform
- **AWS SQS**: Simple queue service
- **Redis Pub/Sub**: Lightweight messaging

#### Implementation Examples

**Python Producer (RabbitMQ)**
```python
import pika
import json

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.queue_declare(queue='user_events')

def publish_user_event(event_type, user_data):
    message = {
        'event_type': event_type,
        'timestamp': datetime.now().isoformat(),
        'data': user_data
    }
    
    channel.basic_publish(
        exchange='',
        routing_key='user_events',
        body=json.dumps(message)
    )
    print(f"Published {event_type} event")

# Example usage
publish_user_event('user_created', {
    'id': 1,
    'name': 'Alice',
    'email': 'alice@example.com'
})

connection.close()
```

**Java Consumer (RabbitMQ)**
```java
import com.rabbitmq.client.*;
import com.fasterxml.jackson.databind.ObjectMapper;

public class UserEventConsumer {
    private final static String QUEUE_NAME = "user_events";
    private final static ObjectMapper objectMapper = new ObjectMapper();

    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            
            DeliverCallback deliverCallback = (consumerTag, delivery) -> {
                String message = new String(delivery.getBody(), "UTF-8");
                System.out.println("Received message: " + message);
                
                try {
                    Map<String, Object> event = objectMapper.readValue(
                        message, Map.class);
                    processUserEvent(event);
                } catch (Exception e) {
                    System.err.println("Error processing message: " + e.getMessage());
                }
            };
            
            channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> {});
            System.out.println("Waiting for messages...");
            Thread.sleep(Long.MAX_VALUE);
        }
    }
    
    private static void processUserEvent(Map<String, Object> event) {
        String eventType = (String) event.get("event_type");
        Map<String, Object> userData = (Map<String, Object>) event.get("data");
        
        System.out.println("Processing " + eventType + " for user: " + userData);
        // Process the event based on type
    }
}
```

#### Best Practices
- Use appropriate message serialization (JSON, Avro, Protobuf)
- Implement dead-letter queues for failed messages
- Monitor queue depths and processing times
- Handle message idempotency
- Implement proper error handling and retry logic

## 3. Shared Memory Integration

### High-Performance Communication
**Direct memory access for performance-critical applications**

#### Use Cases
- **High-Frequency Trading**: Microsecond-level latency requirements
- **Real-time Processing**: Audio/video processing pipelines
- **Game Development**: Communication between game engine and plugins

#### Implementation Considerations

**C++ Shared Memory Example**
```cpp
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <iostream>

struct SharedData {
    int counter;
    double values[100];
    char message[256];
};

class SharedMemoryManager {
private:
    int shm_fd;
    SharedData* shared_data;
    const char* shm_name = "/my_shared_memory";
    
public:
    bool create() {
        shm_fd = shm_open(shm_name, O_CREAT | O_RDWR, 0666);
        if (shm_fd == -1) {
            std::cerr << "Failed to create shared memory" << std::endl;
            return false;
        }
        
        ftruncate(shm_fd, sizeof(SharedData));
        
        shared_data = (SharedData*)mmap(
            NULL, sizeof(SharedData), 
            PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0
        );
        
        if (shared_data == MAP_FAILED) {
            std::cerr << "Failed to map shared memory" << std::endl;
            return false;
        }
        
        return true;
    }
    
    void writeData(const SharedData& data) {
        memcpy(shared_data, &data, sizeof(SharedData));
    }
    
    void readData(SharedData& data) {
        memcpy(&data, shared_data, sizeof(SharedData));
    }
    
    ~SharedMemoryManager() {
        if (shared_data != MAP_FAILED) {
            munmap(shared_data, sizeof(SharedData));
        }
        if (shm_fd != -1) {
            close(shm_fd);
        }
    }
};
```

**Python Access to Shared Memory**
```python
import mmap
import struct
import ctypes

class SharedMemoryAccess:
    def __init__(self, shm_name="/my_shared_memory"):
        self.shm_name = shm_name
        self.shm_fd = None
        self.shared_mem = None
        
    def open(self):
        self.shm_fd = os.open(self.shm_name, os.O_RDWR)
        self.shared_mem = mmap.mmap(
            self.shm_fd, 
            ctypes.sizeof(SharedData),
            access=mmap.ACCESS_WRITE
        )
        
    def read_data(self):
        # Read data from shared memory
        # Implementation depends on data structure
        pass
        
    def close(self):
        if self.shared_mem:
            self.shared_mem.close()
        if self.shm_fd:
            os.close(self.shm_fd)
```

#### Best Practices
- Use proper synchronization mechanisms (semaphores, mutexes)
- Implement memory safety checks
- Handle process crashes gracefully
- Consider memory alignment for performance
- Document memory layout thoroughly

## 4. Foreign Function Interface (FFI)

### Direct Language Interop
**Calling functions from one language directly in another**

#### Common FFI Implementations

**Python ctypes for C Libraries**
```python
from ctypes import *

# Load the shared library
lib = CDLL('./libmylibrary.so')

# Define function prototypes
lib.add_numbers.argtypes = [c_int, c_int]
lib.add_numbers.restype = c_int

lib.process_data.argtypes = [c_char_p, c_int]
lib.process_data.restype = c_void_p

# Use the functions
result = lib.add_numbers(5, 3)
print(f"5 + 3 = {result}")

data = b"Hello from Python"
result_ptr = lib.process_data(data, len(data))
```

**Java JNI for Native Code**
```java
public class NativeInterface {
    // Load the native library
    static {
        System.loadLibrary("mylibrary");
    }
    
    // Declare native methods
    public native int addNumbers(int a, int b);
    public native String processData(String input);
    
    public static void main(String[] args) {
        NativeInterface ni = new NativeInterface();
        
        int result = ni.addNumbers(5, 3);
        System.out.println("5 + 3 = " + result);
        
        String processed = ni.processData("Hello from Java");
        System.out.println("Processed: " + processed);
    }
}
```

**Rust FFI Example**
```rust
#[no_mangle]
pub extern "C" fn add_numbers(a: i32, b: i32) -> i32 {
    a + b
}

#[no_mangle]
pub extern "C" fn process_string(input: *const libc::c_char) -> *mut libc::c_char {
    let c_str = unsafe { std::ffi::CStr::from_ptr(input) };
    let r_str = c_str.to_str().unwrap_or("");
    
    let processed = format!("Processed: {}", r_str);
    let c_processed = std::ffi::CString::new(processed).unwrap();
    c_processed.into_raw()
}

#[no_mangle]
pub extern "C" fn free_string(s: *mut libc::c_char) {
    unsafe {
        if !s.is_null() {
            let _ = std::ffi::CString::from_raw(s);
        }
    }
}
```

#### Best Practices
- Handle memory management carefully
- Use proper error handling across language boundaries
- Consider performance implications of marshaling data
- Document FFI interfaces thoroughly
- Test FFI boundaries extensively

## Pattern Selection Guidelines

### Choose API-Based Integration When:
- Services are developed by different teams
- You need language-agnostic communication
- Scalability and fault isolation are priorities
- You have well-defined service boundaries

### Choose Message Queues When:
- You need asynchronous processing
- Services need to be decoupled
- You require load balancing and buffering
- Event-driven architecture is appropriate

### Choose Shared Memory When:
- Performance is critical (microsecond latency)
- Components run on the same machine
- You have tight coupling requirements
- Memory overhead must be minimized

### Choose FFI When:
- You need to leverage existing native libraries
- Performance is critical for specific operations
- You have small, well-defined interfaces
- Memory safety can be carefully managed

## Conclusion

The choice of integration pattern depends on your specific requirements, team structure, and performance needs. Most real-world systems use a combination of these patterns, applying the right approach for each use case.

Understanding the trade-offs between these patterns enables you to design systems that are both performant and maintainable, leveraging the strengths of multiple programming languages effectively.