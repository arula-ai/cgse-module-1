# CGSE-A Lab 1: Build Instructions for Claude Code

## Overview

This document provides instructions for Claude Code to build the complete Lab 1 hands-on environment for the CGSE-Advanced Module 1: AI-Assisted Development Fundamentals.

**Output:** Two fully functional projects (Angular and Java/Spring Boot) with intentional security vulnerabilities for students to find and fix.

---

## Project Requirements

### Session Format
- 20 min instruction + 60 min hands-on lab + 10 min Q&A
- Students use GitHub Copilot to find/fix vulnerabilities
- Target: 70%+ test coverage on AuthService

### Learning Objectives
1. Apply RIFCC framework for AI prompts
2. Identify security vulnerabilities (OWASP Top 10)
3. Execute "Critique then Create" pattern
4. Generate tests with AI assistance

---

## PART 1: Angular Project Build

### 1.1 Project Initialization

Create an Angular 19+ project with the following structure:

```
cgse-lab1-angular/
├── src/
│   ├── app/
│   │   ├── components/
│   │   │   └── user-detail/
│   │   │       ├── user-detail.component.ts
│   │   │       ├── user-detail.component.html
│   │   │       └── user-detail.component.spec.ts
│   │   ├── services/
│   │   │   ├── user.service.ts          # Vulnerable - students fix
│   │   │   ├── user.service.spec.ts     # Minimal starter tests
│   │   │   ├── auth.service.ts          # Vulnerable - PRIMARY FOCUS
│   │   │   └── auth.service.spec.ts     # Minimal starter tests
│   │   ├── models/
│   │   │   └── user.model.ts
│   │   ├── app.component.ts
│   │   ├── app.config.ts
│   │   └── app.routes.ts
│   ├── environments/
│   │   ├── environment.ts
│   │   └── environment.prod.ts
│   └── main.ts
├── docs/                                 # Empty templates for students
│   ├── SECURITY_AUDIT.md
│   ├── AUTH_SERVICE_CRITIQUE.md
│   └── AI_USAGE_LOG.md
├── angular.json
├── package.json
├── tsconfig.json
├── karma.conf.js
└── README.md
```

### 1.2 Package.json Dependencies

```json
{
  "name": "cgse-lab1-angular",
  "version": "1.0.0",
  "scripts": {
    "start": "ng serve",
    "build": "ng build",
    "test": "ng test",
    "test:ci": "ng test --no-watch --code-coverage",
    "lint": "ng lint"
  },
  "dependencies": {
    "@angular/animations": "^17.0.0",
    "@angular/common": "^17.0.0",
    "@angular/compiler": "^17.0.0",
    "@angular/core": "^17.0.0",
    "@angular/forms": "^17.0.0",
    "@angular/platform-browser": "^17.0.0",
    "@angular/platform-browser-dynamic": "^17.0.0",
    "@angular/router": "^17.0.0",
    "rxjs": "~7.8.0",
    "tslib": "^2.3.0",
    "zone.js": "~0.14.0"
  },
  "devDependencies": {
    "@angular-devkit/build-angular": "^17.0.0",
    "@angular/cli": "^17.0.0",
    "@angular/compiler-cli": "^17.0.0",
    "@types/jasmine": "~5.1.0",
    "jasmine-core": "~5.1.0",
    "karma": "~6.4.0",
    "karma-chrome-launcher": "~3.2.0",
    "karma-coverage": "~2.2.0",
    "karma-jasmine": "~5.1.0",
    "karma-jasmine-html-reporter": "~2.1.0",
    "typescript": "~5.2.0"
  }
}
```

### 1.3 Vulnerable User Service (user.service.ts)

Create with these INTENTIONAL vulnerabilities:

