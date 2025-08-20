# Testing Guide

## Overview

Phantom Power follows a comprehensive testing strategy across all services to ensure reliability, performance, and maintainability. This guide covers testing approaches for the frontend, backend, and audio processing service.

## Testing Philosophy

### Testing Pyramid
We follow the testing pyramid approach:
- **Unit Tests (70%)**: Fast, isolated tests for individual components/functions
- **Integration Tests (20%)**: Tests for service interactions and database operations
- **End-to-End Tests (10%)**: Full application workflow tests

### Test-Driven Development (TDD)
- Write tests before implementing features when possible
- Ensure all new code has corresponding tests
- Maintain high test coverage (>80% target)

## Frontend Testing (Next.js)

### Testing Stack
- **Jest** - JavaScript testing framework
- **React Testing Library** - React component testing utilities
- **MSW (Mock Service Worker)** - API mocking
- **Cypress** - End-to-end testing
- **@testing-library/jest-dom** - Custom Jest matchers

### Setup

```bash
cd frontend

# Install testing dependencies (already in package.json)
pnpm install

# Run tests
pnpm test

# Run tests in watch mode
pnpm test:watch

# Run tests with coverage
pnpm test:coverage

# Run E2E tests
pnpm test:e2e
```

### Unit Testing Components

**Example: UserProfile Component Test**

```typescript
// components/__tests__/UserProfile.test.tsx
import { render, screen } from '@testing-library/react'
import { UserProfile } from '../UserProfile'
import { mockUser } from '../../__mocks__/users'

describe('UserProfile Component', () => {
  it('renders user information correctly', () => {
    render(<UserProfile user={mockUser} />)
    
    expect(screen.getByText(mockUser.displayName)).toBeInTheDocument()
    expect(screen.getByText(mockUser.bio)).toBeInTheDocument()
    expect(screen.getByAltText('User avatar')).toHaveAttribute(
      'src', 
      mockUser.avatarUrl
    )
  })

  it('shows follow button for other users', () => {
    render(<UserProfile user={mockUser} currentUserId="different-id" />)
    
    expect(screen.getByRole('button', { name: /follow/i })).toBeInTheDocument()
  })

  it('does not show follow button for current user', () => {
    render(<UserProfile user={mockUser} currentUserId={mockUser.id} />)
    
    expect(screen.queryByRole('button', { name: /follow/i })).not.toBeInTheDocument()
  })
})
```

### Testing Custom Hooks

**Example: useAudioPlayer Hook Test**

```typescript
// hooks/__tests__/useAudioPlayer.test.ts
import { renderHook, act } from '@testing-library/react'
import { useAudioPlayer } from '../useAudioPlayer'

// Mock HTMLAudioElement
Object.defineProperty(window, 'HTMLAudioElement', {
  writable: true,
  value: jest.fn().mockImplementation(() => ({
    play: jest.fn().mockResolvedValue(undefined),
    pause: jest.fn(),
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    currentTime: 0,
    duration: 100,
  })),
})

describe('useAudioPlayer Hook', () => {
  it('initializes with default state', () => {
    const { result } = renderHook(() => useAudioPlayer('test-url.mp3'))
    
    expect(result.current.isPlaying).toBe(false)
    expect(result.current.currentTime).toBe(0)
    expect(result.current.duration).toBe(0)
  })

  it('plays and pauses audio', async () => {
    const { result } = renderHook(() => useAudioPlayer('test-url.mp3'))
    
    act(() => {
      result.current.play()
    })
    
    expect(result.current.isPlaying).toBe(true)
    
    act(() => {
      result.current.pause()
    })
    
    expect(result.current.isPlaying).toBe(false)
  })
})
```

### API Testing with MSW

**Setup MSW Handlers:**

```typescript
// __mocks__/handlers.ts
import { rest } from 'msw'
import { mockUsers, mockAudioFiles } from './data'

export const handlers = [
  // Auth endpoints
  rest.post('/api/auth/login', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json({
        success: true,
        data: {
          token: 'mock-jwt-token',
          user: mockUsers[0]
        }
      })
    )
  }),

  // User endpoints
  rest.get('/api/users/profile/:userId', (req, res, ctx) => {
    const { userId } = req.params
    const user = mockUsers.find(u => u.id === userId)
    
    if (!user) {
      return res(ctx.status(404), ctx.json({ error: 'User not found' }))
    }
    
    return res(ctx.status(200), ctx.json({ success: true, data: { user } }))
  }),

  // Audio endpoints
  rest.get('/api/users/:userId/audio', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json({
        success: true,
        data: {
          audioFiles: mockAudioFiles,
          pagination: { page: 1, total: mockAudioFiles.length }
        }
      })
    )
  })
]
```

