# Blogwatcher — RSS/Atom Feed Monitoring

Track blog and RSS/Atom feed updates with the blogwatcher-cli tool.

## Installation

```bash
# Go
go install github.com/JulienTant/blogwatcher-cli/cmd/blogwatcher-cli@latest

# Docker
docker run --rm -v blogwatcher-cli:/data ghcr.io/julientant/blogwatcher-cli

# Binary
curl -sL https://github.com/JulienTant/blogwatcher-cli/releases/latest/download/blogwatcher-cli_darwin_arm64.tar.gz | tar xz -C /usr/local/bin blogwatcher-cli
```

## Commands

```bash
# Add blog
blogwatcher-cli add "My Blog" https://example.com

# Scan
blogwatcher-cli scan

# List articles
blogwatcher-cli articles

# Mark read
blogwatcher-cli read 1

# Import OPML
blogwatcher-cli import subscriptions.opml
```