# Engineering Best Practices Guide

A comprehensive guide covering software engineering best practices, including code quality, testing strategies, documentation, and more.

## Building the Book

This project uses [mdBook](https://rust-lang.github.io/mdBook/) to generate the documentation.

### Prerequisites

- mdBook installed ([installation instructions](https://rust-lang.github.io/mdBook/guide/installation.html))
- Vale (optional, for prose linting)

### Build Locally

```bash
mdbook build
```

The generated book will be in the `book/` directory. Open `book/index.html` in your browser.

### Serve Locally

```bash
mdbook serve
```

This will start a local web server at `http://localhost:3000` with live reload.

## Linting

This project uses [Vale](https://vale.sh/) for prose linting to maintain consistent writing quality.

### Installing Vale

**macOS:**
```bash
brew install vale
```

**Linux:**
```bash
# Download latest release
wget https://github.com/errata-ai/vale/releases/download/v3.0.7/vale_3.0.7_Linux_64-bit.tar.gz
tar -xvzf vale_3.0.7_Linux_64-bit.tar.gz
sudo mv vale /usr/local/bin/
```

**Windows:**
```powershell
choco install vale
```

### Running Vale

First, sync the style guides:
```bash
vale sync
```

Then lint the documentation:
```bash
vale src
```

### Vale Configuration

Vale is configured via `.vale.ini` and uses the Microsoft Writing Style Guide. You can customize the rules by editing `.vale.ini`.

## GitHub Pages

The documentation is automatically deployed to GitHub Pages when changes are pushed to the `master` branch.

View the live site at: https://athola.github.io/best-practices/

## Contributing

When contributing to this documentation:

1. Write clear, concise content
2. Follow the existing structure and style
3. Run `vale src` to check your prose
4. Build locally to verify formatting
5. Commit with descriptive messages

## License

This project uses dual licensing to separately cover the code and the documentation content.

### Code License (MIT)

The mdBook configuration, scripts, theme customizations, and other software components are licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.

This allows you to freely use, modify, and distribute the project's code in both open-source and proprietary projects.

### Documentation Content License (CC BY 4.0)

The book content—including all text, images, and creative assets in the `src/` directory—is licensed under the **Creative Commons Attribution 4.0 International License (CC BY 4.0)**. See the [LICENSE-CONTENT](LICENSE-CONTENT) file for details.

You are free to:
- **Share** — copy and redistribute the material in any medium or format
- **Adapt** — remix, transform, and build upon the material for any purpose, even commercially

Under the following terms:
- **Attribution** — You must give appropriate credit, provide a link to the license, and indicate if changes were made

For more information, visit: https://creativecommons.org/licenses/by/4.0/

---

This documentation is maintained by the Engineering Community.
