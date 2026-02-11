# PRO_JSON
Bugfixes of Codesys PRO_JSON library by Pro Electric (Author: TVM)

Source: https://forge.codesys.com/lib/pro-json/home/Home/

- **Ver 19**: array issue fixes, rootless json
- **Ver 20**: multidimensional arrays + ver 19


## Version 19 : Array errors fix, rootless json

### Array Export Repair

**Symptom**: JSON export produced malformed structure around arrays (wrong closes / misplaced fields), especially with nested arrays and object depth changes.

**Root cause**: JSONVAR.FB_Init did not correctly close dropped array/object levels when path depth shifted down between adjacent variables.

**Fix**: Added shift-down closure logic in FB_Init to explicitly mark EndArray/EndObject on dropped levels and preserve proper InArray/InObject state.

**Result**: Array/object boundaries became valid; nested array output no longer leaked sibling fields into wrong objects.

### Rootless Option (secondary feature/fix)

**Goal**: Allow JSON output without top wrapper object ({"codesysVarName": {...}}), i.e. emit root content directly.

**Implementation**: Added Rootless switch in STRUCT_TO_JSON and guarded wrapper-level open/close/name emission using root-level filtering.

**Related fix**: Corrected close-bracket behavior under IgnoreNull=TRUE to avoid orphan ] tokens in rootless/minimal output paths.

**Result**:

- `Rootless=FALSE`: legacy wrapped JSON preserved.
- `Rootless=TRUE`: valid rootless JSON emitted.
- `IgnoreNull=TRUE/FALSE` both generate valid JSON.

## Version 20: Multidimensional arrays

- Added robust multidimensional array support to both STRUCT_TO_JSON (compose) and JSON_TO_STRUCT (parse).
- Array handling is now vector-based (ArrayDimCount + ArrayIndexes[]) instead of single-index only.
- Legacy ArrayIndex is still supported as a compatibility mirror (no API break).
- Compose no longer flattens normal 2D/3D arrays into duplicate-key output; nested JSON arrays are emitted correctly.
- Overflow dimensions beyond GPL_JSON.MAX_ARRAY_DIMENSIONS are now policy-driven (flatten-to-last vs non-flatten branch).
- Existing Rootless and IgnoreNull behavior remains unchanged.
- No public interface/signature changes; patch is backward-compatible for existing 1D and non-array usage.
