# Audio Analysis API

## Overview

The Audio Analysis API is a Python FastAPI microservice that provides intelligent audio processing capabilities using librosa, madmom, and other audio analysis libraries. It extracts musical features like tempo, key, genre, and advanced metrics from uploaded audio files.

## Base URL

```
Development: http://localhost:8000
Production: https://audio-api.phantompower.app
```

## Supported Audio Formats

- **MP3** - MPEG Audio Layer III
- **WAV** - Uncompressed audio
- **FLAC** - Free Lossless Audio Codec
- **M4A** - MPEG-4 Audio
- **AAC** - Advanced Audio Coding
- **OGG** - Ogg Vorbis

**File Size Limits:**
- Maximum file size: 50MB
- Maximum duration: 30 minutes
- Minimum duration: 5 seconds

## Analysis Capabilities

### Basic Audio Features
- **Tempo/BPM** - Beat detection and tempo estimation
- **Key Detection** - Musical key and scale identification
- **Time Signature** - Meter detection (4/4, 3/4, etc.)
- **Duration** - Precise audio length calculation

### Advanced Audio Features
- **Genre Classification** - ML-based genre detection
- **Mood Analysis** - Emotional content analysis
- **Spectral Features** - MFCC, spectral centroid, rolloff
- **Rhythmic Features** - Beat strength, rhythm complexity
- **Harmonic Features** - Chroma features, key strength

### Custom Metrics
- **Dynamics Quotient** - Measure of dynamic range and variation
- **MAD Divergence** - Melodic and harmonic divergence analysis
- **Texture Complexity** - Audio texture and timbre analysis
- **Energy Distribution** - Frequency band energy analysis

## Endpoints

### Health Check

Check service health and status.

**Endpoint:** `GET /health`

**Response (200 OK):**
```json
{
  "status": "healthy",
  "timestamp": "2025-07-19T20:00:00Z",
  "version": "1.0.0",
  "dependencies": {
    "librosa": "0.10.1",
    "madmom": "0.16.1",
    "database": "connected"
  }
}
```

### Analyze Audio File

Perform comprehensive audio analysis on uploaded file.

**Endpoint:** `POST /analyze`

**Content-Type:** `multipart/form-data`

**Request Body:**
- `audio` (file, required) - Audio file to analyze
- `analysis_type` (string, optional) - Type of analysis: "basic", "full", "custom"
- `return_waveform` (boolean, optional) - Include waveform data in response
- `custom_metrics` (string[], optional) - Specific metrics to calculate

**Example Request:**
```bash
curl -X POST http://localhost:8000/analyze \
  -F "audio=@cosmic_jam.mp3" \
  -F "analysis_type=full" \
  -F "return_waveform=true"
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "analysis": {
      "id": "01HRSTAIRES1ABC1234567890",
      "audioFileId": "01HYN91QZAYVNNRH53SJ1KMBCY",
      "basic": {
        "tempo": 120.5,
        "key": "C#m",
        "timeSignature": "4/4",
        "duration": 212.4,
        "sampleRate": 44100,
        "channels": 2
      },
      "advanced": {
        "genre": {
          "primary": "ambient",
          "secondary": "electronic",
          "confidence": 0.89,
          "probabilities": {
            "ambient": 0.89,
            "electronic": 0.67,
            "experimental": 0.45,
            "drone": 0.34
          }
        },
        "mood": {
          "valence": 0.3,
          "arousal": 0.2,
          "dominance": 0.4,
          "mood_labels": ["contemplative", "calm", "mysterious"]
        },
        "spectral": {
          "mfcc": [12.3, -8.7, 4.2, -2.1, 1.8],
          "spectralCentroid": 2847.6,
          "spectralRolloff": 5632.1,
          "zeroCrossingRate": 0.045
        },
        "rhythmic": {
          "beatStrength": 0.72,
          "rhythmComplexity": 0.34,
          "onsetDensity": 1.2,
          "tempoStability": 0.91
        },
        "harmonic": {
          "chromaVector": [0.8, 0.1, 0.3, 0.9, 0.2, 0.1, 0.7, 0.4, 0.6, 0.3, 0.2, 0.5],
          "keyStrength": 0.84,
          "harmonicComplexity": 0.56,
          "tonalCentroid": [0.32, 0.68]
        }
      },
      "customMetrics": {
        "dynamicsQuotient": {
          "value": 0.67,
          "description": "Measure of dynamic range variation",
          "methodology": "RMS variation over time windows"
        },
        "madDivergence": {
          "value": 0.23,
          "description": "Melodic and harmonic divergence from Western scales",
          "methodology": "Chroma deviation from major/minor templates"
        },
        "textureComplexity": {
          "value": 0.45,
          "description": "Audio texture and timbre complexity",
          "methodology": "Spectral feature variance analysis"
        }
      },
      "waveform": {
        "peaks": [0.1, 0.3, 0.8, 0.5, 0.2, 0.9, 0.4],
        "rms": [0.05, 0.12, 0.34, 0.28, 0.15, 0.41, 0.19],
        "timeAxis": [0, 30, 60, 90, 120, 150, 180, 210],
        "samplePoints": 1024
      },
      "confidence": 0.91,
      "processingTime": 8.7,
      "modelVersion": "librosa-0.10.1-v2",
      "analyzedAt": "2025-07-19T20:15:00Z"
    }
  }
}
```

