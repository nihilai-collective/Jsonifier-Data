# Jsonifier-Data

The ground-truth JSON inputs for [Jsonifier](https://github.com/nihilai-collective/Jsonifier)'s unit tests and [Json-Performance](https://github.com/nihilai-collective/Json-Performance)'s benchmarks, kept in one place with every duplicate removed.

## Using It

Pin it to a specific commit so tests and benchmarks always read the same bytes.

### As a submodule

```sh
git submodule add https://github.com/nihilai-collective/Jsonifier-Data.git Jsonifier-Data
git -C Jsonifier-Data checkout <commit-hash>
git add Jsonifier-Data
git commit -m "Pin Jsonifier-Data to <commit-hash>"
```

The last two commands record the pinned commit in your repository. Without them, the submodule pointer stays at whatever commit was checked out when it was added.

To move to a newer commit later:

```sh
git -C Jsonifier-Data fetch
git -C Jsonifier-Data checkout <new-commit-hash>
git add Jsonifier-Data
git commit -m "Bump Jsonifier-Data to <new-commit-hash>"
```

### With CMake

```cmake
include(FetchContent)
FetchContent_Declare(
	jsonifier_data
	GIT_REPOSITORY https://github.com/nihilai-collective/Jsonifier-Data.git
	GIT_TAG <commit-hash>
)
FetchContent_MakeAvailable(jsonifier_data)
```

This repository has no `CMakeLists.txt`, so `FetchContent_MakeAvailable` only downloads it and adds no targets. Use `jsonifier_data_SOURCE_DIR` to locate the files:

```cmake
target_compile_definitions(my_tests PRIVATE
	JSONIFIER_DATA_DIR="${jsonifier_data_SOURCE_DIR}"
)
```

### Filenames

Many filenames contain spaces and parentheses (for example `Canada Test (Minified).json`). Quote paths in shell scripts and CMake, and percent-encode them in URLs.

## Layout

| Folder | Contents |
| --- | --- |
| `corpus-files/` | Benchmark documents and single-type tests (see below). |
| `corpus-files/package_json/` | Real-world `package.json` files for the package.json parsing benchmarks. `Sources.md` lists where each one came from and its license. |
| `unit-tests/fast_float_corpus/` | Plain-text (not JSON) number lists for float parsing, one value per line: `canada.txt` (full-precision coordinates), `canada_short.txt` (the same coordinates at two decimals), `mesh.txt` (mesh vertex data) and `contrived.txt` (edge cases around 2^53, the largest double, the smallest subnormal, and mantissas hundreds of digits long). |
| `unit-tests/JSONTestSuite/test_parsing/` | The JSONTestSuite parsing cases: `y_` must parse, `n_` must fail, `i_` is implementation defined. |
| `unit-tests/jsonchecker/pass/`, `unit-tests/jsonchecker/fail/` | The json.org JSON_checker conformance cases. |
| `unit-tests/round_trip/` | Minified documents that must serialize back to identical bytes after parsing. |
| `unit-tests/json/` | Inputs for the minify, prettify and validate unit tests. |

### Benchmark documents in `corpus-files/`

- Apache Builds
- Canada
- CitmCatalog
- Discord
- Github Events
- Google Maps Response
- Instruments
- Marine IK
- Random
- Twitter

Each comes in some or all of these variants: Minified, Prettified, Small, Stream (Comma-Separated) and NDJSON.

### Single-type tests in `corpus-files/`

Bool, Double, Int64, Uint64, String, Minify and Prettify.

### Round-trip expectations

Files in `unit-tests/round_trip/` that contain floating-point numbers only round-trip byte for byte if the serializer writes the shortest representation that parses back to the same double. A serializer that uses a fixed precision will fail these files even if every value is correct.

## Duplicates and `aliases.json`

Every data file here is byte-unique. Where two old paths held identical bytes, only one file is kept, and `aliases.json` maps each dropped path to the file that replaced it:

```json
"unit-tests/json/Canada Test (Minified).json": "corpus-files/Canada Test (Minified).json"
```

To load a file by an old path, check `aliases.json` first and fall back to the path itself.

Rules for aliases:

- Aliases are permanent. An entry is never removed once published, so old paths keep resolving at every later commit.
- Every alias points directly at a file that exists. An alias never points at another alias, so a single lookup is always enough.
- If a kept file is renamed or moved, every alias that points at it is updated in the same commit, and its old path is added as a new alias.

## License

See [License](License.md). The third-party datasets (JSONTestSuite, JSON_checker, the fast_float corpus, the classic benchmark documents and the collected `package.json` files) remain under their original authors' terms. The license of each `package.json` file is listed in `corpus-files/package_json/Sources.md`.