```typescript
/**
 * CGSE-A Lab 1 - Angular Starter Code
 * UserService - Contains INTENTIONAL security vulnerabilities
 * 
 * ⚠️ WARNING: DO NOT use in production!
 * 
 * VULNERABILITIES TO PLANT:
 * 1. Line ~25: URL parameter injection (string interpolation)
 * 2. Line ~35: PII logging to console
 * 3. Line ~45: Mass assignment (accepts any fields)
 * 4. Line ~55: User enumeration endpoint
 * 5. Line ~65: No error handling
 */

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { User } from '../models/user.model';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private apiUrl = 'http://localhost:3000/api';

  constructor(private http: HttpClient) {}

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(`${this.apiUrl}/users`);
  }

  getUserById(id: string): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/users/${id}`);
  }

  // VULNERABILITY 1: SQL-like injection via URL parameter
  // Direct string interpolation allows injection attacks
  getUserByFilter(filter: string): Observable<User[]> {
    // ❌ VULNERABLE: Direct interpolation
    return this.http.get<User[]>(`${this.apiUrl}/users?filter=${filter}`);
  }

  // VULNERABILITY 2: PII logging
  searchUsers(query: string): Observable<User[]> {
    console.log('Search query:', query); // ❌ May log PII
    return this.http.get<User[]>(`${this.apiUrl}/users/search?q=${query}`);
  }

  // VULNERABILITY 3: No input validation, PII logged
  createUser(user: User): Observable<User> {
    console.log('Creating user:', user); // ❌ Logs entire user object including password
    return this.http.post<User>(`${this.apiUrl}/users`, user);
  }

  // VULNERABILITY 4: Mass assignment - accepts any fields
  updateUser(id: string, user: Partial<User>): Observable<User> {
    // ❌ Could update role, isAdmin, etc.
    return this.http.put<User>(`${this.apiUrl}/users/${id}`, user);
  }

  deleteUser(id: string): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/users/${id}`);
  }

  // VULNERABILITY 5: Information disclosure - user enumeration
  checkEmailExists(email: string): Observable<boolean> {
    // ❌ Allows attackers to enumerate valid emails
    return this.http.get<boolean>(`${this.apiUrl}/users/check-email?email=${email}`);
  }
}
```

### 1.4 Vulnerable Auth Service (auth.service.ts) - PRIMARY FOCUS

Create with these INTENTIONAL vulnerabilities:

