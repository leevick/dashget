# DashGet

A Python toolkit for working with DASH (Dynamic Adaptive Streaming over HTTP) and HLS (HTTP Live Streaming) video content, including downloading segments and analyzing fMP4 representations.

## Tools

### dashget - DASH Segment Downloader

Downloads DASH video segments from MPD manifests with multi-threaded support and resume capability.

### hlsget - HLS Segment Downloader

Downloads HLS video segments from M3U8 playlists with multi-threaded support for all variants and audio tracks.

### dashdump - fMP4 Representation Analyzer

Analyzes fMP4 DASH representations using Bento4's mp4dump tool. Generates detailed MP4 structure dumps for each representation.

## Features

### dashget Features

- **DASH Manifest Parsing**: Automatically parses MPD (Media Presentation Description) files
- **Multi-threaded Downloads**: Configurable parallel download threads for faster downloads
- **Resume Support**: Smart resume functionality - skips already downloaded complete segments
- **File Verification**: Verifies file completeness by comparing local and remote file sizes
- **Template Support**: Handles DASH URL templates with `$RepresentationID$`, `$Number$`, and `$Time$` variables
- **SegmentTimeline Support**: Supports both duration-based and timeline-based segment templates
- **Duration Parsing**: Supports multiple duration formats including ISO 8601
- **Thread-safe Output**: Coordinated output from multiple download threads

### hlsget Features

- **HLS Master Playlist Parsing**: Automatically parses M3U8 master playlists
- **Multi-variant Support**: Downloads all video quality variants automatically
- **Multi-audio Support**: Downloads all audio tracks (e.g., multiple languages)
- **Multi-threaded Downloads**: Configurable parallel download threads for faster downloads
- **File Verification**: Verifies file completeness by comparing local and remote file sizes
- **Initialization Segment Support**: Handles EXT-X-MAP initialization segments
- **Playlist Preservation**: Saves all media playlists for offline playback
- **Thread-safe Output**: Coordinated output from multiple download threads

### dashdump Features

- **Automatic fMP4 Detection**: Identifies and processes only fMP4 format representations
- **Selective Processing**: Process all representations or filter by specific IDs
- **Structure Analysis**: Generates detailed MP4 box structure dumps with file offsets
- **Binary Concatenation**: Creates playable concatenated media files from all segments
- **Hexdump Generation**: Produces hexdump output for low-level debugging
- **Smart File Discovery**: Automatically finds MPD manifests and segments in directories
- **Centralized Output**: Saves all output files in the manifest directory for easy access

## Installation

### Requirements

- Python 3.6+
- `isodate` library for ISO 8601 duration parsing (for dashget)
- Bento4 tools (for mp4dump) - specifically the `mp4dump` command

### Install Python Dependencies

```bash
pip install isodate
```

### Install Bento4 (for dashdump)

```bash
# On Ubuntu/Debian
sudo apt-get install bento4

# On macOS with Homebrew
brew install bento4

# Or download from: https://github.com/axiomatic-systems/Bento4
```

### Make Scripts Executable

```bash
chmod +x dashget hlsget dashdump
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

### hlsget - Download HLS Content

#### Basic Usage

```bash
./hlsget <HLS_MASTER_PLAYLIST_URL>
```

#### With Custom Thread Count

```bash
./hlsget <HLS_MASTER_PLAYLIST_URL> --threads 8
```

#### Examples

```bash
# Download with default 4 threads
./hlsget https://example.com/video/master.m3u8

# Download with 8 parallel threads
./hlsget https://example.com/video/master.m3u8 -t 8

# Download with maximum threads (use with caution)
./hlsget https://example.com/video/master.m3u8 --max-threads 16
```

#### Command Line Options (dashget)

- `dash_url`: URL of the DASH manifest (.mpd file) - **required**
- `-t, --threads`: Maximum number of parallel download threads (default: 4)
- `--max-threads`: Alias for `--threads`

#### Command Line Options (hlsget)

- `hls_url`: URL of the HLS master playlist (.m3u8 file) - **required**
- `-t, --threads`: Maximum number of parallel download threads (default: 4)
- `--max-threads`: Alias for `--threads`

### dashdump - Analyze and Concatenate fMP4 Representations

#### Basic Usage

```bash
# Process all fMP4 representations in current directory
./dashdump

# Process all fMP4 representations in specific directory
./dashdump -d /path/to/dash/content

# Process specific representation(s) by ID
./dashdump -r 1
./dashdump -r 1 2 3

