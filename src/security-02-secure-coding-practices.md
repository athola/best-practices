# Secure Coding Practices

## Introduction

Secure coding practices are essential for developing software that can withstand malicious attacks and protect sensitive data. This section covers fundamental secure coding techniques, common vulnerabilities, and best practices for writing secure code across different programming languages and platforms.

## Input Validation and Output Encoding

### Input Validation Principles

**Validate All Input**
- Never trust input from external sources
- Validate for type, length, format, and range
- Use allowlists rather than blocklists
- Validate on both client and server sides

**Common Input Validation Techniques**
```python
# Python input validation example
import re

def validate_email(email):
    if not email or len(email) > 254:
        return False
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(pattern, email) is not None

def validate_integer_input(value, min_val=0, max_val=100):
    try:
        num = int(value)
        return min_val <= num <= max_val
    except (ValueError, TypeError):
        return False
```

**File Path Validation**
```java
// Java file path validation
import java.nio.file.Path;
import java.nio.file.Paths;

public boolean isSafeFilePath(String userPath, String baseDir) {
    try {
        Path basePath = Paths.get(baseDir).normalize();
        Path userPathObj = Paths.get(baseDir, userPath).normalize();
        return userPathObj.startsWith(basePath);
    } catch (Exception e) {
        return false;
    }
}
```

### Output Encoding

**Context-Aware Encoding**
- HTML encoding for web applications
- URL encoding for parameters
- JavaScript encoding for dynamic content
- CSS encoding for style attributes

**HTML Encoding Example**
```javascript
// JavaScript HTML encoding
function htmlEncode(str) {
    return str.replace(/&/g, '&amp;')
              .replace(/</g, '&lt;')
              .replace(/>/g, '&gt;')
              .replace(/"/g, '&quot;')
              .replace(/'/g, '&#39;');
}

// Usage in React
function SafeComponent({ userContent }) {
    return <div dangerouslySetInnerHTML={{ 
        __html: htmlEncode(userContent) 
    }} />;
}
```

## Authentication and Session Management

### Secure Authentication

**Password Storage**
```python
# Python password hashing with bcrypt
import bcrypt

def hash_password(password):
    salt = bcrypt.gensalt()
    return bcrypt.hashpw(password.encode('utf-8'), salt)

def verify_password(password, hashed_password):
    return bcrypt.checkpw(password.encode('utf-8'), hashed_password)
```

**Multi-Factor Authentication**
```python
# Python TOTP implementation
import pyotp
import qrcode

def generate_totp_secret():
    return pyotp.random_base32()

def generate_totp_qrcode(secret, username):
    totp = pyotp.TOTP(secret)
    provisioning_uri = totp.provisioning_uri(username, issuer_name="YourApp")
    return qrcode.make(provisioning_uri)

def verify_totp(secret, token):
    totp = pyotp.TOTP(secret)
    return totp.verify(token, valid_window=1)
```

### Session Management

**Secure Session Handling**
```python
# Python Flask session security
from flask import Flask, session
import secrets
import datetime

app = Flask(__name__)
app.secret_key = secrets.token_hex(32)  # Generate secure secret key

@app.before_request
def make_session_permanent():
    session.permanent = True
    app.permanent_session_lifetime = datetime.timedelta(minutes=30)

# Secure session configuration
app.config.update(
    SESSION_COOKIE_SECURE=True,    # Only send over HTTPS
    SESSION_COOKIE_HTTPONLY=True,  # Prevent JavaScript access
    SESSION_COOKIE_SAMESITE='Lax'  # CSRF protection
)
```

**Session Token Generation**
```javascript
// Node.js secure session token
const crypto = require('crypto');

function generateSessionToken() {
    return crypto.randomBytes(32).toString('hex');
}

function validateSessionToken(token) {
    // Validate token format and expiration
    if (!token || typeof token !== 'string' || token.length !== 64) {
        return false;
    }
    
    // Check against database and expiration
    return checkTokenInDatabase(token);
}
```

## Authorization and Access Control

### Role-Based Access Control (RBAC)

**RBAC Implementation**
```python
# Python RBAC implementation
from enum import Enum
from functools import wraps

class Role(Enum):
    ADMIN = "admin"
    USER = "user"
    GUEST = "guest"

class User:
    def __init__(self, username, role):
        self.username = username
        self.role = role

def require_role(required_role):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            if not hasattr(g, 'user') or g.user.role != required_role:
                return "Access denied", 403
            return f(*args, **kwargs)
        return decorated_function
    return decorator

# Usage
@app.route('/admin')
@require_role(Role.ADMIN)
def admin_panel():
    return "Admin panel"
```