```typescript
/**
 * CGSE-A Lab 1 - Angular Starter Code
 * AuthService - PRIMARY FOCUS for Critique & Create
 * 
 * ⚠️ WARNING: Contains INTENTIONAL security vulnerabilities!
 * 
 * VULNERABILITIES TO PLANT:
 * 1. Line ~30: Token stored in localStorage (XSS risk)
 * 2. Line ~35: 'any' types used (type safety)
 * 3. Line ~45: Token and user logged to console
 * 4. Line ~60: No token validation (only checks existence)
 * 5. Line ~70: No subscription cleanup (memory leak)
 * 6. Line ~80: Hardcoded token expiry magic number
 * 7. Line ~90: Incomplete logout (doesn't invalidate server session)
 * 8. Line ~100: JWT parsed without signature validation
 */

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { BehaviorSubject, Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

interface LoginResponse {
  token: string;
  user: any; // ❌ VULNERABILITY: Using 'any' type
}

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  private apiUrl = 'http://localhost:3000/api';
  private token: string | null = null;
  
  // ❌ VULNERABILITY: Using 'any' for user
  private currentUserSubject = new BehaviorSubject<any>(null);
  currentUser$ = this.currentUserSubject.asObservable();

  // ❌ VULNERABILITY: Hardcoded magic number
  private readonly TOKEN_EXPIRY_MS = 3600000;

  constructor(private http: HttpClient) {
    this.loadTokenFromStorage();
  }

  login(email: string, password: string): Observable<LoginResponse> {
    return this.http.post<LoginResponse>(`${this.apiUrl}/auth/login`, {
      email,
      password
    }).pipe(
      tap(response => {
        // ❌ VULNERABILITY: Storing in localStorage (XSS risk)
        localStorage.setItem('auth_token', response.token);
        localStorage.setItem('user', JSON.stringify(response.user));
        
        this.token = response.token;
        this.currentUserSubject.next(response.user);
        
        // ❌ VULNERABILITY: Logging sensitive data
        console.log('User logged in:', response.user);
        console.log('Token:', response.token);
      })
    );
  }

  register(email: string, password: string, name: string): Observable<LoginResponse> {
    // ❌ VULNERABILITY: No password strength validation
    // ❌ VULNERABILITY: No email format validation
    return this.http.post<LoginResponse>(`${this.apiUrl}/auth/register`, {
      email,
      password,
      name
    }).pipe(
      tap(response => {
        localStorage.setItem('auth_token', response.token);
        this.token = response.token;
        this.currentUserSubject.next(response.user);
      })
    );
  }

  // ❌ VULNERABILITY: Incomplete logout - doesn't invalidate server session
  logout(): void {
    localStorage.removeItem('auth_token');
    localStorage.removeItem('user');
    this.token = null;
    this.currentUserSubject.next(null);
    // Missing: Server-side session invalidation
    // Missing: Navigation
  }

  getToken(): string | null {
    // ❌ VULNERABILITY: No expiry check
    return this.token || localStorage.getItem('auth_token');
  }

  // ❌ VULNERABILITY: Only checks if token exists, not if valid
  isAuthenticated(): boolean {
    const token = this.getToken();
    return !!token;
  }

  // ❌ VULNERABILITY: Parses JWT without signature validation
  isTokenExpired(): boolean {
    const token = this.getToken();
    if (!token) return true;

    try {
      // ❌ VULNERABLE: No signature verification
      const payload = JSON.parse(atob(token.split('.')[1]));
      const expiry = payload.exp * 1000;
      return Date.now() > expiry;
    } catch {
      return true;
    }
  }

  // ❌ NOT IMPLEMENTED - causes session timeout issues
  refreshToken(): Observable<LoginResponse> {
    throw new Error('Not implemented');
  }

  // ❌ VULNERABILITY: Trusts localStorage without validation
  private loadTokenFromStorage(): void {
    const token = localStorage.getItem('auth_token');
    const userStr = localStorage.getItem('user');
    
    if (token) {
      this.token = token;
    }
    
    if (userStr) {
      try {
        const user = JSON.parse(userStr);
        this.currentUserSubject.next(user);
      } catch (e) {
        console.error('Failed to parse user data');
      }
    }
  }

  changePassword(oldPassword: string, newPassword: string): Observable<void> {
    // ❌ VULNERABILITY: No password strength validation
    return this.http.post<void>(`${this.apiUrl}/auth/change-password`, {
      oldPassword,
      newPassword
    });
  }
}
```

### 1.5 User Model (user.model.ts)

```typescript
export interface User {
  id: string;
  email: string;
  name: string;
  password?: string;
  role?: string;
  isAdmin?: boolean;
  createdAt?: Date;
  updatedAt?: Date;
}

export interface UserCreateRequest {
  email: string;
  password: string;
  name: string;
}

export interface UserUpdateRequest {
  name?: string;
  email?: string;
}
```

### 1.6 Minimal Starter Test (auth.service.spec.ts)

Provide a basic test file that students will expand:

```typescript
import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { AuthService } from './auth.service';

describe('AuthService', () => {
  let service: AuthService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    // Clear localStorage before each test
    localStorage.clear();
    
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [AuthService]
    });
    service = TestBed.inject(AuthService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify();
    localStorage.clear();
  });

  it('should be created', () => {
    expect(service).toBeTruthy();
  });

  // TODO: Students add more tests here using AI assistance
  // Target: >70% coverage on this service
  
  // Hint: Test these scenarios:
  // - login success
  // - login failure
  // - logout clears state
  // - isAuthenticated returns correct value
  // - token storage security
});
```

