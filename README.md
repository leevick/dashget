# DashGet

A Python command-line tool for downloading DASH (Dynamic Adaptive Streaming over HTTP) video segments from MPD manifests.

## Features

- **DASH Manifest Parsing**: Automatically parses MPD (Media Presentation Description) files
- **Multi-threaded Downloads**: Configurable parallel download threads for faster downloads
- **Resume Support**: Smart resume functionality - skips already downloaded complete segments
- **File Verification**: Verifies file completeness by comparing local and remote file sizes
- **Template Support**: Handles DASH URL templates with `$RepresentationID$` and `$Number$` variables
- **Duration Parsing**: Supports multiple duration formats including ISO 8601
- **Thread-safe Output**: Coordinated output from multiple download threads

## Installation

### Requirements

- Python 3.6+
- `isodate` library for ISO 8601 duration parsing

### Install Dependencies

```bash
pip install isodate
```

### Make Executable

```bash
chmod +x dashget
```

## Usage

### Basic Usage

```bash
./dashget <DASH_MANIFEST_URL>
```

### With Custom Thread Count

```bash
./dashget <DASH_MANIFEST_URL> --threads 8
```

### Examples

```bash
# Download with default 4 threads
./dashget https://example.com/video/manifest.mpd

# Download with 8 parallel threads
./dashget https://example.com/video/manifest.mpd -t 8

# Download with maximum threads (use with caution)
./dashget https://example.com/video/manifest.mpd --max-threads 16
```

## Command Line Options

- `dash_url`: URL of the DASH manifest (.mpd file) - **required**
- `-t, --threads`: Maximum number of parallel download threads (default: 4)
- `--max-threads`: Alias for `--threads`

## How It Works

1. **Downloads the MPD manifest** from the provided URL
2. **Parses the manifest** to extract:
   - Base URLs
   - Representation information
   - Segment templates
   - Duration and timing information
3. **Creates download tasks** for each representation's segments
4. **Downloads segments in parallel** using configurable thread pool
5. **Verifies completeness** and resumes interrupted downloads

## Output

The tool creates a directory structure matching the DASH manifest organization:
- Downloads the manifest file (`.mpd`)
- Creates subdirectories for different representations
- Downloads initialization segments and media segments
- Provides progress output and download summary

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

## Contributing

Contributions are welcome! Please feel free to submit issues and enhancement requests.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.