### Attribute-Based Access Control (ABAC)

**ABAC Implementation**
```java
// Java ABAC example
public class AccessController {
    public boolean checkAccess(User user, Resource resource, Action action) {
        // Check user attributes
        if (user.getDepartment().equals("HR") && 
            resource.getType().equals("employee_record")) {
            return action.equals(Action.READ);
        }
        
        // Check time-based access
        if (user.getRole().equals("CONTRACTOR") && 
            !isBusinessHours()) {
            return false;
        }
        
        // Check location-based access
        if (resource.getSensitivity().equals("HIGH") && 
            !user.getLocation().equals("OFFICE")) {
            return false;
        }
        
        return true;
    }
}
```

## Cryptography Best Practices

### Encryption Implementation

**Symmetric Encryption**
```python
# Python AES encryption
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
import base64
import os

def generate_key(password: str, salt: bytes = None) -> bytes:
    if salt is None:
        salt = os.urandom(16)
    
    kdf = PBKDF2HMAC(
        algorithm=hashes.SHA256(),
        length=32,
        salt=salt,
        iterations=100000,
    )
    key = base64.urlsafe_b64encode(kdf.derive(password.encode()))
    return key

def encrypt_data(data: str, key: bytes) -> bytes:
    f = Fernet(key)
    return f.encrypt(data.encode())

def decrypt_data(encrypted_data: bytes, key: bytes) -> str:
    f = Fernet(key)
    return f.decrypt(encrypted_data).decode()
```

**Asymmetric Encryption**
```java
// Java RSA encryption
import java.security.*;
import javax.crypto.*;
import java.util.Base64;

public class RSAEncryption {
    private KeyPair keyPair;
    
    public RSAEncryption() throws NoSuchAlgorithmException {
        KeyPairGenerator keyGen = KeyPairGenerator.getInstance("RSA");
        keyGen.initialize(2048);
        this.keyPair = keyGen.generateKeyPair();
    }
    
    public String encrypt(String plaintext) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA/ECB/OAEPWithSHA-256AndMGF1Padding");
        cipher.init(Cipher.ENCRYPT_MODE, keyPair.getPublic());
        byte[] encryptedBytes = cipher.doFinal(plaintext.getBytes());
        return Base64.getEncoder().encodeToString(encryptedBytes);
    }
    
    public String decrypt(String ciphertext) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA/ECB/OAEPWithSHA-256AndMGF1Padding");
        cipher.init(Cipher.DECRYPT_MODE, keyPair.getPrivate());
        byte[] decryptedBytes = cipher.doFinal(Base64.getDecoder().decode(ciphertext));
        return new String(decryptedBytes);
    }
}
```

### Hashing and Digital Signatures

**Secure Hashing**
```python
# Python secure hashing
import hashlib
import hmac

def secure_hash(data: str, salt: str = "") -> str:
    """SHA-256 with salt"""
    return hashlib.sha256((data + salt).encode()).hexdigest()

def hmac_sha256(data: str, key: str) -> str:
    """HMAC for message authentication"""
    return hmac.new(key.encode(), data.encode(), hashlib.sha256).hexdigest()

def verify_integrity(data: str, expected_hash: str, salt: str = "") -> bool:
    """Verify data integrity"""
    actual_hash = secure_hash(data, salt)
    return hmac.compare_digest(actual_hash, expected_hash)
```

## Secure Error Handling and Logging

### Secure Error Handling

**Error Handling Best Practices**
```python
# Python secure error handling
import logging
from flask import jsonify

def handle_error(error):
    # Log detailed error for debugging
    logging.error(f"Error occurred: {str(error)}", exc_info=True)
    
    # Return generic error to client
    if isinstance(error, ValueError):
        return jsonify({"error": "Invalid input provided"}), 400
    elif isinstance(error, PermissionError):
        return jsonify({"error": "Access denied"}), 403
    else:
        return jsonify({"error": "An unexpected error occurred"}), 500

# Usage in API endpoint
@app.route('/api/data')
def get_data():
    try:
        # Process request
        result = process_request()
        return jsonify({"data": result})
    except Exception as e:
        return handle_error(e)
```

### Secure Logging