### 1.7 Environment Configuration

```typescript
// environment.ts
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000/api'
};

// environment.prod.ts
export const environment = {
  production: true,
  apiUrl: '/api'
};
```

### 1.8 Karma Configuration for Coverage

```javascript
// karma.conf.js
module.exports = function (config) {
  config.set({
    basePath: '',
    frameworks: ['jasmine', '@angular-devkit/build-angular'],
    plugins: [
      require('karma-jasmine'),
      require('karma-chrome-launcher'),
      require('karma-jasmine-html-reporter'),
      require('karma-coverage'),
      require('@angular-devkit/build-angular/plugins/karma')
    ],
    client: {
      jasmine: {},
      clearContext: false
    },
    jasmineHtmlReporter: {
      suppressAll: true
    },
    coverageReporter: {
      dir: require('path').join(__dirname, './coverage/cgse-lab1-angular'),
      subdir: '.',
      reporters: [
        { type: 'html' },
        { type: 'text-summary' },
        { type: 'lcov' }
      ],
      check: {
        global: {
          statements: 50,
          branches: 50,
          functions: 50,
          lines: 50
        }
      }
    },
    reporters: ['progress', 'kjhtml', 'coverage'],
    browsers: ['Chrome', 'ChromeHeadless'],
    restartOnFileChange: true
  });
};
```

### 1.9 Documentation Templates

Create empty templates in `/docs/`:

**SECURITY_AUDIT.md:**
```markdown
# Security Audit Report

## Service: UserService

### Vulnerabilities Found

| # | Line | Type (OWASP) | Severity | Description | Suggested Fix |
|---|------|--------------|----------|-------------|---------------|
| 1 |      |              |          |             |               |
| 2 |      |              |          |             |               |
| 3 |      |              |          |             |               |

## Service: AuthService

### Vulnerabilities Found

| # | Line | Type (OWASP) | Severity | Description | Suggested Fix |
|---|------|--------------|----------|-------------|---------------|
| 1 |      |              |          |             |               |
| 2 |      |              |          |             |               |
| 3 |      |              |          |             |               |

## Priority Remediation Order
1. 
2. 
3. 
```

**AUTH_SERVICE_CRITIQUE.md:**
```markdown
# Auth Service Critique

## Top 5 Issues Identified

### Issue 1: [Title]
- **Line:** 
- **Category:** Security / Quality / Performance / Testability
- **Description:** 
- **Recommended Fix:** 

### Issue 2: [Title]
- **Line:** 
- **Category:** 
- **Description:** 
- **Recommended Fix:** 

### Issue 3: [Title]
- **Line:** 
- **Category:** 
- **Description:** 
- **Recommended Fix:** 

### Issue 4: [Title]
- **Line:** 
- **Category:** 
- **Description:** 
- **Recommended Fix:** 

### Issue 5: [Title]
- **Line:** 
- **Category:** 
- **Description:** 
- **Recommended Fix:** 
```

**AI_USAGE_LOG.md:**
```markdown
# AI Usage Log - Lab 1

## Prompt 1: Security Audit
- **Purpose:** 
- **RIFCC Components Used:** R☐ I☐ F☐ C☐ C☐
- **AI Output Quality (1-5):** 
- **Modifications Made:** 

## Prompt 2: Critique
- **Purpose:** 
- **RIFCC Components Used:** R☐ I☐ F☐ C☐ C☐
- **AI Output Quality (1-5):** 
- **Modifications Made:** 

## Prompt 3: Create/Refactor
- **Purpose:** 
- **RIFCC Components Used:** R☐ I☐ F☐ C☐ C☐
- **AI Output Quality (1-5):** 
- **Modifications Made:** 

## Prompt 4: Test Generation
- **Purpose:** 
- **RIFCC Components Used:** R☐ I☐ F☐ C☐ C☐
- **AI Output Quality (1-5):** 
- **Modifications Made:** 

## Key Learnings
1. What worked well with AI assistance:
2. What required human judgment:
3. Security issues AI missed:

## Time Estimate
- Without AI: ~X hours
- With AI: ~Y hours
- Efficiency gain: ~Z%
```