# Combine options
./dashdump -d /path/to/dash/content -r 2
```

#### Command Line Options

- `-d, --directory`: Directory containing DASH manifest (default: current directory)
- `-r, --reps`: Specific representation ID(s) to process (default: all fMP4 representations)

#### Output Files

For each processed representation with ID `{rep_id}`, the following files are created in the same directory as the manifest:
- `{rep_id}.dump` - Detailed MP4 box structure with file offsets and segment filenames
- `{rep_id}.m4s` - Concatenated binary file (init segment + all media segments)
- `{rep_id}.hex` - Hexdump of the concatenated binary file (for debugging)

#### Examples

```bash
# After downloading with dashget, analyze all representations
./dashget https://example.com/video/manifest.mpd
./dashdump

# Only analyze video representation "1"
./dashdump -r 1

# Process multiple specific representations
./dashdump -r video1 audio1

# Work with content in different directory
./dashdump -d ~/Downloads/dash_content -r 2
```

## Workflow Examples

### DASH Workflow

Complete workflow for downloading and analyzing DASH content:

```bash
# 1. Download DASH content
./dashget https://example.com/video/manifest.mpd

# 2. Analyze all fMP4 representations
./dashdump

# 3. Or analyze specific representation
./dashdump -r video1

# 4. View the generated dump file (saved in manifest directory)
less video1.dump
```

### HLS Workflow

Complete workflow for downloading HLS content:

```bash
# 1. Download HLS content (all variants and audio tracks)
./hlsget https://example.com/video/master.m3u8

# 2. All playlists and segments are now available locally
# The master playlist is saved as master.m3u8
# Media playlists are saved with their original names
# All segments are in the hls/ subdirectory

# 3. You can play the downloaded content offline using any HLS player
```

## How It Works

### dashget Workflow

1. **Downloads the MPD manifest** from the provided URL
2. **Parses the manifest** to extract:
   - Base URLs
   - Representation information
   - Segment templates (SegmentTemplate or SegmentTimeline)
   - Duration and timing information
3. **Creates download tasks** for each representation's segments
4. **Downloads segments in parallel** using configurable thread pool
5. **Verifies completeness** and resumes interrupted downloads

### hlsget Workflow

1. **Downloads the master playlist** from the provided URL
2. **Parses the master playlist** to extract:
   - Video variant playlists (different quality levels)
   - Audio media playlists (different languages)
3. **Downloads all media playlists** (.m3u8 files)
4. **Parses each media playlist** to extract:
   - Initialization segments (EXT-X-MAP)
   - Media segment URLs
5. **Downloads all segments in parallel** using configurable thread pool
6. **Verifies completeness** by comparing file sizes

### dashdump Workflow

1. **Locates MPD manifest** in the specified directory
2. **Parses manifest** to identify all representations
3. **Filters fMP4 representations** (skips non-fMP4 formats)
4. **For each representation**:
   - Finds initialization segment and media segments
   - Runs `mp4dump --verbosity 3` on each segment
   - Writes output with file offsets to `.dump` file in the manifest directory
   - Concatenates all segments to `.m4s` file
   - Generates hexdump to `.hex` file
5. **Generates summary** of processed representations

## Output

### dashget Output

The tool creates a directory structure matching the DASH manifest organization:
- Downloads the manifest file (`.mpd`)
- Creates subdirectories as needed for segments
- Downloads initialization segments and media segments
- Provides progress output and download summary

### hlsget Output

The tool creates the following structure:
- Master playlist (e.g., `master.m3u8`)
- All media playlists (e.g., `video_eng=1001000.m3u8`, `audio_eng=64008.m3u8`)
- `hls/` subdirectory containing all segments:
  - Initialization segments (`.m4s` files from EXT-X-MAP)
  - Media segments (numbered `.m4s` files)
- Progress output and download summary

### dashdump Output

For each representation, three files are created in the same directory as the manifest:
- **Dump file** (`{rep_id}.dump`): Text file with detailed MP4 structure including:
  - Box hierarchy and types
  - File offsets in decimal and hexadecimal with segment filenames
  - Box sizes and attributes
  - Sample information
- **Concatenated file** (`{rep_id}.m4s`): Binary file with all segments combined (playable)
- **Hexdump file** (`{rep_id}.hex`): Hexadecimal dump of the concatenated file for debugging

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
- Skip non-fMP4 representations in dashdump
- Validation of required tools (mp4dump command)

## Contributing

Contributions are welcome! Please feel free to submit issues and enhancement requests.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.