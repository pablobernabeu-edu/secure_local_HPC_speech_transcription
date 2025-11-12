# System Architecture

## Core Components

1. **Audio Enhancement Pipeline**

   - Spectral noise reduction using first 0.5s as noise reference
   - Dynamic range compression (4:1 ratio above 0.1 threshold)
   - Signal amplification (3.61x gain)
   - 16kHz resampling for optimised speech recognition
   - Automatic gain control and normalization

2. **Transcription Engine**

   - Default: OpenAI Whisper Large v3 model
   - Optional: Custom HuggingFace models (e.g., other Whisper variants)
   - GPU-accelerated inference with automatic fallback to CPU (Central Processing Unit)
   - Automatic model caching and optimisation
   - Robust error handling and fallback mechanisms

3. **Text Processing Subsystem**

   - **Automatic Quality Improvements**: Intelligent detection and correction of transcription artifacts
     - Massive repetition removal (handles AI (Artificial Intelligence)-generated artifacts like repeated sentences)
     - Punctuation spacing fixes (proper spacing before/after punctuation marks)
     - Enhanced name masking with context awareness to prevent false positives
   - Unicode artifact removal and emoji filtering
   - Repetitive pattern detection and correction
   - Comprehensive spelling correction database
   - **Enhanced personal name masking**: Curated multilingual database with 1,793 names across 9 languages (English, Chinese, French, German, Hindi, Spanish, Italian, Arabic, Polynesian)
     - Intelligent filtering prevents masking common English words (e.g., "The", "Okay", "Yeah", "Well", "But")
     - Case-insensitive name detection for comprehensive coverage
     - Surname-specific detection with [SURNAME] placeholder
     - Optional Facebook database (730K+ first names, 980K+ surnames) for comprehensive global coverage with higher false positive rate
   - Professional formatting and structure standardization

4. **Output Management**
   - Dual format generation (plain text and Microsoft Word documents)
   - Organized directory structure with preserved enhanced audio
   - Comprehensive processing metadata and timestamps

## Processing Pipeline

1. **Audio Enhancement**: Signal conditioning and noise reduction
2. **Transcription**: ML (Machine Learning)-based speech-to-text conversion
3. **Automatic Quality Improvements**:
   - **Repetition Detection**: Removes conspicuous repetitions typically caused by AI transcription glitches (e.g., sentences repeated 80+ times)
   - **Optional Repetition Fixing**: Additional filtering with `--fix-spurious-repetitions` flag to remove subtle repetitive patterns and artifacts introduced by Whisper models
   - **Punctuation Correction**: Fixes spacing issues around punctuation marks, quotation marks and parentheses
   - **Smart Name Masking**: Production-grade privacy protection with global database (730K+ first names, 980K+ surnames)
     - Intelligent filtering prevents masking common English words like "The", "Okay", "Yeah", "Well", "But"  
     - Case-insensitive detection covers names regardless of capitalization
     - Surname-specific detection with [SURNAME] placeholder
     - Comprehensive exclusion list for conversation words, prepositions and common nouns
4. **Text Processing**: Additional cleaning, correction and privacy protection
5. **Output Generation**: Formatted document creation

## Configuration Parameters

| Parameter         | Default                   | Description                           |
| ----------------- | ------------------------- | ------------------------------------- |
| Model             | `openai/whisper-large-v3` | Default transcription model           |
| Sampling Rate     | 16kHz                     | Audio processing frequency            |
| Gain              | 3.61x                     | Signal amplification factor           |
| Compression Ratio | 4:1                       | Dynamic range compression             |
| Noise Alpha       | 2.0                       | Spectral subtraction coefficient      |
| Name Masking      | Optional                  | Privacy protection for personal names |

## Internal Architecture

The system is implemented as a monolithic Python script, `transcription.py`, which integrates all core components:

- **Audio Enhancement**: Spectral noise reduction, dynamic range compression and signal amplification
- **ML Model Integration**: Direct integration with HuggingFace Transformers for Whisper model inference
- **Text Processing**: Comprehensive cleaning, spelling corrections and privacy protection
- **Speaker Attribution**: pyannote.audio integration for speaker diarization
- **Output Generation**: Dual format generation for TXT and DOCX files
- **CLI Interface**: Main workflow orchestration via `transcription.py`

This monolithic architecture ensures simplicity, maintainability and ease of deployment without external module dependencies.