### Integration Testing

**Example: Audio Upload Flow**

```typescript
// pages/__tests__/upload.integration.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react'
import { server } from '../__mocks__/server'
import UploadPage from '../upload'
import { AuthProvider } from '../contexts/AuthContext'

// Mock file for testing
const mockAudioFile = new File(['audio content'], 'test.mp3', {
  type: 'audio/mpeg'
})

describe('Audio Upload Integration', () => {
  it('uploads audio file successfully', async () => {
    render(
      <AuthProvider>
        <UploadPage />
      </AuthProvider>
    )
    
    const fileInput = screen.getByLabelText(/select audio file/i)
    const titleInput = screen.getByLabelText(/title/i)
    const submitButton = screen.getByRole('button', { name: /upload/i })
    
    // Fill form
    fireEvent.change(fileInput, { target: { files: [mockAudioFile] } })
    fireEvent.change(titleInput, { target: { value: 'Test Track' } })
    
    // Submit form
    fireEvent.click(submitButton)
    
    // Verify success message
    await waitFor(() => {
      expect(screen.getByText(/upload successful/i)).toBeInTheDocument()
    })
  })

  it('shows error for invalid file type', async () => {
    const invalidFile = new File(['content'], 'test.txt', { type: 'text/plain' })
    
    render(
      <AuthProvider>
        <UploadPage />
      </AuthProvider>
    )
    
    const fileInput = screen.getByLabelText(/select audio file/i)
    fireEvent.change(fileInput, { target: { files: [invalidFile] } })
    
    await waitFor(() => {
      expect(screen.getByText(/invalid file type/i)).toBeInTheDocument()
    })
  })
})
```

### End-to-End Testing with Cypress

**Example: User Authentication Flow**

```typescript
// cypress/e2e/auth.cy.ts
describe('User Authentication', () => {
  beforeEach(() => {
    cy.visit('/login')
  })

  it('allows user to register and login', () => {
    // Go to register page
    cy.contains('Sign up').click()
    
    // Fill registration form
    cy.get('[data-testid="email-input"]').type('test@example.com')
    cy.get('[data-testid="password-input"]').type('TestPass123!')
    cy.get('[data-testid="confirm-password-input"]').type('TestPass123!')
    
    // Submit registration
    cy.get('[data-testid="register-button"]').click()
    
    // Should redirect to dashboard
    cy.url().should('include', '/dashboard')
    cy.contains('Welcome to Phantom Power').should('be.visible')
  })

  it('shows error for invalid credentials', () => {
    cy.get('[data-testid="email-input"]').type('invalid@example.com')
    cy.get('[data-testid="password-input"]').type('wrongpassword')
    cy.get('[data-testid="login-button"]').click()
    
    cy.contains('Invalid credentials').should('be.visible')
  })
})
```

## Backend Testing (Spring Boot)

### Testing Stack
- **JUnit 5** - Java testing framework
- **Mockito** - Mocking framework
- **Spring Boot Test** - Integration testing support
- **TestContainers** - Database testing with Docker
- **WireMock** - HTTP service mocking

### Setup

```bash
cd backend

# Run all tests
mvn test

# Run tests with coverage
mvn jacoco:prepare-agent test jacoco:report

# Run integration tests only
mvn test -Dtest="**/*IntegrationTest"

# Run specific test class
mvn test -Dtest="UserServiceTest"
```

### Unit Testing Services

**Example: UserService Test**

