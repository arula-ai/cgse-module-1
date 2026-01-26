# Generate Security Tests

Generate comprehensive unit tests with a focus on security scenarios.

## Instructions

Act as a testing expert specializing in security testing. Generate unit tests that:

1. **Verify Security Fixes** - Confirm vulnerabilities have been properly remediated
2. **Test Authentication Flows** - Login, logout, token validation, session handling
3. **Test Authorization** - Access control, privilege boundaries
4. **Test Input Validation** - Boundary conditions, malformed input, injection attempts
5. **Test Error Handling** - Graceful failures without information disclosure

## Test Categories Required

### Authentication Tests
- Valid credential authentication
- Invalid password rejection
- Non-existent user handling (timing attack prevention)
- Token generation and validation
- Token expiration handling
- Logout and session cleanup

### Security-Specific Tests
- Password hashing verification (not stored plain text)
- No sensitive data in logs (mock console/logger to verify)
- Proper error messages (no information disclosure)
- Input sanitization

### Edge Cases
- Empty/null inputs
- Extremely long inputs
- Special characters
- Concurrent requests (if applicable)

## Output Format

Provide complete, runnable test file with:
- All necessary imports
- Proper test setup (mocks, fixtures)
- Descriptive test names using `@DisplayName` or `describe/it` patterns
- Comments explaining security relevance of each test
- Both positive and negative test cases

## Framework-Specific Guidelines

### Angular (Jasmine/Karma)
- Use `HttpClientTestingModule` for HTTP mocking
- Use `fakeAsync`/`tick` for async operations
- Clear storage in `beforeEach`/`afterEach`

### Java (JUnit 5/Mockito)
- Use `@ExtendWith(MockitoExtension.class)`
- Use `@Mock` for dependencies
- Use AssertJ assertions for readability
- Use `@DisplayName` for test documentation

## Coverage Target

Aim for >70% line coverage with emphasis on:
- All public methods tested
- All error paths tested
- All security-critical paths tested