**Logging Best Practices**
```python
# Python secure logging
import logging
from logging.handlers import RotatingFileHandler
import json

class SecureFormatter(logging.Formatter):
    def format(self, record):
        # Remove sensitive data from log messages
        message = super().format(record)
        
        # Redact sensitive patterns
        sensitive_patterns = [
            r'password=[^&\s]+',
            r'token=[^&\s]+',
            r'api_key=[^&\s]+',
            r'credit_card=\d+'
        ]
        
        for pattern in sensitive_patterns:
            message = re.sub(pattern, '[REDACTED]', message)
        
        return message

# Configure secure logging
def setup_secure_logging():
    logger = logging.getLogger('secure_app')
    logger.setLevel(logging.INFO)
    
    # File handler with rotation
    file_handler = RotatingFileHandler(
        'secure_app.log',
        maxBytes=10485760,  # 10MB
        backupCount=5
    )
    file_handler.setFormatter(SecureFormatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    ))
    
    logger.addHandler(file_handler)
    return logger
```

## Memory Safety and Buffer Overflow Prevention

### Buffer Overflow Prevention

**Safe String Operations**
```c
// C safe string operations
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

void safe_string_copy(char *dest, const char *src, size_t dest_size) {
    if (dest == NULL || src == NULL || dest_size == 0) {
        return;
    }
    
    strncpy(dest, src, dest_size - 1);
    dest[dest_size - 1] = '\0';
}

int safe_string_concat(char *dest, const char *src, size_t dest_size) {
    if (dest == NULL || src == NULL || dest_size == 0) {
        return -1;
    }
    
    size_t dest_len = strlen(dest);
    if (dest_len >= dest_size - 1) {
        return -1;
    }
    
    strncat(dest, src, dest_size - dest_len - 1);
    return 0;
}
```

**Memory Management in C++**
```cpp
// C++ safe memory management
#include <memory>
#include <vector>
#include <string>

class SafeDataProcessor {
private:
    std::vector<std::string> data;
    std::unique_ptr<int[]> buffer;
    size_t buffer_size;
    
public:
    SafeDataProcessor(size_t size) : buffer_size(size) {
        buffer = std::make_unique<int[]>(size);
    }
    
    void addData(const std::string& item) {
        // Automatic memory management with vector
        data.push_back(item);
    }
    
    bool processBuffer(size_t index, int value) {
        if (index >= buffer_size) {
            return false;  // Bounds checking
        }
        buffer[index] = value;
        return true;
    }
    
    // No need for manual cleanup - RAII handles it
};
```

### Safe Integer Operations

**Integer Overflow Prevention**
```python
# Python safe integer operations
import sys

def safe_add(a, b):
    """Safe addition with overflow checking"""
    result = a + b
    
    # Check for overflow
    if (a > 0 and b > 0 and result < 0) or (a < 0 and b < 0 and result > 0):
        raise OverflowError("Integer overflow detected")
    
    return result

def safe_multiply(a, b):
    """Safe multiplication with overflow checking"""
    if a == 0 or b == 0:
        return 0
    
    result = a * b
    
    # Check for overflow
    if (a != 0 and result // a != b):
        raise OverflowError("Integer overflow detected")
    
    return result

def safe_divide(a, b):
    """Safe division with zero division and overflow checking"""
    if b == 0:
        raise ZeroDivisionError("Division by zero")
    
    # Check for potential overflow
    if a == sys.minsize and b == -1:
        raise OverflowError("Integer overflow detected")
    
    return a // b
```

## Common Vulnerabilities and Prevention

### SQL Injection Prevention

**Parameterized Queries**
```python
# Python parameterized queries with SQLAlchemy
from sqlalchemy import create_engine, text
from sqlalchemy.orm import sessionmaker

class SecureDatabase:
    def __init__(self, connection_string):
        self.engine = create_engine(connection_string)
        self.Session = sessionmaker(bind=self.engine)
    
    def get_user_by_id(self, user_id):
        """Safe query with parameterization"""
        session = self.Session()
        try:
            query = text("SELECT * FROM users WHERE id = :user_id")
            result = session.execute(query, {'user_id': user_id})
            return result.fetchone()
        finally:
            session.close()
    
    def search_users(self, search_term):
        """Safe search with parameterization"""
        session = self.Session()
        try:
            query = text("SELECT * FROM users WHERE username LIKE :search_term")
            search_pattern = f"%{search_term}%"
            result = session.execute(query, {'search_term': search_pattern})
            return result.fetchall()
        finally:
            session.close()
```

### Cross-Site Scripting (XSS) Prevention