**Error Responses:**
- `400 Bad Request` - Invalid file format or corrupted audio
- `413 Payload Too Large` - File exceeds size limit
- `422 Unprocessable Entity` - Missing required fields
- `429 Too Many Requests` - Rate limit exceeded
- `500 Internal Server Error` - Analysis processing failed

### Batch Analysis

Analyze multiple audio files in a single request.

**Endpoint:** `POST /analyze/batch`

**Content-Type:** `multipart/form-data`

**Request Body:**
- `audio_files` (file[], required) - Array of audio files to analyze (max 10 files)
- `analysis_type` (string, optional) - Analysis type for all files

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "batchId": "01HYBATCH1ABC123456789",
    "results": [
      {
        "filename": "track1.mp3",
        "status": "completed",
        "analysis": {
          "tempo": 128.0,
          "key": "Am",
          "genre": "techno"
        }
      },
      {
        "filename": "track2.mp3",
        "status": "failed",
        "error": "Corrupted audio file"
      }
    ],
    "summary": {
      "total": 2,
      "completed": 1,
      "failed": 1,
      "processingTime": 15.3
    }
  }
}
```

### Get Analysis Result

Retrieve previously computed analysis results.

**Endpoint:** `GET /analysis/{analysisId}`

**Parameters:**
- `analysisId` (path) - Analysis result ID

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "analysis": {
      "id": "01HRSTAIRES1ABC1234567890",
      "status": "completed",
      "createdAt": "2025-07-19T20:15:00Z",
      "results": {
        "tempo": 120.5,
        "key": "C#m",
        "genre": "ambient"
      }
    }
  }
}
```

### Compare Audio Files

Compare two audio files and calculate similarity metrics.

**Endpoint:** `POST /compare`

**Content-Type:** `multipart/form-data`

**Request Body:**
- `audio1` (file, required) - First audio file
- `audio2` (file, required) - Second audio file
- `comparison_type` (string, optional) - "similarity", "distance", "features"

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "comparison": {
      "overallSimilarity": 0.73,
      "metrics": {
        "tempoSimilarity": 0.89,
        "keySimilarity": 0.45,
        "genreSimilarity": 0.92,
        "spectralSimilarity": 0.67,
        "rhythmicSimilarity": 0.78
      },
      "features": {
        "audio1": {
          "tempo": 120.5,
          "key": "C#m",
          "genre": "ambient"
        },
        "audio2": {
          "tempo": 118.2,
          "key": "Em",
          "genre": "ambient"
        }
      },
      "recommendation": "High compatibility for collaboration"
    }
  }
}
```

### Get Supported Formats

Retrieve list of supported audio formats and their specifications.

**Endpoint:** `GET /formats`

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "formats": [
      {
        "extension": "mp3",
        "mimeType": "audio/mpeg",
        "maxSize": 52428800,
        "quality": "lossy",
        "supported": true
      },
      {
        "extension": "wav",
        "mimeType": "audio/wav",
        "maxSize": 52428800,
        "quality": "lossless",
        "supported": true
      },
      {
        "extension": "flac",
        "mimeType": "audio/flac",
        "maxSize": 52428800,
        "quality": "lossless",
        "supported": true
      }
    ],
    "limits": {
      "maxFileSize": 52428800,
      "maxDuration": 1800,
      "minDuration": 5,
      "maxBatchSize": 10
    }
  }
}
```

## Analysis Types

### Basic Analysis
Fast analysis including:
- Tempo detection
- Key detection
- Duration calculation
- Basic spectral features

**Processing Time:** ~2-5 seconds
**Use Case:** Quick preview, real-time applications

