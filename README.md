# CLIF 101

A gentle introduction to coding with CLIF (Common Longitudinal ICU data Format) datasets.

🌐 **Live site:** [https://common-longitudinal-icu-data-format.github.io/CLIF-101/](https://common-longitudinal-icu-data-format.github.io/CLIF-101/)

## What's Inside

- **Loading Data** - Use clifpy to efficiently load and explore CLIF tables
- **Validation** - Quality control against CLIF schemas and mCIDE vocabularies
- **Analysis Patterns** - Common workflows for ICU research
- **Project Template** - Structure your work for federated collaboration
- **Best Practices** - Tips and gotchas from the CLIF community

## Quick Start

```bash
# Install clifpy
pip install clifpy

# Load your CLIF data
from clifpy import ClifOrchestrator

clif = ClifOrchestrator(data_dir="path/to/clif/data")
clif.load_tables(["patient", "hospitalization", "vitals", "labs"])
```

## Local Development

To run the site locally:

```bash
# Install Jekyll
gem install bundler jekyll

# Serve locally
bundle exec jekyll serve

# Visit http://localhost:4000/CLIF-101/
```

## Contributing

Found an issue or want to add content? PRs welcome!

1. Fork the repo
2. Create a branch (`git checkout -b add-new-section`)
3. Make your changes
4. Submit a PR

## License

MIT License - see [LICENSE](LICENSE) for details.

---

Part of the [CLIF Consortium](https://clif-icu.com)