**XSS Prevention Techniques**
```javascript
// React XSS prevention
import React from 'react';
import DOMPurify from 'dompurify';

function SafeContent({ content }) {
    // Sanitize HTML content
    const cleanHTML = DOMPurify.sanitize(content, {
        ALLOWED_TAGS: ['b', 'i', 'u', 'p', 'br', 'ul', 'ol', 'li'],
        ALLOWED_ATTR: []
    });
    
    return (
        <div 
            className="safe-content"
            dangerouslySetInnerHTML={{ __html: cleanHTML }}
        />
    );
}

// Safe URL handling
function SafeLink({ url, children }) {
    const validateUrl = (url) => {
        try {
            const parsedUrl = new URL(url);
            return ['http:', 'https:'].includes(parsedUrl.protocol);
        } catch {
            return false;
        }
    };
    
    if (!validateUrl(url)) {
        return <span className="invalid-link">{children}</span>;
    }
    
    return (
        <a href={url} target="_blank" rel="noopener noreferrer">
            {children}
        </a>
    );
}
```

### Cross-Site Request Forgery (CSRF) Prevention

**CSRF Protection**
```python
# Python Flask CSRF protection
from flask import Flask, request, session
import secrets
import hashlib

app = Flask(__name__)
app.secret_key = secrets.token_hex(32)

def generate_csrf_token():
    if 'csrf_token' not in session:
        session['csrf_token'] = secrets.token_hex(32)
    return session['csrf_token']

def validate_csrf_token():
    token = request.form.get('csrf_token')
    if not token or token != session.get('csrf_token'):
        return False
    return True

@app.route('/transfer', methods=['POST'])
def transfer_money():
    if not validate_csrf_token():
        return "CSRF token validation failed", 403
    
    # Process transfer
    amount = request.form.get('amount')
    recipient = request.form.get('recipient')
    
    # Transfer logic here
    return "Transfer successful"

# Template usage
@app.route('/transfer_form')
def transfer_form():
    csrf_token = generate_csrf_token()
    return f'''
    <form method="POST" action="/transfer">
        <input type="hidden" name="csrf_token" value="{csrf_token}">
        <input type="number" name="amount" placeholder="Amount">
        <input type="text" name="recipient" placeholder="Recipient">
        <button type="submit">Transfer</button>
    </form>
    '''
```

## Security Testing and Code Review

### Security Code Review Checklist

**Authentication and Authorization**
- [ ] Passwords are properly hashed and salted
- [ ] Session tokens are generated using cryptographically secure RNG
- [ ] Session timeout is implemented
- [ ] Access controls are enforced on server-side
- [ ] Privilege escalation is prevented

**Input Validation and Output Encoding**
- [ ] All user input is validated
- [ ] Output is properly encoded based on context
- [ ] File path validation is implemented
- [ ] Parameterized queries are used for database access
- [ ] SQL injection prevention is in place

**Data Protection**
- [ ] Sensitive data is encrypted at rest and in transit
- [ ] Proper key management is implemented
- [ ] Data retention policies are followed
- [ ] Backup data is protected
- [ ] Secure deletion of sensitive data

**Error Handling and Logging**
- [ ] Detailed errors are not exposed to users
- [ ] Sensitive data is not logged
- [ ] Proper error handling is implemented
- [ ] Security events are logged
- [ ] Log files are protected

### Security Testing Tools

**Static Application Security Testing (SAST)**
```bash
# Semgrep security scanning
semgrep --config=auto --severity=ERROR ./src/

# Bandit Python security scanning
bandit -r ./src/

# SonarQube security analysis
sonar-scanner -Dsonar.projectKey=myproject -Dsonar.sources=./src
```

**Dynamic Application Security Testing (DAST)**
```bash
# OWASP ZAP scanning
zap-baseline.py -t https://localhost:3000 -r zap_report.html

# Nmap security scanning
nmap -sV --script vuln localhost

# SQLMap testing
sqlmap -u "https://localhost:3000/search?q=test" --batch
```

## Conclusion

Secure coding practices are fundamental to building resilient software systems that can withstand malicious attacks and protect sensitive data. By implementing proper input validation, secure authentication and authorization, robust cryptography, and comprehensive error handling, developers can significantly reduce the attack surface of their applications.

Security must be integrated throughout the development lifecycle, from initial design through deployment and maintenance. Regular security testing, code reviews, and staying informed about emerging threats are essential for maintaining strong security postures in an evolving threat landscape.

Remember that security is not a one-time implementation but a continuous process of improvement and adaptation to new challenges and vulnerabilities.