```java
// src/test/java/com/phantompower/service/UserServiceTest.java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;
    
    @Mock
    private ProfileRepository profileRepository;
    
    @Mock
    private PasswordEncoder passwordEncoder;
    
    @InjectMocks
    private UserService userService;

    @Test
    void shouldCreateUserSuccessfully() {
        // Given
        RegisterRequest request = new RegisterRequest(
            "test@example.com", 
            "password123", 
            "password123"
        );
        
        User savedUser = User.builder()
            .id("user123")
            .email("test@example.com")
            .passwordHash("hashed-password")
            .build();
            
        when(userRepository.existsByEmail(request.getEmail())).thenReturn(false);
        when(passwordEncoder.encode(request.getPassword())).thenReturn("hashed-password");
        when(userRepository.save(any(User.class))).thenReturn(savedUser);
        
        // When
        UserResponse result = userService.createUser(request);
        
        // Then
        assertThat(result.getId()).isEqualTo("user123");
        assertThat(result.getEmail()).isEqualTo("test@example.com");
        verify(userRepository).save(any(User.class));
        verify(profileRepository).save(any(Profile.class));
    }

    @Test
    void shouldThrowExceptionWhenEmailExists() {
        // Given
        RegisterRequest request = new RegisterRequest(
            "existing@example.com", 
            "password123", 
            "password123"
        );
        
        when(userRepository.existsByEmail(request.getEmail())).thenReturn(true);
        
        // When & Then
        assertThatThrownBy(() -> userService.createUser(request))
            .isInstanceOf(EmailAlreadyExistsException.class)
            .hasMessage("User with email existing@example.com already exists");
    }
}
```

### Integration Testing Controllers

**Example: AuthController Integration Test**

```java
// src/test/java/com/phantompower/controller/AuthControllerIntegrationTest.java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@TestPropertySource(properties = {
    "spring.datasource.url=jdbc:h2:mem:testdb",
    "spring.jpa.hibernate.ddl-auto=create-drop"
})
class AuthControllerIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    void shouldRegisterUserSuccessfully() {
        // Given
        RegisterRequest request = new RegisterRequest(
            "test@example.com",
            "SecurePass123!",
            "SecurePass123!"
        );
        
        // When
        ResponseEntity<ApiResponse<AuthResponse>> response = restTemplate.postForEntity(
            "/api/auth/register",
            request,
            new ParameterizedTypeReference<ApiResponse<AuthResponse>>() {}
        );
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody().isSuccess()).isTrue();
        assertThat(response.getBody().getData().getUser().getEmail())
            .isEqualTo("test@example.com");
        assertThat(response.getBody().getData().getToken()).isNotNull();
        
        // Verify user was saved to database
        Optional<User> savedUser = userRepository.findByEmail("test@example.com");
        assertThat(savedUser).isPresent();
    }

    @Test
    void shouldReturnConflictWhenEmailExists() {
        // Given - Create existing user
        User existingUser = User.builder()
            .email("existing@example.com")
            .passwordHash("hashed-password")
            .build();
        userRepository.save(existingUser);
        
        RegisterRequest request = new RegisterRequest(
            "existing@example.com",
            "SecurePass123!",
            "SecurePass123!"
        );
        
        // When
        ResponseEntity<ApiResponse<Object>> response = restTemplate.postForEntity(
            "/api/auth/register",
            request,
            new ParameterizedTypeReference<ApiResponse<Object>>() {}
        );
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CONFLICT);
        assertThat(response.getBody().isSuccess()).isFalse();
    }
}
```

### Database Testing with TestContainers

**Example: Repository Test with PostgreSQL**

```java
// src/test/java/com/phantompower/repository/UserRepositoryIntegrationTest.java
@DataJpaTest
@Testcontainers
class UserRepositoryIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private UserRepository userRepository;

    @Test
    void shouldFindUserByEmail() {
        // Given
        User user = User.builder()
            .email("test@example.com")
            .passwordHash("hashed-password")
            .build();
        entityManager.persistAndFlush(user);
        
        // When
        Optional<User> found = userRepository.findByEmail("test@example.com");
        
        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getEmail()).isEqualTo("test@example.com");
    }

    @Test
    void shouldReturnEmptyWhenUserNotFound() {
        // When
        Optional<User> found = userRepository.findByEmail("nonexistent@example.com");
        
        // Then
        assertThat(found).isEmpty();
    }
}
```

## Audio Service Testing (Python)

### Testing Stack
- **pytest** - Python testing framework
- **pytest-asyncio** - Async testing support
- **httpx** - HTTP client for testing FastAPI
- **pytest-mock** - Mocking utilities
- **faker** - Test data generation

### Setup

```bash
cd audio-service

# Activate virtual environment
source venv/bin/activate

# Install test dependencies
pip install pytest pytest-asyncio httpx pytest-cov pytest-mock

# Run tests
pytest

# Run tests with coverage
pytest --cov=app --cov-report=html

# Run specific test file
pytest tests/test_analysis.py

# Run with verbose output
pytest -v
```

### Unit Testing Audio Analysis

