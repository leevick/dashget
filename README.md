# DashGet

A Python toolkit for working with DASH (Dynamic Adaptive Streaming over HTTP) video content, including downloading segments and analyzing fMP4 representations.

## Tools

### dashget - DASH Segment Downloader

Downloads DASH video segments from MPD manifests with multi-threaded support and resume capability.

### mp4dump - fMP4 Representation Analyzer

Analyzes fMP4 DASH representations using Bento4's mp4dump tool. Generates detailed MP4 structure dumps for each representation.

## Features

### dashget Features

- **DASH Manifest Parsing**: Automatically parses MPD (Media Presentation Description) files
- **Multi-threaded Downloads**: Configurable parallel download threads for faster downloads
- **Resume Support**: Smart resume functionality - skips already downloaded complete segments
- **File Verification**: Verifies file completeness by comparing local and remote file sizes
- **Template Support**: Handles DASH URL templates with `$RepresentationID$` and `$Number$` variables
- **Duration Parsing**: Supports multiple duration formats including ISO 8601
- **Thread-safe Output**: Coordinated output from multiple download threads

### mp4dump Features

- **Automatic fMP4 Detection**: Identifies and processes only fMP4 format representations
- **Selective Processing**: Process all representations or filter by specific IDs
- **Structure Analysis**: Generates detailed MP4 box structure dumps with file offsets
- **Smart File Discovery**: Automatically finds MPD manifests and segments in directories
- **Centralized Output**: Saves all dump files in the manifest directory for easy access

## Installation

### Requirements

- Python 3.6+
- `isodate` library for ISO 8601 duration parsing (for dashget)
- Bento4 tools (for mp4dump) - specifically the `mp4dump` command

### Install Python Dependencies

```bash
pip install isodate
```

### Install Bento4 (for mp4dump)

```bash
# On Ubuntu/Debian
sudo apt-get install bento4

# On macOS with Homebrew
brew install bento4

# Or download from: https://github.com/axiomatic-systems/Bento4
```

### Make Scripts Executable

```bash
chmod +x dashget mp4dump
```

## Usage

### dashget - Download DASH Content

#### Basic Usage

```bash
./dashget <DASH_MANIFEST_URL>
```

#### With Custom Thread Count

```bash
./dashget <DASH_MANIFEST_URL> --threads 8
```

#### Examples

```bash
# Download with default 4 threads
./dashget https://example.com/video/manifest.mpd

# Download with 8 parallel threads
./dashget https://example.com/video/manifest.mpd -t 8

# Download with maximum threads (use with caution)
./dashget https://example.com/video/manifest.mpd --max-threads 16
```

#### Command Line Options

- `dash_url`: URL of the DASH manifest (.mpd file) - **required**
- `-t, --threads`: Maximum number of parallel download threads (default: 4)
- `--max-threads`: Alias for `--threads`

### mp4dump - Analyze and Concatenate fMP4 Representations

#### Basic Usage

```bash
# Process all fMP4 representations in current directory
./mp4dump

# Process all fMP4 representations in specific directory
./mp4dump -d /path/to/dash/content

# Process specific representation(s) by ID
./mp4dump -r 1
./mp4dump -r 1 2 3

# Combine options
./mp4dump -d /path/to/dash/content -r 2
```

#### Command Line Options

- `-d, --directory`: Directory containing DASH manifest (default: current directory)
- `-r, --reps`: Specific representation ID(s) to process (default: all fMP4 representations)

#### Output Files

For each processed representation with ID `{rep_id}`, a dump file is created in the same directory as the manifest:
- `{rep_id}.dump` - Detailed MP4 box structure with file offsets (saved in manifest directory)

#### Examples

```bash
# After downloading with dashget, analyze all representations
./dashget https://example.com/video/manifest.mpd
./mp4dump

# Only analyze video representation "1"
./mp4dump -r 1

# Process multiple specific representations
./mp4dump -r video1 audio1

# Work with content in different directory
./mp4dump -d ~/Downloads/dash_content -r 2
```

## Workflow Example

Complete workflow for downloading and analyzing DASH content:

```bash
# 1. Download DASH content
./dashget https://example.com/video/manifest.mpd

# 2. Analyze all fMP4 representations
./mp4dump

# 3. Or analyze specific representation
./mp4dump -r video1

# 4. View the generated dump file (saved in manifest directory)
less video1.dump
```

## How It Works

### dashget Workflow

1. **Downloads the MPD manifest** from the provided URL
2. **Parses the manifest** to extract:
   - Base URLs
   - Representation information
   - Segment templates
   - Duration and timing information
3. **Creates download tasks** for each representation's segments
4. **Downloads segments in parallel** using configurable thread pool
5. **Verifies completeness** and resumes interrupted downloads

### mp4dump Workflow

1. **Locates MPD manifest** in the specified directory
2. **Parses manifest** to identify all representations
3. **Filters fMP4 representations** (skips non-fMP4 formats)
4. **For each representation**:
   - Finds initialization segment and media segments
   - Runs `mp4dump --verbosity 3` on each segment
   - Writes output with file offsets to `.dump` file in the manifest directory
5. **Generates summary** of processed representations

## Output

### dashget Output

The tool creates a directory structure matching the DASH manifest organization:
- Downloads the manifest file (`.mpd`)
- Creates subdirectories for different representations
- Downloads initialization segments and media segments
- Provides progress output and download summary

### mp4dump Output

For each representation, a dump file is created in the same directory as the manifest:
- **Dump file** (`{rep_id}.dump`): Text file with detailed MP4 structure including:
  - Box hierarchy and types
  - File offsets in decimal and hexadecimal
  - Box sizes and attributes
  - Sample information

## Thread Safety

DashGet uses thread-safe mechanisms for:
- Coordinated console output
- File existence checking
- Download verification

## Error Handling

- Graceful handling of network errors
- File size verification and re-download when needed
- Detailed error reporting for failed downloads
- Automatic retry logic for incomplete segments
- Skip non-fMP4 representations in mp4dump
- Validation of required tools (mp4dump command)

## Contributing

Contributions are welcome! Please feel free to submit issues and enhancement requests.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.