### 1.10 README.md

```markdown
# CGSE-A Lab 1: AI-Assisted Development Fundamentals (Angular)

## ⚠️ Educational Purpose Only

This project contains **intentional security vulnerabilities** for training purposes.
**DO NOT** use this code in production!

## Quick Start

```bash
# Install dependencies
npm install

# Start development server
npm start

# Run tests with coverage
npm run test:ci
```

## Lab Tasks

1. **Security Audit** (15 min) - Find vulnerabilities in `user.service.ts`
2. **Critique & Create** (20 min) - Refactor `auth.service.ts`
3. **Test Coverage** (15 min) - Achieve >70% coverage
4. **Documentation** (10 min) - Complete docs in `/docs`

## Success Criteria

- [ ] 3+ vulnerabilities documented
- [ ] AuthService refactored (no localStorage)
- [ ] >70% test coverage
- [ ] All documentation complete

## Resources

- [Angular Security Guide](https://angular.io/guide/security)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
```

---

## PART 2: Java 21/Spring Boot Project Build

### 2.1 Project Structure

```
cgse-lab1-java/
├── src/
│   ├── main/
│   │   ├── java/com/arula/lab1/
│   │   │   ├── Lab1Application.java
│   │   │   ├── config/
│   │   │   │   └── SecurityConfig.java
│   │   │   ├── controller/
│   │   │   │   └── UserController.java
│   │   │   ├── service/
│   │   │   │   ├── UserService.java       # Vulnerable
│   │   │   │   └── AuthService.java       # PRIMARY FOCUS
│   │   │   ├── repository/
│   │   │   │   └── UserRepository.java
│   │   │   ├── model/
│   │   │   │   └── User.java
│   │   │   └── exception/
│   │   │       └── UserNotFoundException.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── application-test.yml
│   └── test/
│       └── java/com/arula/lab1/
│           └── service/
│               ├── UserServiceTest.java    # Minimal starter
│               └── AuthServiceTest.java    # Minimal starter
├── docs/                                   # Empty templates
│   ├── SECURITY_AUDIT.md
│   ├── AUTH_SERVICE_CRITIQUE.md
│   └── AI_USAGE_LOG.md
├── pom.xml
└── README.md
```

