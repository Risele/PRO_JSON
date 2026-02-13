# PRO_JSON
Bugfixes of Codesys PRO_JSON library by Pro Electric (Author: TVM)

Source: https://forge.codesys.com/lib/pro-json/home/Home/

- **Ver 19**: array issue fixes, rootless json
- **Ver 20**: Ver 19 +  multidimensional arrays
- **Ver 21**: Ver 20 + option to add single long string field to the end of the root object. 
- **PRO_JSON File IO** - set of FBs to read/save JSON structs to text files. Compatible with last version.

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

## Version 21: Long Custom Field Support

- Added optional custom root-level string field injection in `STRUCT_TO_JSON` for long texts.
- Custom field is configured before compose (via `SetCustomField` method)
- Custom value is passed by pointer to String, so no large JSONVAR value copy is required.
- Added explicit precheck for buffer capacity before custom-field append; returns error if it would exceed GPL_JSON.MAX_JSON_STRING.
- Added JSON escaping while writing pointer text (", \, control chars), preserving valid JSON output.
- Existing behavior is unchanged when custom field is not enabled.
