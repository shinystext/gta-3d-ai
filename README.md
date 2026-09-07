# GTA Asset Search

**Early alpha — work in progress.** Local, evidence-backed asset discovery for GTA modding with humans and AI agents. Search filenames and metadata, inspect visual packets, describe what you actually see, then retrieve those descriptions with optional multilingual text embeddings.

This is not a bundled GTA dataset, an image-recognition service, or a one-click scene generator. You supply your own local game files and an agent/human capable of inspecting images. Blender CLI stays the rendering and building workflow.

## What this enables

Ask an agent for a café, a sheriff office, a street market, or suitable props without relying only on obscure filenames. The useful loop is:

**Local assets → metadata index → rendered views → real visual descriptions → lexical / semantic shortlist → visual verification → Blender construction.**

The pictures below are examples from our wider AI-assisted Blender CLI workflow, using native assets and custom geometry. They show the kind of work asset discovery supports. **This repository does not include their scene generators or downloadable game assets, and search alone did not create these scenes.**

| Coffee shop | LSSD interior |
| --- | --- |
| ![The Daily Grind café, rendered in Blender](docs/images/coffee-shop.png) | ![Sheriff reception, rendered in Blender](docs/images/lssd.png) |
| Grove Street night market | Open-world employment agency |
| ![Night market with reused street props](docs/images/grove-market.png) | ![Employment agency on the upper terrace](docs/images/employment-agency.png) |

## Install and try it without GTA

Python **3.12 recommended**, Python 3.10+ supported by the core. SQLite must include FTS5. No server, SSH access, paid API key, or Blender installation is needed for this toy demo.

```sh
git clone https://github.com/Dryxio/gta-asset-search.git
cd gta-asset-search
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install -e .
python examples/make_demo.py --out output/demo
asset-catalog --db output/demo/catalog.sqlite search 'red chair'
asset-catalog --db output/demo/catalog.sqlite stats
asset-catalog-visual --db output/demo/catalog.sqlite prepare \
  --keys output/demo/keys.json --overrides output/demo/overrides.json \
  --out output/demo/packets
```

On Windows, create the environment with `py -3.12 -m venv .venv` and activate it with `.venv\Scripts\Activate.ps1`. If your Python lacks `venv`/`ensurepip`, `uv venv --python 3.12 .venv` and `uv pip install --python .venv/bin/python -e .` are an alternative on macOS/Linux.

The demo generates two original toy images with opaque names and explicit synthetic descriptions. It tests plumbing, **not GTA search quality**. Use a new output directory when rerunning it.

### Optional semantic search

```sh
python -m pip install -e '.[semantic]'
asset-catalog-semantic --db output/demo/catalog.sqlite build
asset-catalog-semantic --db output/demo/catalog.sqlite search \
  'a place to sit down' --kind model --hybrid
```

This embeds **text**, including descriptions made after visual inspection, with `intfloat/multilingual-e5-small` at a pinned revision. The first use downloads model weights from Hugging Face; subsequent runs use the local cache. No image is automatically sent to a vision provider. Unannotated objects cannot acquire visual meaning simply by embedding their filenames. The default annotation corpus and the opt-in metadata corpus stay distinct; every semantic response reports coverage. Similarity scores are not probabilities.

## Use your own San Andreas files

The initial source adapter targets **classic PC San Andreas**, not Definitive Edition or console archives. Keep generated output outside your game installation:

```sh
asset-catalog-sources --game-root '/path/to/GTA San Andreas' \
  --out output/my-sa-source
asset-catalog --db output/my-sa.sqlite build --root output/my-sa-source --game sa
asset-catalog --db output/my-sa.sqlite search 'office desk' --kind model --limit 20
asset-catalog --db output/my-sa.sqlite jobs --limit 40
```

The adapter reads active IDE declarations in `data/gta.dat`, `data/default.ide`, optional `--ide FILE` extras, IMG v2 archives under `models/`, optional `--archive FILE` extras, and loose PC TXDs. It writes model and texture **metadata**, with a `source-report.json` listing failures. It does not export game binaries or pixels, modify the game, include inactive mod assets automatically, inventory all standalone clothing DFFs, or compute placements/dimensions. A dictionary may contain texture names whose pixels have not been visually inspected.

Already have a catalogue? Supply the [documented JSON layout](docs/catalog-schema.md) with `build --root`. Namespaces support `sa`, `vc`, and `gta3`; full VC/III source adapters and validation are future work.

**A fresh installation starts without our local annotations.** Render a bounded batch and have an agent/human review it before expecting useful visual semantic retrieval. See the [agent guide](AGENTS.md) and [Blender CLI workflow](docs/blender-cli.md).

## What works in this alpha

- Incremental SQLite/FTS5 import, bilingual lexical hints, game/kind/dimension filters, provenance and explicit partial matches.
- Local multilingual text embeddings, stale-vector exclusion and hybrid reciprocal-rank fusion.
- Local image overrides, immutable evidence packets, blind contact sheets, confidence/ambiguity tracking and evidence-hash checks.
- Query-specific shortlists with explicit constraints, separate from general-purpose annotations.
- Bounded campaign planning, resumable four-view Blender rendering, exact pixel transfer with parent provenance, and conservative render-signal diagnostics.
- Exact-byte scans and an optional cache deduplication planner; source game assets are never consolidated.

## Limits and roadmap

**Coverage is incomplete.** Existing unit tests do not prove that every relevant object is found. Missing models, transparent/ambiguous atlases, source variants, skinned geometry, and unreviewed appearances remain real gaps. Static rendering rejects skinned DFFs instead of inventing a successful preview. Metadata and source changes still need deliberate rebuild/review; this is not a filesystem watcher.

Our earlier local campaign indexed 50,017 entries (16,838 models including extra mod entries, and 33,179 texture occurrences). Its first enrichment checkpoint passed with 1,730 usable direct visual descriptions at that historical snapshot. **Those data and annotations are not distributed, those counts are not a standard vanilla-game inventory, and that checkpoint does not establish exhaustive recall.**

The public [issues](https://github.com/Dryxio/gta-asset-search/issues) track:

- Independent relevance / missed-candidate evaluation (CP1).
- End-to-end scale, interruption recovery, freshness, throughput and storage validation (CP3).
- Complete model review and honest unresolved-case accounting (CP4).
- Texture occurrence coverage, exact pixels, alpha and real model/atlas context (CP5).
- Later VC/III adapters and new-asset maintenance (CP6, outside the original SA campaign).

See [roadmap criteria](docs/roadmap.md), [validation](docs/validation.md), and [architecture / costs](docs/architecture.md).

## Tests

```sh
python -m unittest discover -s tests -p 'test_asset_catalog*.py'
# Optional real pretrained model smoke, after installing .[semantic]:
ASSET_SEMANTIC_INTEGRATION=1 python -m unittest discover -s tests -p test_asset_catalog_semantic.py
# Optional Node extraction helpers:
npm ci --ignore-scripts
npm test
```

Default tests generate synthetic fixtures in temporary directories and need no game or network. The real embedding smoke downloads weights if uncached. CI runs the offline suite; it does not ship GTA data or promise gameplay validation.

## License and data

Original tool code: [MIT](LICENSE). [Third-party notices](THIRD_PARTY_NOTICES.md) cover separately installed dependencies, including GPL-licensed DragonFF. No DragonFF code, game binaries, texture packs, private catalog database, or reviewed GTA annotation dataset is bundled. Demo screenshots illustrate modding work and retain the rights of their underlying game content; they are not an asset redistribution license. Not affiliated with Rockstar Games or Take-Two Interactive.
