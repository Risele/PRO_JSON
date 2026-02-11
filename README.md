# PRO_JSON
Bugfixes of Codesys PRO_JSON library by Pro Electric (Author: TVM)

Source: https://forge.codesys.com/lib/pro-json/home/Home/

## Version 19

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