**Example: Audio Analysis Service Test**

```python
# tests/test_analysis.py
import pytest
import numpy as np
from unittest.mock import Mock, patch
from app.services.analysis import AudioAnalysisService
from app.models.analysis import AnalysisResult

class TestAudioAnalysisService:
    
    def setup_method(self):
        self.service = AudioAnalysisService()
        
    @patch('librosa.load')
    @patch('librosa.tempo')
    @patch('librosa.feature.chroma_stft')
    def test_analyze_audio_file_success(self, mock_chroma, mock_tempo, mock_load):
        # Given
        mock_load.return_value = (np.random.random(44100), 22050)  # 1 second of audio
        mock_tempo.return_value = np.array([120.0])
        mock_chroma.return_value = np.random.random((12, 100))
        
        audio_file_path = 'test_audio.mp3'
        
        # When
        result = self.service.analyze_audio_file(audio_file_path)
        
        # Then
        assert isinstance(result, AnalysisResult)
        assert result.tempo == 120.0
        assert result.key is not None
        assert result.confidence > 0
        mock_load.assert_called_once_with(audio_file_path, sr=22050)
        
    def test_analyze_audio_file_invalid_path(self):
        # Given
        invalid_path = 'nonexistent_file.mp3'
        
        # When & Then
        with pytest.raises(FileNotFoundError):
            self.service.analyze_audio_file(invalid_path)
            
    @patch('librosa.load')
    def test_analyze_audio_file_corrupted_audio(self, mock_load):
        # Given
        mock_load.side_effect = Exception("Corrupted audio file")
        
        # When & Then
        with pytest.raises(Exception, match="Corrupted audio file"):
            self.service.analyze_audio_file('corrupted.mp3')

class TestTempoExtraction:
    
    def test_extract_tempo_from_audio(self):
        # Given
        service = AudioAnalysisService()
        # Create synthetic audio with known tempo
        sr = 22050
        duration = 5  # seconds
        t = np.linspace(0, duration, sr * duration)
        # 120 BPM = 2 beats per second
        audio = np.sin(2 * np.pi * 2 * t)  # 2 Hz beat frequency
        
        # When
        tempo = service._extract_tempo(audio, sr)
        
        # Then
        assert isinstance(tempo, float)
        assert 110 <= tempo <= 130  # Allow some tolerance
```

### API Testing with FastAPI TestClient

**Example: Analysis Endpoint Test**

```python
# tests/test_api.py
import pytest
from fastapi.testclient import TestClient
from unittest.mock import patch, Mock
from app.main import app
from app.models.analysis import AnalysisResult

client = TestClient(app)

class TestAnalysisAPI:
    
    def test_health_check(self):
        # When
        response = client.get("/health")
        
        # Then
        assert response.status_code == 200
        assert response.json()["status"] == "healthy"
        
    @patch('app.services.analysis.AudioAnalysisService.analyze_audio_file')
    def test_analyze_audio_success(self, mock_analyze):
        # Given
        mock_result = AnalysisResult(
            tempo=120.5,
            key="C#m",
            genre="ambient",
            confidence=0.95,
            metadata={"timbre": 0.72, "rhythmComplexity": 0.45}
        )
        mock_analyze.return_value = mock_result
        
        # Create a mock audio file
        audio_content = b"fake audio content"
        files = {"audio": ("test.mp3", audio_content, "audio/mpeg")}
        
        # When
        response = client.post("/analyze", files=files)
        
        # Then
        assert response.status_code == 200
        data = response.json()
        assert data["success"] is True
        assert data["data"]["tempo"] == 120.5
        assert data["data"]["key"] == "C#m"
        assert data["data"]["genre"] == "ambient"
        
    def test_analyze_audio_invalid_file_type(self):
        # Given
        text_content = b"this is not an audio file"
        files = {"audio": ("test.txt", text_content, "text/plain")}
        
        # When
        response = client.post("/analyze", files=files)
        
        # Then
        assert response.status_code == 400
        data = response.json()
        assert data["success"] is False
        assert "Invalid file type" in data["error"]["message"]
        
    def test_analyze_audio_missing_file(self):
        # When
        response = client.post("/analyze")
        
        # Then
        assert response.status_code == 422  # Validation error
        
    def test_analyze_audio_file_too_large(self):
        # Given - Create file larger than 50MB limit
        large_content = b"x" * (51 * 1024 * 1024)  # 51MB
        files = {"audio": ("large.mp3", large_content, "audio/mpeg")}
        
        # When
        response = client.post("/analyze", files=files)
        
        # Then
        assert response.status_code == 413  # Payload too large
```

