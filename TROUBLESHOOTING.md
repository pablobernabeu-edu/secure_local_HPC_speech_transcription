# Troubleshooting

## Known Issues to Avoid

### Python and Package Version Compatibility

**Known Issues:**
- ❌ Python 3.13 + PyTorch 2.5.1: No wheels available (torchvision 0.20.1 missing)
- ❌ PyTorch 2.7.1+cpu on GPU nodes: Symbol mismatch with torchcodec (`undefined symbol: _ZNK3c107SymBool...`)
- ❌ PyTorch 2.8.0+cpu: May have torchcodec compatibility issues on some systems
- ❌ Pyannote 4.0.1: Requires PyTorch 2.8.0+ (incompatible with stable 2.5.1)

**Recommended Versions (HPC Production):**
- **Python**: 3.12.x (full wheel availability, production-stable)
- **PyTorch**: 2.5.1+cpu (last stable version before breaking ABI changes)
- **TorchVision**: 0.20.1+cpu
- **TorchAudio**: 2.5.1+cpu
- **Transformers**: 4.45.0+
- **Pyannote.audio**: 3.1.0-3.3.x (NOT 4.x - requires torch>=2.8)

**Why these specific versions?**
- **Python 3.12**: Full wheel availability, production-stable
- **PyTorch 2.5.1**: Last stable version before 2.7.x introduced breaking ABI changes
- **TorchCodec compatibility**: PyTorch 2.5.1 works with transformers audio backend
- **Pyannote 3.x**: Compatible with PyTorch 2.5.1 (v4.x requires torch>=2.8.0)

## Common Issues and Solutions

### 1. Audio File Issues

**"Audio file not found" error**
- Ensure audio files are in `audio_input/` directory
- Check file extensions (`.wav`, `.mp3`, `.m4a`, etc.)
- Use `hpc/submit_transcription.sh` to auto-detect files

**Supported audio formats:**
- `.wav`, `.mp3`, `.m4a`, `.flac`, `.ogg`, `.aac`

### 2. Model Download Issues

**Model download failures**
- Check internet connectivity
- Verify HuggingFace model name
- Use default model: `openai/whisper-large-v3`
- Pre-download models: `python setup/download_model.py`

**Timeout during batch processing:**
- Run `python setup/download_model.py` on login node before submitting batch jobs
- This caches models locally to avoid network timeouts

### 3. GPU and Memory Issues

**GPU memory errors**
- Reduce batch size or use CPU processing
- Check available GPU memory with `nvidia-smi`
- Use `--memory` flag to increase allocation

**"No GPU detected"**
- Ensure job submitted to HTC cluster: `--clusters=htc`
- Check GPU constraints are properly applied
- Verify GPU availability: `nvidia-smi`

**Out of memory errors**
- Increase memory allocation with `--memory 64G` or `--memory 128G`
- Use CPU mode if GPU memory insufficient
- Process shorter audio segments

### 4. SLURM Job Issues

**SLURM job failures**
- Check resource allocation in batch script
- Verify account and partition settings
- Monitor job logs for specific errors

**Timeout errors**
- Jobs may timeout based on partition limits
- Use `--time-limit` flag to increase time allocation
- Consider using medium partition for longer jobs

**Job not starting:**
- Check queue status: `squeue --me`
- Verify partition availability
- Check account quotas

### 5. Environment Issues

**"Environment not found"**
- Verify conda environment exists in `$DATA/speech_transcription_env`
- Run setup script: `python setup/install_requirements.py`
- Activate environment: `source activate_project_env.sh`

**Module loading failures**
- Scripts fall back to default versions
- Check `module avail` for available versions
- Load specific modules manually if needed

**Package installation to wrong location:**
- Always activate `activate_project_env.sh` BEFORE installing packages
- Packages should install to `$DATA` (project space), not `~/.local/` (personal quota)

### 6. Processing Issues

**Transcription quality issues:**
- Enable audio enhancement: `--enhance-audio`
- Use repetition fixing: `--fix-spurious-repetitions`
- Specify language: `--language english` (or other language)
- Try a different model if quality is poor

**Name masking false positives:**
- Use `--language english` to enable automatic common word filtering
- Exclude specific names: `--exclude-names-from-masking "Name1,Name2"`
- Use curated database instead of Facebook database (default behavior)

**Speaker attribution not working:**
- Verify pyannote setup: `python setup/setup_pyannote.py`
- Check HuggingFace token is configured
- See [PYANNOTE_SETUP_GUIDE.md](PYANNOTE_SETUP_GUIDE.md) for detailed setup

### 7. Output Issues

**Missing output files:**
- Check `output/transcripts/` directory
- Review job logs for errors: `tail -f transcription_<job_id>_<task_id>.out`
- Verify disk space: `df -h $DATA`

**Incorrect output format:**
- System generates both `.txt` and `.docx` files by default
- With speaker attribution, additional `_with_speakers` versions are created

## Processing Modes: Batch vs Single File

**IMPORTANT:** The system supports two processing approaches:

### 1. Batch Processing (Recommended for Multiple Files)

Process ALL audio files in the `audio_input/` directory:

```bash
# Processes all files with default settings
hpc/submit_transcription.sh

# With name masking enabled
hpc/submit_transcription.sh --mask-personal-names

# Force English-only transcription
hpc/submit_transcription.sh --language english --mask-personal-names

# Combine multiple options
hpc/submit_transcription.sh --mask-personal-names --language english --fix-spurious-repetitions
```

The system automatically detects all audio files and creates appropriate job arrays.

### 2. Single File Processing

Process one specific audio file:

```bash
# Using submit_transcription.sh (recommended)
hpc/submit_transcription.sh --single-file audio_input/interview.wav --mask-personal-names

# With custom output name
hpc/submit_transcription.sh --single-file audio_input/recording.wav --output-name "client_meeting" --mask-personal-names
```

## Performance Troubleshooting

### Slow Processing

**If transcription is taking too long:**
- Verify GPU is being used: check job logs for GPU detection
- Check GPU utilization: `nvidia-smi`
- Consider shorter audio segments
- Use appropriate time limits: `--time-limit 06:00:00`

### Expected Performance

- Single file processing: ~40 minutes per hour of audio
- Batch processing: Linear scaling with available GPU resources
- Memory usage: 8-16GB per concurrent job
- Disk I/O: Enhanced audio files require 2x original file size

## Debug Steps

1. **Check SLURM output files:** `slurm-<jobid>_<taskid>.out`
2. **Verify GPU allocation:** `scontrol show job <jobid>`
3. **Test environment manually:** `srun -p interactive --gres=gpu:1 --pty /bin/bash`
4. **Check disk space:** `df -h $DATA`
5. **Review logs:** `tail -f transcription_<job_id>_<task_id>.out`

## Getting Help

### Documentation Resources

- [README.md](README.md) - Main documentation
- [docs/architecture.md](docs/architecture.md) - System architecture details
- [docs/advanced_usage.md](docs/advanced_usage.md) - Advanced features
- [docs/hpc_usage.md](docs/hpc_usage.md) - HPC-specific guidance
- [PYANNOTE_SETUP_GUIDE.md](PYANNOTE_SETUP_GUIDE.md) - Speaker attribution setup

### HPC Support

For ARC-specific issues, consult:
- [ARC User Guide](https://arc-user-guide.readthedocs.io/)
- [ARC Support](https://www.arc.ox.ac.uk/support)

### Reporting Issues

When reporting issues, please include:
- Python version: `python --version`
- PyTorch version: `python -c "import torch; print(torch.__version__)"`
- Error messages from log files
- Command used to run transcription
- Audio file characteristics (duration, format, size)
