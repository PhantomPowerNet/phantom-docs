# Code Standards

## Overview

Phantom Power follows industry best practices inspired by major technology companies to ensure code quality, maintainability, and team collaboration. These standards promote consistency, readability, and reliability across our entire codebase.

## General Principles

### Clean Code Fundamentals
1. **Readability First**: Code is read more often than it's written
2. **Single Responsibility**: Each function, class, and module should have one clear purpose
3. **DRY (Don't Repeat Yourself)**: Eliminate code duplication through abstraction
4. **KISS (Keep It Simple, Stupid)**: Prefer simple, straightforward solutions
5. **YAGNI (You Aren't Gonna Need It)**: Don't build features until they're actually needed

### Documentation Philosophy
Following Google's and Microsoft's documentation standards:

**HOW Comments**: Explain complex algorithms, business logic, or non-obvious code patterns
```typescript
// HOW: Using debounce to prevent excessive API calls during user typing
// We wait 300ms after the user stops typing before triggering the search
const debouncedSearch = useMemo(
  () => debounce((query: string) => {
    performSearch(query);
  }, 300),
  []
);
```

**WHY Comments**: Explain business decisions, architectural choices, or trade-offs
```java
// WHY: We use optimistic locking here instead of pessimistic locking
// because audio file uploads are rare and conflicts are unlikely.
// This improves performance for the common case while handling
// the rare conflict scenario gracefully.
@Version
private Long version;
```

## Language-Specific Standards

### TypeScript/JavaScript (Frontend)

#### Code Style
Following Airbnb's JavaScript Style Guide with modifications:

```typescript
//  Good: Descriptive naming, clear types, proper error handling
interface AudioAnalysisResult {
  tempo: number;
  key: string;
  confidence: number;
  analyzedAt: Date;
}

async function analyzeAudioFile(
  file: File,
  options: AnalysisOptions = {}
): Promise<AudioAnalysisResult> {
  // WHY: Validate file size early to prevent unnecessary processing
  // and provide better user experience
  if (file.size > MAX_FILE_SIZE) {
    throw new AudioFileError('File size exceeds maximum limit', {
      maxSize: MAX_FILE_SIZE,
      actualSize: file.size
    });
  }

  try {
    // HOW: FormData is used here because the audio service expects
    // multipart/form-data for file uploads
    const formData = new FormData();
    formData.append('audio', file);
    formData.append('options', JSON.stringify(options));

    const response = await audioService.analyze(formData);
    
    return {
      tempo: response.data.tempo,
      key: response.data.key,
      confidence: response.data.confidence,
      analyzedAt: new Date(response.data.analyzedAt)
    };
  } catch (error) {
    // WHY: We wrap API errors in domain-specific errors to provide
    // better error handling and debugging information
    throw new AudioAnalysisError('Failed to analyze audio file', {
      cause: error,
      fileName: file.name,
      fileSize: file.size
    });
  }
}

// L Bad: Unclear naming, no error handling, missing types
function analyze(f: any) {
  const fd = new FormData();
  fd.append('audio', f);
  return fetch('/api/analyze', { method: 'POST', body: fd });
}
```

#### Component Standards
```typescript
//  Good: Well-structured React component with proper TypeScript
import React, { useState, useEffect, useCallback } from 'react';
import { AudioFile, User } from '@/types';
import { useAudioPlayer } from '@/hooks/useAudioPlayer';
import { Button, Card, Progress } from '@/components/ui';

interface AudioPlayerCardProps {
  audioFile: AudioFile;
  currentUser: User;
  onLike?: (audioFileId: string) => void;
  onShare?: (audioFile: AudioFile) => void;
  className?: string;
}

/**
 * AudioPlayerCard component displays an audio file with playback controls,
 * user interactions (like, share), and visual waveform representation.
 * 
 * WHY: This component encapsulates all audio playback functionality
 * to ensure consistent behavior across the application.
 */
export const AudioPlayerCard: React.FC<AudioPlayerCardProps> = ({
  audioFile,
  currentUser,
  onLike,
  onShare,
  className = ''
}) => {
  const [isLiked, setIsLiked] = useState(false);
  const {
    isPlaying,
    currentTime,
    duration,
    progress,
    play,
    pause,
    seek
  } = useAudioPlayer(audioFile.fileUrl);

  // HOW: We check the user's like status on component mount
  // and whenever the audioFile changes
  useEffect(() => {
    const checkLikeStatus = async () => {
      try {
        const liked = await audioService.isLikedByUser(
          audioFile.id,
          currentUser.id
        );
        setIsLiked(liked);
      } catch (error) {
        // WHY: We silently fail here because like status is not critical
        // for the core functionality of audio playback
        console.warn('Failed to check like status:', error);
      }
    };

    checkLikeStatus();
  }, [audioFile.id, currentUser.id]);

  const handleLike = useCallback(async () => {
    try {
      // WHY: Optimistic update for better user experience
      // We revert if the API call fails
      setIsLiked(prev => !prev);
      
      await audioService.toggleLike(audioFile.id, currentUser.id);
      onLike?.(audioFile.id);
    } catch (error) {
      // Revert optimistic update
      setIsLiked(prev => !prev);
      throw new Error('Failed to update like status');
    }
  }, [audioFile.id, currentUser.id, onLike]);

  const handleShare = useCallback(() => {
    onShare?.(audioFile);
  }, [audioFile, onShare]);

  return (
    <Card className={`audio-player-card ${className}`}>
      {/* Component JSX structure */}
    </Card>
  );
};

// L Bad: Unclear component structure, missing types, no error handling
function Player({ file, user }: any) {
  const [playing, setPlaying] = useState(false);
  
  return (
    <div>
      <button onClick={() => setPlaying(!playing)}>
        {playing ? 'Pause' : 'Play'}
      </button>
    </div>
  );
}
```

#### Testing Standards
```typescript
//  Good: Comprehensive test with clear naming and setup
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { vi } from 'vitest';
import { AudioPlayerCard } from './AudioPlayerCard';
import { mockAudioFile, mockUser } from '@/__mocks__/testData';

// WHY: We mock the audio service to isolate component testing
// from external dependencies and ensure predictable test results
vi.mock('@/services/audioService', () => ({
  isLikedByUser: vi.fn(),
  toggleLike: vi.fn()
}));

describe('AudioPlayerCard', () => {
  // HOW: Setup common test data and mocks before each test
  // to ensure clean state and reduce test interdependencies
  beforeEach(() => {
    vi.clearAllMocks();
  });

  describe('when user interacts with like button', () => {
    it('should toggle like status optimistically', async () => {
      // Arrange
      const mockOnLike = vi.fn();
      const { audioService } = await import('@/services/audioService');
      (audioService.isLikedByUser as any).mockResolvedValue(false);
      (audioService.toggleLike as any).mockResolvedValue(true);

      render(
        <AudioPlayerCard
          audioFile={mockAudioFile}
          currentUser={mockUser}
          onLike={mockOnLike}
        />
      );

      // Act
      const likeButton = screen.getByRole('button', { name: /like/i });
      fireEvent.click(likeButton);

      // Assert - Check optimistic update
      expect(screen.getByRole('button', { name: /unlike/i })).toBeInTheDocument();
      
      // Wait for API call to complete
      await waitFor(() => {
        expect(audioService.toggleLike).toHaveBeenCalledWith(
          mockAudioFile.id,
          mockUser.id
        );
      });
      
      expect(mockOnLike).toHaveBeenCalledWith(mockAudioFile.id);
    });

    it('should revert optimistic update when API call fails', async () => {
      // Test error handling scenario
    });
  });
});
```

### Java (Backend)

#### Code Style
Following Google Java Style Guide:

```java
//  Good: Clean, well-documented Spring Boot service
package com.phantompower.service;

import com.phantompower.dto.UserRegistrationRequest;
import com.phantompower.dto.UserResponse;
import com.phantompower.entity.User;
import com.phantompower.entity.Profile;
import com.phantompower.exception.EmailAlreadyExistsException;
import com.phantompower.repository.UserRepository;
import com.phantompower.repository.ProfileRepository;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import java.time.Instant;
import java.util.Optional;

/**
 * Service class responsible for user management operations including
 * registration, authentication, and profile management.
 * 
 * WHY: This service encapsulates all user-related business logic
 * to maintain separation of concerns and enable easier testing.
 */
@Service
@RequiredArgsConstructor
@Slf4j
public class UserService {

    private final UserRepository userRepository;
    private final ProfileRepository profileRepository;
    private final PasswordEncoder passwordEncoder;
    private final EmailService emailService;

    /**
     * Creates a new user account with associated profile.
     * 
     * WHY: We use a transactional approach to ensure both user and profile
     * are created atomically, preventing inconsistent state.
     * 
     * @param request the user registration data
     * @return UserResponse containing the created user information
     * @throws EmailAlreadyExistsException if email is already registered
     */
    @Transactional
    public UserResponse createUser(UserRegistrationRequest request) {
        log.info("Creating new user with email: {}", request.getEmail());
        
        // WHY: We check email existence early to provide fast feedback
        // and avoid unnecessary password hashing for duplicate emails
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new EmailAlreadyExistsException(
                "User with email " + request.getEmail() + " already exists"
            );
        }

        try {
            // HOW: We use a builder pattern for clean object construction
            // and encode the password using Spring Security's encoder
            User user = User.builder()
                .email(request.getEmail())
                .passwordHash(passwordEncoder.encode(request.getPassword()))
                .isVerified(false)
                .role(UserRole.USER)
                .createdAt(Instant.now())
                .updatedAt(Instant.now())
                .build();

            User savedUser = userRepository.save(user);
            log.debug("User created with ID: {}", savedUser.getId());

            // WHY: We create a basic profile immediately to ensure
            // the user has a complete account setup
            Profile profile = Profile.builder()
                .userId(savedUser.getId())
                .displayName(extractDisplayNameFromEmail(request.getEmail()))
                .bio("")
                .isPublic(true)
                .createdAt(Instant.now())
                .updatedAt(Instant.now())
                .build();

            profileRepository.save(profile);
            
            // HOW: We send verification email asynchronously to avoid
            // blocking the registration process
            emailService.sendVerificationEmailAsync(savedUser);

            return UserResponse.fromEntity(savedUser);
            
        } catch (Exception e) {
            log.error("Failed to create user with email: {}", request.getEmail(), e);
            throw new UserCreationException("Failed to create user account", e);
        }
    }

    /**
     * Retrieves user by ID with optional profile information.
     * 
     * @param userId the user ID to search for
     * @param includeProfile whether to include profile information
     * @return Optional containing UserResponse if found
     */
    @Transactional(readOnly = true)
    public Optional<UserResponse> getUserById(String userId, boolean includeProfile) {
        log.debug("Retrieving user by ID: {}, includeProfile: {}", userId, includeProfile);
        
        return userRepository.findById(userId)
            .map(user -> {
                UserResponse response = UserResponse.fromEntity(user);
                
                if (includeProfile) {
                    // HOW: We use optional chaining to handle cases where
                    // profile might not exist (data integrity issue)
                    profileRepository.findByUserId(userId)
                        .ifPresent(profile -> response.setProfile(ProfileResponse.fromEntity(profile)));
                }
                
                return response;
            });
    }

    /**
     * Extracts a display name from email address.
     * 
     * WHY: This provides a reasonable default display name for new users
     * while they can update it later.
     */
    private String extractDisplayNameFromEmail(String email) {
        // HOW: Take the part before @ and capitalize first letter
        String localPart = email.substring(0, email.indexOf('@'));
        return localPart.substring(0, 1).toUpperCase() + localPart.substring(1);
    }
}
```

#### Controller Standards
```java
//  Good: RESTful controller with proper validation and error handling
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
@Slf4j
@Validated
public class UserController {

    private final UserService userService;

    /**
     * Creates a new user account.
     * 
     * WHY: We use POST for creation following REST conventions
     * and return 201 Created with the created resource.
     */
    @PostMapping("/register")
    @ResponseStatus(HttpStatus.CREATED)
    public ResponseEntity<ApiResponse<UserResponse>> registerUser(
            @Valid @RequestBody UserRegistrationRequest request) {
        
        log.info("User registration attempt for email: {}", request.getEmail());
        
        try {
            UserResponse user = userService.createUser(request);
            
            // WHY: We use a consistent API response wrapper for all endpoints
            // to provide uniform error handling and metadata
            ApiResponse<UserResponse> response = ApiResponse.<UserResponse>builder()
                .success(true)
                .data(user)
                .message("User created successfully")
                .timestamp(Instant.now())
                .build();
                
            return ResponseEntity.status(HttpStatus.CREATED).body(response);
        } catch (EmailAlreadyExistsException e) {
            return handleEmailConflict(e);
        }
    }

    /**
     * Retrieves user profile information.
     */
    @GetMapping("/profile/{userId}")
    public ResponseEntity<ApiResponse<UserResponse>> getUserProfile(
            @PathVariable @NotBlank String userId,
            @RequestParam(defaultValue = "true") boolean includeProfile,
            Authentication authentication) {
        
        log.debug("Retrieving user profile for ID: {}", userId);
        
        return userService.getUserById(userId, includeProfile)
            .map(user -> {
                // WHY: We check privacy settings to ensure users can control
                // who sees their profile information
                if (!canViewProfile(user, authentication)) {
                    throw new ProfileAccessDeniedException("Profile is private");
                }
                
                ApiResponse<UserResponse> response = ApiResponse.<UserResponse>builder()
                    .success(true)
                    .data(user)
                    .build();
                    
                return ResponseEntity.ok(response);
            })
            .orElse(ResponseEntity.notFound().build());
    }

    private ResponseEntity<ApiResponse<UserResponse>> handleEmailConflict(
            EmailAlreadyExistsException e) {
        ApiResponse<UserResponse> response = ApiResponse.<UserResponse>builder()
            .success(false)
            .error(ErrorDetails.builder()
                .code("EMAIL_ALREADY_EXISTS")
                .message(e.getMessage())
                .build())
            .timestamp(Instant.now())
            .build();
            
        return ResponseEntity.status(HttpStatus.CONFLICT).body(response);
    }
}
```

#### Testing Standards
```java
//  Good: Comprehensive service test with proper mocking
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;
    
    @Mock
    private ProfileRepository profileRepository;
    
    @Mock
    private PasswordEncoder passwordEncoder;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;

    private UserRegistrationRequest validRequest;
    private User sampleUser;

    @BeforeEach
    void setUp() {
        validRequest = UserRegistrationRequest.builder()
            .email("test@example.com")
            .password("SecurePass123!")
            .confirmPassword("SecurePass123!")
            .build();
            
        sampleUser = User.builder()
            .id("user-123")
            .email("test@example.com")
            .passwordHash("hashed-password")
            .isVerified(false)
            .role(UserRole.USER)
            .createdAt(Instant.now())
            .build();
    }

    @Test
    @DisplayName("Should create user successfully with valid request")
    void shouldCreateUserSuccessfully() {
        // Given
        when(userRepository.existsByEmail(validRequest.getEmail()))
            .thenReturn(false);
        when(passwordEncoder.encode(validRequest.getPassword()))
            .thenReturn("hashed-password");
        when(userRepository.save(any(User.class)))
            .thenReturn(sampleUser);
        when(profileRepository.save(any(Profile.class)))
            .thenReturn(Profile.builder().userId(sampleUser.getId()).build());

        // When
        UserResponse result = userService.createUser(validRequest);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getEmail()).isEqualTo(validRequest.getEmail());
        assertThat(result.getId()).isEqualTo(sampleUser.getId());
        
        // Verify interactions
        verify(userRepository).existsByEmail(validRequest.getEmail());
        verify(passwordEncoder).encode(validRequest.getPassword());
        verify(userRepository).save(argThat(user -> 
            user.getEmail().equals(validRequest.getEmail()) &&
            user.getPasswordHash().equals("hashed-password")
        ));
        verify(profileRepository).save(any(Profile.class));
        verify(emailService).sendVerificationEmailAsync(sampleUser);
    }

    @Test
    @DisplayName("Should throw EmailAlreadyExistsException when email exists")
    void shouldThrowExceptionWhenEmailExists() {
        // Given
        when(userRepository.existsByEmail(validRequest.getEmail()))
            .thenReturn(true);

        // When & Then
        assertThatThrownBy(() -> userService.createUser(validRequest))
            .isInstanceOf(EmailAlreadyExistsException.class)
            .hasMessageContaining("already exists");
            
        // Verify no other interactions
        verify(passwordEncoder, never()).encode(anyString());
        verify(userRepository, never()).save(any(User.class));
    }
}
```

### Python (Audio Service)

#### Code Style
Following PEP 8 and Google's Python Style Guide:

```python
#  Good: Well-structured FastAPI service with proper typing
from typing import List, Optional, Dict, Any
from datetime import datetime
from pathlib import Path

import librosa
import numpy as np
from fastapi import FastAPI, HTTPException, UploadFile, File, Depends
from pydantic import BaseModel, Field, validator
from sqlalchemy.orm import Session

from app.database import get_db
from app.models.analysis import AnalysisResult
from app.services.audio_processor import AudioProcessor
from app.core.config import settings
from app.core.logging import get_logger

logger = get_logger(__name__)


class AnalysisRequest(BaseModel):
    """Request model for audio analysis.
    
    WHY: Using Pydantic models ensures request validation and
    provides automatic API documentation generation.
    """
    analysis_type: str = Field(
        default="full",
        description="Type of analysis to perform",
        regex="^(basic|full|custom)$"
    )
    return_waveform: bool = Field(
        default=False,
        description="Whether to include waveform data in response"
    )
    custom_metrics: Optional[List[str]] = Field(
        default=None,
        description="Specific metrics to calculate for custom analysis"
    )

    @validator('custom_metrics')
    def validate_custom_metrics(cls, v, values):
        """Validate custom metrics are only provided for custom analysis type."""
        if values.get('analysis_type') == 'custom' and not v:
            raise ValueError('custom_metrics required for custom analysis type')
        return v


class AnalysisResponse(BaseModel):
    """Response model for audio analysis results."""
    id: str
    audio_file_id: str
    basic: Dict[str, Any]
    advanced: Optional[Dict[str, Any]] = None
    custom_metrics: Optional[Dict[str, Any]] = None
    waveform: Optional[Dict[str, Any]] = None
    confidence: float
    processing_time: float
    model_version: str
    analyzed_at: datetime


class AudioAnalysisService:
    """Service class for audio analysis operations.
    
    WHY: Separating business logic from API endpoints makes the code
    more testable and maintainable. This follows the dependency
    injection pattern for better modularity.
    """

    def __init__(self, audio_processor: AudioProcessor):
        self.audio_processor = audio_processor
        
    async def analyze_audio_file(
        self,
        audio_file: UploadFile,
        request: AnalysisRequest,
        db: Session
    ) -> AnalysisResponse:
        """Analyze an uploaded audio file.
        
        Args:
            audio_file: The uploaded audio file
            request: Analysis configuration
            db: Database session for storing results
            
        Returns:
            AnalysisResponse containing the analysis results
            
        Raises:
            HTTPException: If file validation or analysis fails
        """
        start_time = datetime.now()
        logger.info(
            "Starting audio analysis for file: %s, type: %s",
            audio_file.filename,
            request.analysis_type
        )
        
        try:
            # WHY: We validate the file early to provide fast feedback
            # and prevent processing invalid files
            await self._validate_audio_file(audio_file)
            
            # HOW: We save the file to a temporary location for processing
            # and clean it up afterwards to prevent disk space issues
            temp_file_path = await self._save_temp_file(audio_file)
            
            try:
                analysis_result = await self._perform_analysis(
                    temp_file_path,
                    request
                )
                
                # WHY: We store results in database for caching and
                # historical analysis tracking
                stored_result = await self._store_analysis_result(
                    analysis_result,
                    audio_file.filename,
                    db
                )
                
                processing_time = (datetime.now() - start_time).total_seconds()
                logger.info(
                    "Audio analysis completed in %.2f seconds for file: %s",
                    processing_time,
                    audio_file.filename
                )
                
                return AnalysisResponse(
                    id=stored_result.id,
                    audio_file_id=stored_result.audio_file_id,
                    basic=analysis_result.basic,
                    advanced=analysis_result.advanced,
                    custom_metrics=analysis_result.custom_metrics,
                    waveform=analysis_result.waveform if request.return_waveform else None,
                    confidence=analysis_result.confidence,
                    processing_time=processing_time,
                    model_version=settings.MODEL_VERSION,
                    analyzed_at=stored_result.created_at
                )
                
            finally:
                # HOW: Clean up temporary file regardless of success/failure
                # to prevent disk space accumulation
                if temp_file_path.exists():
                    temp_file_path.unlink()
                    
        except Exception as e:
            logger.error(
                "Audio analysis failed for file: %s, error: %s",
                audio_file.filename,
                str(e),
                exc_info=True
            )
            raise HTTPException(
                status_code=500,
                detail=f"Audio analysis failed: {str(e)}"
            ) from e
    
    async def _validate_audio_file(self, audio_file: UploadFile) -> None:
        """Validate uploaded audio file.
        
        WHY: Early validation prevents processing invalid files and
        provides clear error messages to users.
        """
        # Check file size
        if audio_file.size > settings.MAX_AUDIO_FILE_SIZE:
            raise HTTPException(
                status_code=413,
                detail=f"File size ({audio_file.size} bytes) exceeds maximum allowed "
                       f"size ({settings.MAX_AUDIO_FILE_SIZE} bytes)"
            )
        
        # Check file format
        if not audio_file.content_type.startswith('audio/'):
            raise HTTPException(
                status_code=400,
                detail=f"Invalid file type: {audio_file.content_type}. "
                       "Only audio files are supported."
            )
        
        # Check file extension
        file_extension = Path(audio_file.filename).suffix.lower()
        if file_extension not in settings.SUPPORTED_AUDIO_FORMATS:
            raise HTTPException(
                status_code=400,
                detail=f"Unsupported file format: {file_extension}. "
                       f"Supported formats: {', '.join(settings.SUPPORTED_AUDIO_FORMATS)}"
            )
    
    async def _perform_analysis(
        self,
        file_path: Path,
        request: AnalysisRequest
    ) -> Dict[str, Any]:
        """Perform the actual audio analysis.
        
        HOW: We use librosa for audio processing and analysis.
        The analysis is performed in multiple stages based on the
        requested analysis type.
        """
        try:
            # Load audio file
            audio_data, sample_rate = librosa.load(str(file_path), sr=None)
            
            if len(audio_data) == 0:
                raise ValueError("Audio file contains no data")
            
            result = {
                'basic': await self._analyze_basic_features(audio_data, sample_rate),
                'confidence': 0.0,
                'waveform': None
            }
            
            if request.analysis_type in ['full', 'custom']:
                result['advanced'] = await self._analyze_advanced_features(
                    audio_data, 
                    sample_rate
                )
            
            if request.analysis_type == 'custom' and request.custom_metrics:
                result['custom_metrics'] = await self._analyze_custom_metrics(
                    audio_data,
                    sample_rate,
                    request.custom_metrics
                )
            
            if request.return_waveform:
                result['waveform'] = await self._generate_waveform_data(
                    audio_data, 
                    sample_rate
                )
            
            # Calculate overall confidence based on individual metric confidences
            result['confidence'] = self._calculate_overall_confidence(result)
            
            return result
            
        except Exception as e:
            logger.error("Audio analysis processing failed: %s", str(e))
            raise ValueError(f"Failed to analyze audio: {str(e)}") from e


# L Bad: Poor structure, no error handling, unclear logic
def analyze(file):
    data, sr = librosa.load(file)
    tempo = librosa.beat.tempo(data, sr=sr)[0]
    return {'tempo': tempo}
```

#### Testing Standards
```python
#  Good: Comprehensive async test with proper mocking
import pytest
import asyncio
from unittest.mock import AsyncMock, MagicMock, patch
from pathlib import Path
from fastapi import UploadFile
from sqlalchemy.orm import Session

from app.services.audio_analysis import AudioAnalysisService, AnalysisRequest
from app.services.audio_processor import AudioProcessor
from app.core.exceptions import AudioAnalysisError


class TestAudioAnalysisService:
    """Test suite for AudioAnalysisService.
    
    WHY: We use a class-based approach to group related tests and
    share common setup between test methods.
    """
    
    @pytest.fixture
    def mock_audio_processor(self):
        """Create mock audio processor for testing."""
        return MagicMock(spec=AudioProcessor)
    
    @pytest.fixture
    def audio_service(self, mock_audio_processor):
        """Create AudioAnalysisService instance with mocked dependencies."""
        return AudioAnalysisService(audio_processor=mock_audio_processor)
    
    @pytest.fixture
    def sample_audio_file(self):
        """Create mock UploadFile for testing."""
        file = MagicMock(spec=UploadFile)
        file.filename = "test_audio.mp3"
        file.content_type = "audio/mpeg"
        file.size = 1024 * 1024  # 1MB
        file.read = AsyncMock(return_value=b"fake audio data")
        return file
    
    @pytest.fixture
    def analysis_request(self):
        """Create sample analysis request."""
        return AnalysisRequest(
            analysis_type="full",
            return_waveform=True,
            custom_metrics=None
        )
    
    @pytest.fixture
    def mock_db_session(self):
        """Create mock database session."""
        return MagicMock(spec=Session)
    
    @pytest.mark.asyncio
    async def test_analyze_audio_file_success(
        self,
        audio_service,
        sample_audio_file,
        analysis_request,
        mock_db_session
    ):
        """Test successful audio file analysis.
        
        WHY: This test covers the happy path scenario to ensure
        the service works correctly with valid inputs.
        """
        # Arrange
        expected_analysis_result = {
            'basic': {'tempo': 120.0, 'key': 'C', 'duration': 180.0},
            'advanced': {'genre': 'rock', 'mood': 'energetic'},
            'confidence': 0.85,
            'waveform': {'peaks': [0.1, 0.8, 0.3]}
        }
        
        with patch.object(audio_service, '_validate_audio_file') as mock_validate, \
             patch.object(audio_service, '_save_temp_file') as mock_save, \
             patch.object(audio_service, '_perform_analysis') as mock_analyze, \
             patch.object(audio_service, '_store_analysis_result') as mock_store:
            
            # Configure mocks
            mock_validate.return_value = None
            mock_save.return_value = Path("/tmp/test_audio.mp3")
            mock_analyze.return_value = expected_analysis_result
            mock_store.return_value = MagicMock(
                id="analysis-123",
                audio_file_id="file-456",
                created_at=datetime.now()
            )
            
            # Act
            result = await audio_service.analyze_audio_file(
                sample_audio_file,
                analysis_request,
                mock_db_session
            )
            
            # Assert
            assert result.id == "analysis-123"
            assert result.audio_file_id == "file-456"
            assert result.basic == expected_analysis_result['basic']
            assert result.advanced == expected_analysis_result['advanced']
            assert result.confidence == expected_analysis_result['confidence']
            assert result.waveform == expected_analysis_result['waveform']
            assert result.processing_time > 0
            
            # Verify method calls
            mock_validate.assert_called_once_with(sample_audio_file)
            mock_save.assert_called_once_with(sample_audio_file)
            mock_analyze.assert_called_once()
            mock_store.assert_called_once()
    
    @pytest.mark.asyncio
    async def test_analyze_audio_file_validation_error(
        self,
        audio_service,
        sample_audio_file,
        analysis_request,
        mock_db_session
    ):
        """Test audio file analysis with validation error.
        
        WHY: Testing error scenarios ensures our service handles
        invalid inputs gracefully and provides meaningful error messages.
        """
        # Arrange
        with patch.object(audio_service, '_validate_audio_file') as mock_validate:
            mock_validate.side_effect = HTTPException(
                status_code=400,
                detail="Invalid file type"
            )
            
            # Act & Assert
            with pytest.raises(HTTPException) as exc_info:
                await audio_service.analyze_audio_file(
                    sample_audio_file,
                    analysis_request,
                    mock_db_session
                )
            
            assert exc_info.value.status_code == 400
            assert "Invalid file type" in str(exc_info.value.detail)
    
    @pytest.mark.asyncio
    async def test_validate_audio_file_size_limit(
        self,
        audio_service,
        sample_audio_file
    ):
        """Test file size validation."""
        # Arrange
        sample_audio_file.size = 100 * 1024 * 1024  # 100MB (exceeds limit)
        
        # Act & Assert
        with pytest.raises(HTTPException) as exc_info:
            await audio_service._validate_audio_file(sample_audio_file)
        
        assert exc_info.value.status_code == 413
        assert "exceeds maximum allowed size" in str(exc_info.value.detail)
    
    @pytest.mark.parametrize("file_extension,content_type,should_pass", [
        (".mp3", "audio/mpeg", True),
        (".wav", "audio/wav", True),
        (".flac", "audio/flac", True),
        (".txt", "text/plain", False),
        (".mp4", "video/mp4", False),
    ])
    @pytest.mark.asyncio
    async def test_validate_audio_file_format(
        self,
        audio_service,
        sample_audio_file,
        file_extension,
        content_type,
        should_pass
    ):
        """Test file format validation with various file types.
        
        WHY: Parameterized tests allow us to test multiple scenarios
        efficiently while maintaining good test coverage.
        """
        # Arrange
        sample_audio_file.filename = f"test{file_extension}"
        sample_audio_file.content_type = content_type
        sample_audio_file.size = 1024  # Small valid size
        
        # Act & Assert
        if should_pass:
            # Should not raise an exception
            await audio_service._validate_audio_file(sample_audio_file)
        else:
            with pytest.raises(HTTPException) as exc_info:
                await audio_service._validate_audio_file(sample_audio_file)
            assert exc_info.value.status_code == 400
```

## Testing Standards

### Test Coverage Requirements
- **Minimum Coverage**: 80% per pull request
- **Sprint Goal**: 90% coverage by sprint end
- **Critical Paths**: 100% coverage for security, payment, and data integrity code

### Test Pyramid Structure
```
    E2E Tests (10%)
   /               \
  /  Integration    \
 /   Tests (20%)    \
/___________________\
|   Unit Tests      |
|     (70%)         |
|___________________|
```

### Testing Best Practices
1. **Test Names**: Should describe what is being tested and expected outcome
2. **AAA Pattern**: Arrange, Act, Assert structure
3. **Single Responsibility**: Each test should verify one specific behavior
4. **Independent Tests**: Tests should not depend on each other
5. **Meaningful Assertions**: Assert on behavior, not implementation details

## Code Review Standards

### Review Checklist
#### Functionality
- [ ] Code solves the stated problem correctly
- [ ] Edge cases are handled appropriately
- [ ] Error handling is comprehensive and user-friendly
- [ ] Performance implications are considered

#### Code Quality
- [ ] Code follows established patterns and conventions
- [ ] Functions and classes have single, clear responsibilities
- [ ] Names are descriptive and unambiguous
- [ ] Comments explain WHY and HOW, not WHAT
- [ ] No code duplication without justification

#### Security
- [ ] No hardcoded secrets or sensitive data
- [ ] Input validation is present and thorough
- [ ] Authentication and authorization are properly implemented
- [ ] SQL injection and XSS vulnerabilities are prevented

#### Testing
- [ ] Unit tests cover new functionality
- [ ] Integration tests cover critical paths
- [ ] Test coverage meets minimum thresholds
- [ ] Tests are maintainable and not brittle

### Review Process
1. **Self-Review**: Author reviews their own PR before requesting review
2. **Automated Checks**: All CI/CD checks must pass before human review
3. **Peer Review**: At least one team member must approve
4. **Security Review**: Required for authentication, payment, or data handling changes
5. **Architectural Review**: Required for significant design changes

## Performance Standards

### Frontend Performance
- **First Contentful Paint**: < 1.5s
- **Largest Contentful Paint**: < 2.5s
- **Cumulative Layout Shift**: < 0.1
- **First Input Delay**: < 100ms
- **Bundle Size**: < 250KB gzipped per route

### Backend Performance
- **API Response Time**: < 200ms for 95th percentile
- **Database Query Time**: < 50ms for 95th percentile
- **Memory Usage**: < 512MB per service instance
- **CPU Usage**: < 70% under normal load

### Audio Service Performance
- **Basic Analysis**: < 5 seconds per file
- **Full Analysis**: < 30 seconds per file
- **Concurrent Processing**: Support 5 simultaneous analyses
- **Memory Efficiency**: Clean up temporary files immediately

## Security Standards

### Authentication & Authorization
- Use JWT tokens with appropriate expiration times
- Implement refresh token rotation
- Follow principle of least privilege for all permissions
- Log all authentication attempts and failures

### Data Protection
- Encrypt sensitive data at rest and in transit
- Use parameterized queries to prevent SQL injection
- Validate and sanitize all user inputs
- Implement rate limiting on all public endpoints

### Secret Management
- Never commit secrets to version control
- Use environment variables for configuration
- Rotate secrets regularly
- Use different secrets per environment

## Documentation Standards

### Code Documentation
- **API Documentation**: OpenAPI/Swagger specifications for all endpoints
- **README Files**: Clear setup and usage instructions
- **Architecture Docs**: High-level system design and data flow
- **Decision Records**: Document significant architectural decisions

### Comment Guidelines
- Focus on WHY and HOW, not WHAT
- Update comments when code changes
- Use TODO comments for known technical debt
- Avoid obvious comments that restate the code

## Dependency Management

### Frontend Dependencies
- Keep dependencies up to date with security patches
- Use exact versions in package.json for consistency
- Regularly audit dependencies for vulnerabilities
- Justify each new dependency addition

### Backend Dependencies
- Use dependency management tools (Maven, Poetry)
- Keep Spring Boot and other frameworks updated
- Monitor for security vulnerabilities
- Document any custom or forked dependencies

### Security Scanning
- Run dependency vulnerability scans in CI/CD
- Address high and critical vulnerabilities promptly
- Use tools like Snyk, OWASP Dependency Check
- Regular security audits of third-party code

## Git Standards

### Commit Message Format
```
type(scope): brief description

Longer description explaining the change in detail.
Include reasoning for the change and contrast with
previous behavior.

Closes #123
```

### Commit Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code changes that neither fix bugs nor add features
- `test`: Adding or modifying tests
- `chore`: Changes to build process, dependencies, etc.

### Branch Naming
- `feature/description-of-feature`
- `fix/description-of-bug-fix`
- `hotfix/critical-issue-description`
- `refactor/component-name`
- `docs/documentation-update`

## Continuous Improvement

### Code Quality Metrics
- Monitor code coverage trends
- Track technical debt using tools like SonarQube
- Measure code complexity and maintainability
- Regular architecture reviews

### Team Practices
- Weekly code review retrospectives
- Monthly technical debt planning sessions
- Quarterly architecture review meetings
- Annual technology stack evaluation

### Learning and Development
- Encourage experimentation in feature branches
- Share learnings through team presentations
- Contribute to open source projects
- Stay updated with industry best practices

<!-- TODO: Add specific linting configurations for each service -->
<!-- TODO: Document database schema design standards -->
<!-- TODO: Add API design guidelines -->
<!-- TODO: Create code quality dashboard -->