### Async Testing

**Example: Async Service Test**

```python
# tests/test_async_analysis.py
import pytest
import asyncio
from unittest.mock import AsyncMock, patch
from app.services.async_analysis import AsyncAnalysisService

class TestAsyncAnalysisService:
    
    @pytest.mark.asyncio
    async def test_batch_analyze_audio_files(self):
        # Given
        service = AsyncAnalysisService()
        audio_files = ['file1.mp3', 'file2.mp3', 'file3.mp3']
        
        with patch.object(service, 'analyze_single_file', new_callable=AsyncMock) as mock_analyze:
            mock_analyze.return_value = Mock(tempo=120, key='C', genre='rock')
            
            # When
            results = await service.batch_analyze(audio_files)
            
            # Then
            assert len(results) == 3
            assert mock_analyze.call_count == 3
            for result in results:
                assert result.tempo == 120
                
    @pytest.mark.asyncio
    async def test_batch_analyze_with_timeout(self):
        # Given
        service = AsyncAnalysisService(timeout=1.0)  # 1 second timeout
        
        with patch.object(service, 'analyze_single_file', new_callable=AsyncMock) as mock_analyze:
            # Simulate slow analysis
            mock_analyze.side_effect = asyncio.sleep(2)  # 2 seconds > 1 second timeout
            
            # When & Then
            with pytest.raises(asyncio.TimeoutError):
                await service.batch_analyze(['slow_file.mp3'])
```

## Performance Testing

### Load Testing with Artillery

**Example: API Load Test Configuration**

```yaml
# artillery-config.yml
config:
  target: 'http://localhost:8080'
  phases:
    - duration: 60
      arrivalRate: 10
      name: "Warm up"
    - duration: 120
      arrivalRate: 50
      name: "Load test"
  variables:
    userEmail:
      - "user1@example.com"
      - "user2@example.com"
      - "user3@example.com"

scenarios:
  - name: "User Authentication Flow"
    weight: 30
    flow:
      - post:
          url: "/api/auth/login"
          json:
            email: "{{ userEmail }}"
            password: "testpass123"
          capture:
            - json: "$.data.token"
              as: "authToken"
      - get:
          url: "/api/users/me"
          headers:
            Authorization: "Bearer {{ authToken }}"
            
  - name: "Audio Upload Flow"
    weight: 20
    flow:
      - post:
          url: "/api/auth/login"
          json:
            email: "{{ userEmail }}"
            password: "testpass123"
          capture:
            - json: "$.data.token"
              as: "authToken"
      - post:
          url: "/api/users/me/audio"
          headers:
            Authorization: "Bearer {{ authToken }}"
          formData:
            title: "Load Test Audio"
            audio: "@./test-files/sample.mp3"
```

**Run Load Tests:**

```bash
# Install Artillery
npm install -g artillery

# Run load test
artillery run artillery-config.yml

# Generate HTML report
artillery run --output report.json artillery-config.yml
artillery report report.json
```

### Database Performance Testing

**Example: Repository Performance Test**

```java
// src/test/java/com/phantompower/repository/UserRepositoryPerformanceTest.java
@SpringBootTest
@Testcontainers
class UserRepositoryPerformanceTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

    @Autowired
    private UserRepository userRepository;

    @Test
    void shouldHandleHighVolumeUserSearch() {
        // Given - Create 10,000 test users
        List<User> users = IntStream.range(0, 10000)
            .mapToObj(i -> User.builder()
                .email("user" + i + "@example.com")
                .passwordHash("hash" + i)
                .build())
            .collect(Collectors.toList());
        userRepository.saveAll(users);
        
        // When - Measure search performance
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        
        List<User> results = userRepository.findByEmailContaining("user1");
        
        stopWatch.stop();
        
        // Then - Should complete within reasonable time
        assertThat(stopWatch.getTotalTimeMillis()).isLessThan(1000); // < 1 second
        assertThat(results).hasSizeGreaterThan(1000); // Should find user1, user10, user11, etc.
    }
}
```

## Continuous Integration Testing

### GitHub Actions Workflow

**Example: CI Testing Pipeline**

