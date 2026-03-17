# peasy-video

[![crates.io](https://img.shields.io/crates/v/peasy-video)](https://crates.io/crates/peasy-video)
[![docs.rs](https://docs.rs/peasy-video/badge.svg)](https://docs.rs/peasy-video)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

Async Rust client for the [PeasyVideo](https://peasyvideo.com) API — calculate resolution, estimate bitrate, and analyze frame rates for video files. Built with reqwest, serde, and tokio.

Built from [PeasyVideo](https://peasyvideo.com), a comprehensive video processing toolkit offering free online tools for trimming, resizing, generating thumbnails, and creating GIFs from video files. The site includes in-depth guides on video codec selection, compression strategies for web delivery, and a glossary covering concepts from H.264 encoding and frame rates to container formats and color grading.

> **Try the interactive tools at [peasyvideo.com](https://peasyvideo.com)** — [Video Resolution Calculator](https://peasyvideo.com/video/video-resolution/), [Video Bitrate Calculator](https://peasyvideo.com/video/video-bitrate/), [Video Framerate Converter](https://peasyvideo.com/video/video-framerate/), and more.

<p align="center">
  <img src="demo.gif" alt="peasy-video demo — video resolution calculation and codec analysis tools in Rust terminal" width="800">
</p>

## Table of Contents

- [Install](#install)
- [Quick Start](#quick-start)
- [What You Can Do](#what-you-can-do)
  - [Video Analysis Tools](#video-analysis-tools)
  - [Browse Reference Content](#browse-reference-content)
  - [Search and Discovery](#search-and-discovery)
- [API Client](#api-client)
  - [Available Methods](#available-methods)
- [Learn More About Video Tools](#learn-more-about-video-tools)
- [Also Available](#also-available)
- [Peasy Developer Tools](#peasy-developer-tools)
- [License](#license)

## Install

```toml
[dependencies]
peasy-video = "0.2.0"
tokio = { version = "1", features = ["full"] }
```

Or via cargo:

```bash
cargo add peasy-video
```

## Quick Start

```rust
use peasy_video::Client;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new();

    // List available video tools
    let tools = client.list_tools(&Default::default()).await?;
    for tool in &tools.results {
        println!("{}: {}", tool.name, tool.description);
    }

    Ok(())
}
```

## What You Can Do

### Video Analysis Tools

Digital video combines spatial resolution (the number of pixels in each frame) with temporal resolution (the number of frames displayed per second) to create the illusion of motion. A 1080p video at 30 fps produces 30 full 1920x1080 frames every second — over 62 million pixels per second of raw data. Codecs like H.264 (AVC) and H.265 (HEVC) use inter-frame prediction, motion compensation, and transform coding to compress this data by 100-1000x, while newer codecs like AV1 push efficiency even further at the cost of encoding time. PeasyVideo provides calculators and analysis tools for understanding these encoding parameters.

| Tool | Slug | Description |
|------|------|-------------|
| Resolution Calculator | `video-resolution` | Calculate pixel counts, aspect ratios, and display dimensions |
| Bitrate Calculator | `video-bitrate` | Estimate file sizes for different bitrate and duration combinations |
| Framerate Converter | `video-framerate` | Analyze frame rate conversions and motion smoothness trade-offs |

```rust
use peasy_video::Client;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new();

    // Get the resolution calculator for pixel and aspect ratio analysis
    let tool = client.get_tool("video-resolution").await?;
    println!("Tool: {}", tool.name);              // Video resolution calculator name
    println!("Description: {}", tool.description); // How resolution calculation works

    // List all available video tools with pagination
    let opts = peasy_video::ListOptions {
        page: Some(1),
        limit: Some(20),
        ..Default::default()
    };
    let tools = client.list_tools(&opts).await?;
    println!("Total video tools available: {}", tools.count);

    Ok(())
}
```

Learn more: [Video Resolution Calculator](https://peasyvideo.com/video/video-resolution/) · [Video Codecs Explained](https://peasyvideo.com/guides/video-codecs-explained/) · [Video Compression for Web Delivery](https://peasyvideo.com/guides/video-compression-web-delivery/)

### Browse Reference Content

PeasyVideo includes a comprehensive glossary of video engineering terminology and practical guides for common workflows. The glossary covers foundational concepts like H.264 (the most widely deployed video codec, used in everything from Blu-ray discs to web streaming), frame rate (the number of individual frames displayed per second, typically 24 fps for cinema, 30 fps for broadcast TV, and 60 fps for gaming), container formats (MP4 and WebM wrap encoded video and audio streams into a single file), and color grading (the process of altering the color and tone of footage for creative or corrective purposes).

| Term | Description |
|------|-------------|
| [AV1](https://peasyvideo.com/glossary/av1/) | AOMedia Video 1 — royalty-free codec with 30% better compression than H.265 |
| [Frame Rate](https://peasyvideo.com/glossary/frame-rate/) | Frames per second — 24 fps (cinema), 30 fps (broadcast), 60 fps (gaming) |
| [Color Grading](https://peasyvideo.com/glossary/color-grading/) | Creative color adjustment for tone, mood, and visual consistency |

```rust
use peasy_video::Client;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new();

    // Browse the video glossary for codec and encoding terminology
    let glossary = client.list_glossary(&peasy_video::ListOptions {
        search: Some("codec".into()), // Search for video encoding concepts
        ..Default::default()
    }).await?;
    for term in &glossary.results {
        println!("{}: {}", term.term, term.definition);
    }

    // Read a guide explaining video codec selection and trade-offs
    let guide = client.get_guide("video-codecs-explained").await?;
    println!("Guide: {} (Level: {})", guide.title, guide.audience_level);

    Ok(())
}
```

Learn more: [Video Glossary](https://peasyvideo.com/glossary/) · [Video Codecs Explained](https://peasyvideo.com/guides/video-codecs-explained/) · [Video Compression for Web Delivery](https://peasyvideo.com/guides/video-compression-web-delivery/)

### Search and Discovery

The API supports full-text search across all content types — tools, glossary terms, guides, use cases, and format documentation. Search results are grouped by content type, making it easy to find the right tool or reference for any video workflow.

```rust
use peasy_video::Client;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new();

    // Search across all video content — tools, glossary, guides, and formats
    let results = client.search("convert mp4", Some(20)).await?;
    println!("Found {} tools, {} glossary terms, {} guides",
        results.results.tools.len(),
        results.results.glossary.len(),
        results.results.guides.len(),
    );

    // Discover format conversion paths — what can WebM convert to?
    let conversions = client.list_conversions(&peasy_video::ListConversionsOptions {
        source: Some("webm".into()), // Find all formats WebM can be converted to
        ..Default::default()
    }).await?;
    for c in &conversions.results {
        println!("{} -> {}", c.source_format, c.target_format);
    }

    Ok(())
}
```

Learn more: [REST API Docs](https://peasyvideo.com/developers/) · [All Video Tools](https://peasyvideo.com/)

## API Client

The client wraps the [PeasyVideo REST API](https://peasyvideo.com/developers/) with strongly-typed Rust structs using serde deserialization.

```rust
use peasy_video::Client;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new();
    // Or with a custom base URL:
    // let client = Client::with_base_url("https://custom.example.com");

    // List tools with filters
    let opts = peasy_video::ListOptions {
        page: Some(1),
        limit: Some(10),
        search: Some("trim".into()),
        ..Default::default()
    };
    let tools = client.list_tools(&opts).await?;

    // Get a specific tool
    let tool = client.get_tool("video-trim").await?;
    println!("{}: {}", tool.name, tool.description);

    // Search across all content
    let results = client.search("trim", Some(20)).await?;
    println!("Found {} tools", results.results.tools.len());

    // Browse the glossary
    let glossary = client.list_glossary(&peasy_video::ListOptions {
        search: Some("mp4".into()),
        ..Default::default()
    }).await?;
    for term in &glossary.results {
        println!("{}: {}", term.term, term.definition);
    }

    // Discover guides
    let guides = client.list_guides(&peasy_video::ListGuidesOptions {
        category: Some("video".into()),
        ..Default::default()
    }).await?;
    for guide in &guides.results {
        println!("{} ({})", guide.title, guide.audience_level);
    }

    // List format conversions
    let conversions = client.list_conversions(&peasy_video::ListConversionsOptions {
        source: Some("webm".into()),
        ..Default::default()
    }).await?;

    Ok(())
}
```

### Available Methods

| Method | Description |
|--------|-------------|
| `list_tools(&opts)` | List tools (paginated, filterable) |
| `get_tool(slug)` | Get tool by slug |
| `list_categories(&opts)` | List tool categories |
| `list_formats(&opts)` | List file formats |
| `get_format(slug)` | Get format by slug |
| `list_conversions(&opts)` | List format conversions |
| `list_glossary(&opts)` | List glossary terms |
| `get_glossary_term(slug)` | Get glossary term |
| `list_guides(&opts)` | List guides |
| `get_guide(slug)` | Get guide by slug |
| `list_use_cases(&opts)` | List use cases |
| `search(query, limit)` | Search across all content |
| `list_sites()` | List Peasy sites |
| `openapi_spec()` | Get OpenAPI specification |

Full API documentation at [peasyvideo.com/developers/](https://peasyvideo.com/developers/).
OpenAPI 3.1.0 spec: [peasyvideo.com/api/openapi.json](https://peasyvideo.com/api/openapi.json).

## Learn More About Video Tools

- **Tools**: [Video Resolution Calculator](https://peasyvideo.com/video/video-resolution/) · [Video Bitrate Calculator](https://peasyvideo.com/video/video-bitrate/) · [Video Framerate Converter](https://peasyvideo.com/video/video-framerate/) · [All Tools](https://peasyvideo.com/)
- **Guides**: [Video Codecs Explained](https://peasyvideo.com/guides/video-codecs-explained/) · [Video Compression for Web Delivery](https://peasyvideo.com/guides/video-compression-web-delivery/) · [All Guides](https://peasyvideo.com/guides/)
- **Glossary**: [AV1](https://peasyvideo.com/glossary/av1/) · [Frame Rate](https://peasyvideo.com/glossary/frame-rate/) · [Color Grading](https://peasyvideo.com/glossary/color-grading/) · [All Terms](https://peasyvideo.com/glossary/)
- **Formats**: [MP4](https://peasyvideo.com/formats/mp4/) · [WebM](https://peasyvideo.com/formats/webm/) · [All Formats](https://peasyvideo.com/formats/)
- **API**: [REST API Docs](https://peasyvideo.com/developers/) · [OpenAPI Spec](https://peasyvideo.com/api/openapi.json)

## Also Available

| Language | Package | Install |
|----------|---------|---------|
| **Python** | [peasy-video](https://pypi.org/project/peasy-video/) | `pip install "peasy-video[all]"` |
| **TypeScript** | [peasy-video](https://www.npmjs.com/package/peasy-video) | `npm install peasy-video` |
| **Go** | [peasy-video-go](https://pkg.go.dev/github.com/peasytools/peasy-video-go) | `go get github.com/peasytools/peasy-video-go` |
| **Ruby** | [peasy-video](https://rubygems.org/gems/peasy-video) | `gem install peasy-video` |

## Peasy Developer Tools

Part of the [Peasy Tools](https://peasytools.com) open-source developer ecosystem.

| Package | PyPI | npm | crates.io | Description |
|---------|------|-----|-----------|-------------|
| peasy-pdf | [PyPI](https://pypi.org/project/peasy-pdf/) | [npm](https://www.npmjs.com/package/peasy-pdf) | [crate](https://crates.io/crates/peasy-pdf) | PDF merge, split, rotate, compress — [peasypdf.com](https://peasypdf.com) |
| peasy-image | [PyPI](https://pypi.org/project/peasy-image/) | [npm](https://www.npmjs.com/package/peasy-image) | [crate](https://crates.io/crates/peasy-image) | Image resize, crop, convert, compress — [peasyimage.com](https://peasyimage.com) |
| peasy-audio | [PyPI](https://pypi.org/project/peasy-audio/) | [npm](https://www.npmjs.com/package/peasy-audio) | [crate](https://crates.io/crates/peasy-audio) | Audio trim, merge, convert, normalize — [peasyaudio.com](https://peasyaudio.com) |
| **peasy-video** | [PyPI](https://pypi.org/project/peasy-video/) | [npm](https://www.npmjs.com/package/peasy-video) | [crate](https://crates.io/crates/peasy-video) | **Video trim, resize, thumbnails, GIF — [peasyvideo.com](https://peasyvideo.com)** |
| peasy-css | [PyPI](https://pypi.org/project/peasy-css/) | [npm](https://www.npmjs.com/package/peasy-css) | [crate](https://crates.io/crates/peasy-css) | CSS minify, format, analyze — [peasycss.com](https://peasycss.com) |
| peasy-compress | [PyPI](https://pypi.org/project/peasy-compress/) | [npm](https://www.npmjs.com/package/peasy-compress) | [crate](https://crates.io/crates/peasy-compress) | ZIP, TAR, gzip compression — [peasytools.com](https://peasytools.com) |
| peasy-document | [PyPI](https://pypi.org/project/peasy-document/) | [npm](https://www.npmjs.com/package/peasy-document) | [crate](https://crates.io/crates/peasy-document) | Markdown, HTML, CSV, JSON conversion — [peasyformats.com](https://peasyformats.com) |
| peasytext | [PyPI](https://pypi.org/project/peasytext/) | [npm](https://www.npmjs.com/package/peasytext) | [crate](https://crates.io/crates/peasytext) | Text case conversion, slugify, word count — [peasytext.com](https://peasytext.com) |

## License

MIT