### 2.2 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>
    
    <groupId>com.arula</groupId>
    <artifactId>cgse-lab1-java</artifactId>
    <version>1.0.0</version>
    <name>CGSE Lab 1 - Java</name>
    <description>AI-Assisted Development Fundamentals Lab</description>
    
    <properties>
        <java.version>17</java.version>
        <jjwt.version>0.12.3</jjwt.version>
    </properties>
    
    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <!-- Database -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- JWT -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>${jjwt.version}</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>${jjwt.version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>${jjwt.version}</version>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
            
            <!-- JaCoCo for coverage -->
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.11</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>prepare-agent</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>report</id>
                        <phase>test</phase>
                        <goals>
                            <goal>report</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

### 2.3 Vulnerable UserService.java

```java
/**
 * CGSE-A Lab 1 - Java/Spring Boot Starter Code
 * UserService - Contains INTENTIONAL security vulnerabilities
 * 
 * ⚠️ WARNING: DO NOT use in production!
 * 
 * VULNERABILITIES TO PLANT:
 * 1. Line ~45: SQL Injection via JPQL string concatenation
 * 2. Line ~60: SQL Injection via LIKE clause
 * 3. Line ~75: PII/Password logging
 * 4. Line ~90: Mass assignment via BeanUtils.copyProperties
 * 5. Line ~105: Timing attack in credential verification
 * 6. Line ~120: Plain text password comparison
 */

package com.arula.lab1.service;

import com.arula.lab1.model.User;
import com.arula.lab1.repository.UserRepository;
import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import org.springframework.beans.BeanUtils;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class UserService {

    private final UserRepository userRepository;
    
    @PersistenceContext
    private EntityManager entityManager;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public List<User> findAll() {
        return userRepository.findAll();
    }

    public User findById(Long id) {
        return userRepository.findById(id).orElse(null);
    }

    // ❌ VULNERABILITY 1: SQL Injection via JPQL concatenation
    public User findByEmailUnsafe(String email) {
        String jpql = "SELECT u FROM User u WHERE u.email = '" + email + "'";
        try {
            return entityManager.createQuery(jpql, User.class).getSingleResult();
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage()); // Info disclosure
            return null;
        }
    }

    // ❌ VULNERABILITY 2: SQL Injection via LIKE clause
    public List<User> searchByName(String namePattern) {
        String jpql = "SELECT u FROM User u WHERE u.name LIKE '%" + namePattern + "%'";
        return entityManager.createQuery(jpql, User.class).getResultList();
    }

    // ❌ VULNERABILITY 3: PII and password logging
    public User createUser(User user) {
        System.out.println("Creating user: " + user.toString());
        System.out.println("Password: " + user.getPassword()); // CRITICAL!
        return userRepository.save(user);
    }

    // ❌ VULNERABILITY 4: Mass assignment
    public User updateUser(Long id, User updatedUser) {
        User user = findById(id);
        if (user == null) {
            throw new RuntimeException("User not found");
        }
        // Copies ALL fields including role, isAdmin, etc.
        BeanUtils.copyProperties(updatedUser, user, "id");
        return userRepository.save(user);
    }

    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }

    // ❌ VULNERABILITY 5: Timing attack
    // ❌ VULNERABILITY 6: Plain text password comparison
    public boolean verifyCredentials(String email, String password) {
        User user = userRepository.findByEmail(email).orElse(null);
        
        // Timing attack: fast return if user doesn't exist
        if (user == null) {
            return false;
        }
        
        // Plain text comparison (no hashing)
        return user.getPassword().equals(password);
    }

    // ❌ VULNERABILITY: User enumeration
    public boolean emailExists(String email) {
        return userRepository.existsByEmail(email);
    }
}
```

### 2.4 Vulnerable AuthService.java - PRIMARY FOCUS

```java
/**
 * CGSE-A Lab 1 - Java/Spring Boot Starter Code
 * AuthService - PRIMARY FOCUS for Critique & Create
 * 
 * ⚠️ WARNING: Contains INTENTIONAL security vulnerabilities!
 * 
 * VULNERABILITIES TO PLANT:
 * 1. Line ~30: Hardcoded weak JWT secret
 * 2. Line ~45: Plain text password storage
 * 3. Line ~60: Timing attack in authentication
 * 4. Line ~75: No old password verification in change
 * 5. Line ~90: Information disclosure in password reset
 * 6. Line ~105: JWT can't be invalidated (logout issue)
 */

package com.arula.lab1.service;

import com.arula.lab1.model.User;
import com.arula.lab1.repository.UserRepository;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.stereotype.Service;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

@Service
public class AuthService {

    private final UserRepository userRepository;
    
    // ❌ VULNERABILITY 1: Hardcoded weak secret
    private static final String JWT_SECRET = "secret123";
    
    // ❌ VULNERABILITY: Very long expiration
    private static final long JWT_EXPIRATION_MS = 30 * 24 * 60 * 60 * 1000L; // 30 days

    public AuthService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    // ❌ VULNERABILITY 2: Plain text password storage
    public User register(String email, String password, String name) {
        User user = new User();
        user.setEmail(email);
        user.setName(name);
        user.setPassword(password); // NOT HASHED!
        return userRepository.save(user);
    }

    // ❌ VULNERABILITY 3: Timing attack
    public String authenticate(String email, String password) {
        User user = userRepository.findByEmail(email).orElse(null);
        
        // Early return reveals user existence
        if (user == null) {
            return null;
        }
        
        // Plain text comparison
        if (!user.getPassword().equals(password)) {
            System.out.println("Failed login for: " + email); // Info disclosure
            return null;
        }
        
        return generateToken(user);
    }

    private String generateToken(User user) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("userId", user.getId());
        claims.put("email", user.getEmail());
        claims.put("role", user.getRole());
        
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(user.getId().toString())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + JWT_EXPIRATION_MS))
            .signWith(SignatureAlgorithm.HS256, JWT_SECRET)
            .compact();
    }

    public boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(JWT_SECRET).parseClaimsJws(token);
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    // ❌ VULNERABILITY 4: No old password verification
    public void changePassword(Long userId, String newPassword) {
        User user = userRepository.findById(userId).orElse(null);
        if (user != null) {
            user.setPassword(newPassword); // Not hashed, no old password check
            userRepository.save(user);
            System.out.println("Password changed for: " + user.getEmail());
        }
    }

    // ❌ VULNERABILITY 5: Information disclosure
    public boolean resetPassword(String email) {
        User user = userRepository.findByEmail(email).orElse(null);
        
        if (user == null) {
            System.out.println("Reset for non-existent: " + email);
            return false; // Reveals email doesn't exist
        }
        
        String tempPassword = "temp" + System.currentTimeMillis(); // Predictable!
        user.setPassword(tempPassword);
        userRepository.save(user);
        System.out.println("Temp password for " + email + ": " + tempPassword);
        return true;
    }

    // ❌ VULNERABILITY 6: JWT can't be invalidated
    public void logout(String token) {
        System.out.println("Logout called but token still valid!");
        // JWTs are stateless - can't invalidate without blacklist
    }
}
```

### 2.5 User.java Entity

```java
package com.arula.lab1.model;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

@Entity
@Table(name = "users")
@Data
@Builder(toBuilder = true)
@NoArgsConstructor
@AllArgsConstructor
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(nullable = false)
    private String name;
    
    @Column(nullable = false)
    private String password;
    
    @Column
    private String role = "USER";
    
    @Column
    private boolean isAdmin = false;
    
    @Column
    private boolean deleted = false;
    
    @Column
    private LocalDateTime deletedAt;
    
    @Column
    private LocalDateTime createdAt;
    
    @Column
    private LocalDateTime updatedAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
    
    // ❌ VULNERABILITY: Password included in toString()
    @Override
    public String toString() {
        return "User{id=" + id + ", email='" + email + "', name='" + name + 
               "', password='" + password + "', role='" + role + "'}";
    }
}
```

### 2.6 UserRepository.java

```java
package com.arula.lab1.repository;

import com.arula.lab1.model.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
}
```

### 2.7 Minimal Starter Test (AuthServiceTest.java)

```java
package com.arula.lab1.service;

import com.arula.lab1.model.User;
import com.arula.lab1.repository.UserRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.Optional;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
@DisplayName("AuthService Tests")
class AuthServiceTest {

    @Mock
    private UserRepository userRepository;
    
    private AuthService authService;
    
    private User testUser;

    @BeforeEach
    void setUp() {
        authService = new AuthService(userRepository);
        
        testUser = User.builder()
            .id(1L)
            .email("test@example.com")
            .name("Test User")
            .password("password123")
            .role("USER")
            .build();
    }

    @Test
    @DisplayName("should be instantiated")
    void shouldBeInstantiated() {
        assertThat(authService).isNotNull();
    }

    // TODO: Students add more tests here using AI assistance
    // Target: >70% coverage on AuthService
    
    // Hints for tests to add:
    // - authenticate with valid credentials
    // - authenticate with invalid credentials
    // - authenticate with non-existent user
    // - register creates user
    // - validateToken with valid token
    // - validateToken with invalid token
    // - changePassword updates password
}
```

### 2.8 Application Configuration

**application.yml:**
```yaml
spring:
  datasource:
    url: jdbc:h2:mem:labdb
    driver-class-name: org.h2.Driver
    username: sa
    password: 
  h2:
    console:
      enabled: true
      path: /h2-console
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        format_sql: true

jwt:
  secret: ${JWT_SECRET:thisIsAVeryLongSecretKeyThatShouldBeAtLeast256BitsForHS256Algorithm}
  expiration-ms: 3600000

logging:
  level:
    com.arula.lab1: DEBUG
    org.springframework.security: DEBUG
```

**application-test.yml:**
```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: false

jwt:
  secret: testSecretKeyThatIsAtLeast256BitsLongForTesting1234567890
  expiration-ms: 3600000
```

### 2.9 README.md for Java

```markdown
# CGSE-A Lab 1: AI-Assisted Development Fundamentals (Java)

## ⚠️ Educational Purpose Only

This project contains **intentional security vulnerabilities** for training purposes.
**DO NOT** use this code in production!

## Quick Start

```bash
# Build project
mvn clean install -DskipTests

# Run application
mvn spring-boot:run

# Run tests with coverage
mvn test jacoco:report

# View coverage report
open target/site/jacoco/index.html
```

## Lab Tasks

1. **Security Audit** (15 min) - Find vulnerabilities in `UserService.java`
2. **Critique & Create** (20 min) - Refactor `AuthService.java`
3. **Test Coverage** (15 min) - Achieve >70% coverage
4. **Documentation** (10 min) - Complete docs in `/docs`

## Success Criteria

- [ ] 3+ vulnerabilities documented (including SQL injection)
- [ ] AuthService refactored (BCrypt, secure JWT)
- [ ] >70% test coverage
- [ ] All documentation complete

## Resources

- [OWASP Java Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Java_Security_Cheat_Sheet.html)
- [Spring Security Reference](https://docs.spring.io/spring-security/reference/)
```

---

## PART 3: Verification Checklist

After building, verify:

### Angular Project
- [ ] `npm install` completes without errors
- [ ] `npm start` launches dev server
- [ ] `npm run test:ci` runs tests (some fail - expected)
- [ ] Coverage report generates in `/coverage`
- [ ] All vulnerable code has clear comments marking issues

### Java Project
- [ ] `mvn clean install -DskipTests` builds successfully
- [ ] `mvn spring-boot:run` starts application
- [ ] `mvn test` runs tests (some pass)
- [ ] `mvn jacoco:report` generates coverage report
- [ ] H2 console accessible at `/h2-console`
- [ ] All vulnerable code has clear comments marking issues

### Documentation
- [ ] `/docs` folder has all 3 template files
- [ ] README.md explains lab tasks clearly
- [ ] Vulnerability comments include line numbers

---

## PART 4: Git Repository Setup

Initialize both projects as Git repos:

```bash
# For each project
git init
git add .
git commit -m "Initial commit: CGSE Lab 1 starter code"

# Create .gitignore
# Angular:
node_modules/
dist/
coverage/
.angular/

# Java:
target/
*.class
*.jar
.idea/
*.iml
```

---

## Summary

This build instruction creates two parallel lab environments:

| Feature | Angular | Java |
|---------|---------|------|
| Framework | Angular 17+ | Spring Boot 3.2 |
| Vulnerabilities | 8+ planted | 8+ planted |
| Primary Focus | AuthService | AuthService |
| Test Framework | Jasmine/Karma | JUnit 5/Mockito |
| Coverage Tool | karma-coverage | JaCoCo |
| Target Coverage | 70%+ | 70%+ |

Students will use GitHub Copilot with RIFCC prompts to find, fix, and test these vulnerabilities in 60 minutes.