### Full Analysis
Comprehensive analysis including:
- All basic features
- Genre classification
- Mood analysis
- Advanced spectral features
- Custom metrics

**Processing Time:** ~10-30 seconds
**Use Case:** Complete profile analysis, matching algorithms

### Custom Analysis
User-specified analysis including:
- Selected metrics only
- Configurable parameters
- Custom model weights

**Processing Time:** Variable
**Use Case:** Specialized applications, research

## Custom Metrics Details

### Dynamics Quotient
Measures the variation in loudness over time, indicating how dynamic vs. static the audio is.

**Formula:** `DQ = std(RMS) / mean(RMS)`
**Range:** 0.0 - 2.0+
**Interpretation:**
- 0.0 - 0.3: Very static/consistent volume
- 0.3 - 0.7: Moderate dynamics
- 0.7 - 1.2: High dynamics
- 1.2+: Extreme dynamics

### MAD Divergence
Measures how much the harmonic content diverges from traditional Western musical scales.

**Formula:** `MAD = mean(|chroma - closest_scale_template|)`
**Range:** 0.0 - 1.0
**Interpretation:**
- 0.0 - 0.2: Strongly tonal, traditional scales
- 0.2 - 0.5: Moderately tonal with some dissonance
- 0.5 - 0.8: Atonal or experimental harmony
- 0.8 - 1.0: Highly dissonant or noise-based

### Texture Complexity
Analyzes the complexity of audio texture based on spectral feature variations.

**Formula:** `TC = weighted_variance(spectral_features)`
**Range:** 0.0 - 1.0
**Interpretation:**
- 0.0 - 0.3: Simple, uniform texture
- 0.3 - 0.6: Moderate complexity
- 0.6 - 0.8: Complex, varied texture
- 0.8 - 1.0: Highly complex, chaotic texture

## Rate Limiting

### Request Limits
- **Per IP Address**: 100 requests per hour
- **Per User**: 500 requests per day (authenticated)
- **Batch Analysis**: 10 files maximum per request
- **File Upload**: 10 uploads per minute

### Processing Limits
- **Concurrent Analysis**: 5 files simultaneously
- **Queue Size**: 50 pending analyses
- **Timeout**: 5 minutes per analysis

## Error Handling

### Error Response Format
```json
{
  "success": false,
  "error": {
    "code": "INVALID_AUDIO_FORMAT",
    "message": "Unsupported audio format. Please use MP3, WAV, FLAC, M4A, or AAC.",
    "details": {
      "providedFormat": "video/mp4",
      "supportedFormats": ["audio/mpeg", "audio/wav", "audio/flac"]
    }
  },
  "timestamp": "2025-07-19T20:00:00Z",
  "requestId": "req_abc123def456"
}
```

### Common Error Codes
- `INVALID_AUDIO_FORMAT` - Unsupported file format
- `FILE_TOO_LARGE` - File exceeds size limit
- `AUDIO_CORRUPTED` - Audio file is corrupted or unreadable
- `DURATION_TOO_SHORT` - Audio shorter than minimum duration
- `DURATION_TOO_LONG` - Audio longer than maximum duration
- `ANALYSIS_FAILED` - Audio processing failed
- `RATE_LIMIT_EXCEEDED` - Too many requests
- `PROCESSING_TIMEOUT` - Analysis took too long
- `INSUFFICIENT_AUDIO_DATA` - Not enough audio content to analyze

## Integration Examples

### JavaScript/TypeScript

```typescript
// Upload and analyze audio file
const analyzeAudio = async (audioFile: File): Promise<AnalysisResult> => {
  const formData = new FormData()
  formData.append('audio', audioFile)
  formData.append('analysis_type', 'full')
  formData.append('return_waveform', 'true')
  
  const response = await fetch('/api/audio-service/analyze', {
    method: 'POST',
    body: formData
  })
  
  if (!response.ok) {
    throw new Error(`Analysis failed: ${response.statusText}`)
  }
  
  const result = await response.json()
  return result.data.analysis
}

// Usage
const fileInput = document.getElementById('audio-upload') as HTMLInputElement
const file = fileInput.files?.[0]

if (file) {
  try {
    const analysis = await analyzeAudio(file)
    console.log(`Detected tempo: ${analysis.basic.tempo} BPM`)
    console.log(`Key: ${analysis.basic.key}`)
    console.log(`Genre: ${analysis.advanced.genre.primary}`)
  } catch (error) {
    console.error('Analysis failed:', error)
  }
}
```