```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  frontend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'pnpm'
          cache-dependency-path: frontend/pnpm-lock.yaml
          
      - name: Install dependencies
        run: cd frontend && pnpm install
        
      - name: Run unit tests
        run: cd frontend && pnpm test:ci
        
      - name: Run E2E tests
        run: cd frontend && pnpm test:e2e:ci
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: frontend/coverage/lcov.info
          flags: frontend

  backend-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
          
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'openjdk'
          java-version: '17'
          
      - name: Cache Maven dependencies
        uses: actions/cache@v3
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('backend/pom.xml') }}
          
      - name: Run tests
        run: cd backend && mvn test
        env:
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/testdb
          SPRING_DATASOURCE_USERNAME: postgres
          SPRING_DATASOURCE_PASSWORD: postgres
          
      - name: Generate coverage report
        run: cd backend && mvn jacoco:report
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: backend/target/site/jacoco/jacoco.xml
          flags: backend

  audio-service-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          
      - name: Install Poetry
        uses: snok/install-poetry@v1
        
      - name: Install dependencies
        run: cd audio-service && poetry install
        
      - name: Run tests
        run: cd audio-service && poetry run pytest --cov=app --cov-report=xml
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: audio-service/coverage.xml
          flags: audio-service
```

## Test Data Management

### Fixtures and Factories

**Example: User Test Factory**

```java
// backend/src/test/java/com/phantompower/factory/UserFactory.java
public class UserFactory {
    
    public static User createUser() {
        return User.builder()
            .id(UUID.randomUUID().toString())
            .email("test@example.com")
            .passwordHash("$2a$10$hashed.password")
            .isVerified(true)
            .role(UserRole.USER)
            .createdAt(Instant.now())
            .build();
    }
    
    public static User createUser(String email) {
        return createUser().toBuilder()
            .email(email)
            .build();
    }
    
    public static List<User> createUsers(int count) {
        return IntStream.range(0, count)
            .mapToObj(i -> createUser("user" + i + "@example.com"))
            .collect(Collectors.toList());
    }
}
```

**Example: Frontend Test Data**

```typescript
// frontend/__mocks__/data.ts
export const mockUser = {
  id: '01HYN8FXZ2W6X8K6YTNPEKXYM7',
  email: 'jane.doe@example.com',
  profile: {
    displayName: 'Jane D',
    bio: 'Producer. Vocalist. Gearhead.',
    location: 'Brooklyn, NY',
    avatarUrl: 'https://example.com/avatar.jpg',
    tags: ['ambient', 'experimental', 'modular']
  }
}

export const mockAudioFile = {
  id: '01HYN91QZAYVNNRH53SJ1KMBCY',
  title: 'Cosmic Jam #3',
  description: 'Weird ambient mod synth groove',
  fileUrl: 'https://example.com/audio.mp3',
  duration: 212.4,
  analysis: {
    tempo: 120.5,
    key: 'C#m',
    genre: 'ambient'
  }
}

// Factory function for creating test data
export const createMockUser = (overrides: Partial<User> = {}): User => ({
  ...mockUser,
  ...overrides
})
```

## Best Practices

### General Testing Principles
1. **Write Clear Test Names** - Test names should describe what is being tested
2. **Follow AAA Pattern** - Arrange, Act, Assert
3. **Test One Thing at a Time** - Each test should verify one specific behavior
4. **Use Meaningful Assertions** - Assert on the actual behavior, not implementation details
5. **Keep Tests Independent** - Tests should not depend on each other

### Mocking Guidelines
1. **Mock External Dependencies** - Database, HTTP services, file system
2. **Don't Mock What You Don't Own** - Avoid mocking standard library functions
3. **Verify Important Interactions** - Use mocks to verify critical method calls
4. **Reset Mocks Between Tests** - Ensure clean state for each test

### Coverage Targets
- **Unit Tests**: >90% line coverage
- **Integration Tests**: >80% critical path coverage
- **E2E Tests**: Cover all major user workflows
- **API Tests**: 100% endpoint coverage

### Performance Testing Guidelines
1. **Establish Baselines** - Measure current performance before optimization
2. **Test Realistic Scenarios** - Use production-like data volumes
3. **Monitor Key Metrics** - Response time, throughput, resource usage
4. **Test Edge Cases** - High load, large files, concurrent users

<!-- TODO: Add mutation testing setup -->
<!-- TODO: Document visual regression testing -->
<!-- TODO: Add security testing guidelines -->
<!-- TODO: Create testing checklist for PRs -->