### Python Client

```python
import requests
import json
from pathlib import Path

class AudioAnalysisClient:
    def __init__(self, base_url: str = "http://localhost:8000"):
        self.base_url = base_url
    
    def analyze_file(self, file_path: Path, analysis_type: str = "full") -> dict:
        """Analyze a single audio file."""
        with open(file_path, 'rb') as f:
            files = {'audio': (file_path.name, f, 'audio/mpeg')}
            data = {
                'analysis_type': analysis_type,
                'return_waveform': 'true'
            }
            
            response = requests.post(
                f"{self.base_url}/analyze",
                files=files,
                data=data,
                timeout=60
            )
            
        response.raise_for_status()
        return response.json()['data']['analysis']
    
    def compare_files(self, file1: Path, file2: Path) -> dict:
        """Compare two audio files."""
        with open(file1, 'rb') as f1, open(file2, 'rb') as f2:
            files = {
                'audio1': (file1.name, f1, 'audio/mpeg'),
                'audio2': (file2.name, f2, 'audio/mpeg')
            }
            
            response = requests.post(
                f"{self.base_url}/compare",
                files=files,
                timeout=120
            )
            
        response.raise_for_status()
        return response.json()['data']['comparison']

# Usage
client = AudioAnalysisClient()

# Analyze single file
analysis = client.analyze_file(Path("cosmic_jam.mp3"))
print(f"Tempo: {analysis['basic']['tempo']} BPM")
print(f"Key: {analysis['basic']['key']}")
print(f"Dynamics: {analysis['customMetrics']['dynamicsQuotient']['value']}")

# Compare two files
comparison = client.compare_files(
    Path("track1.mp3"), 
    Path("track2.mp3")
)
print(f"Similarity: {comparison['overallSimilarity']:.2f}")
```

### Java/Spring Boot Integration

```java
@Service
public class AudioAnalysisService {
    
    private final RestTemplate restTemplate;
    private final String audioServiceUrl;
    
    public AudioAnalysisService(
        RestTemplate restTemplate,
        @Value("${audio.service.url}") String audioServiceUrl
    ) {
        this.restTemplate = restTemplate;
        this.audioServiceUrl = audioServiceUrl;
    }
    
    public AnalysisResult analyzeAudio(MultipartFile audioFile) {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.MULTIPART_FORM_DATA);
        
        MultiValueMap<String, Object> body = new LinkedMultiValueMap<>();
        body.add("audio", audioFile.getResource());
        body.add("analysis_type", "full");
        
        HttpEntity<MultiValueMap<String, Object>> requestEntity = 
            new HttpEntity<>(body, headers);
        
        ResponseEntity<ApiResponse<AnalysisResponse>> response = 
            restTemplate.exchange(
                audioServiceUrl + "/analyze",
                HttpMethod.POST,
                requestEntity,
                new ParameterizedTypeReference<ApiResponse<AnalysisResponse>>() {}
            );
        
        if (response.getStatusCode().is2xxSuccessful() && 
            response.getBody() != null && 
            response.getBody().isSuccess()) {
            return response.getBody().getData().getAnalysis();
        }
        
        throw new AudioAnalysisException("Failed to analyze audio file");
    }
}
```

## Performance Considerations

### Processing Time Guidelines
- **Short files (<1 min)**: 2-10 seconds
- **Medium files (1-5 min)**: 10-30 seconds  
- **Long files (5-30 min)**: 30-120 seconds

### Optimization Strategies
1. **Use Basic Analysis** for real-time applications
2. **Batch Processing** for multiple files
3. **Async Processing** for long files
4. **Caching** for repeated analyses
5. **File Preprocessing** to optimize formats

### Resource Usage
- **CPU**: High during analysis (multi-core recommended)
- **Memory**: 100-500MB per concurrent analysis
- **Storage**: Temporary files during processing
- **Network**: Bandwidth for file uploads

## Monitoring and Observability

### Health Metrics
- Service uptime and availability
- Average processing time per file type
- Error rates by error type
- Queue depth and processing capacity

### Performance Metrics
- Request rate and response time
- File processing throughput
- Resource utilization (CPU, memory)
- Cache hit rates

### Logging
- All API requests and responses
- Analysis processing steps
- Error details and stack traces
- Performance timing information

<!-- TODO: Add webhook support for async processing -->
<!-- TODO: Implement audio preprocessing pipeline -->
<!-- TODO: Add machine learning model versioning -->
<!-- TODO: Document advanced configuration options -->