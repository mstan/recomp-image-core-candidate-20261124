# recomp-image-core

This dated repository is retained for existing pinned consumers. Current development and the complete media provider are at [mstan/recomp-image-core](https://github.com/mstan/recomp-image-core). Existing commits and license grants remain available here.


Current licensing and the grant for existing owner contributions are documented in [LICENSE-SCOPE.md](LICENSE-SCOPE.md).

Private reusable host-native, read-only disc-image library. No console runtime or game inputs are required to build it. Production consumers are intentionally unchanged pending combined integration experiments.

## Build

```sh
cmake -S . -B build -DBUILD_TESTING=ON
cmake --build build --parallel 2
ctest --test-dir build --output-on-failure --timeout 0
```

The pinned libchdr source archive builds offline. A parent already providing `chdr-static` can reuse its target; it must enable raw sector data and subcode. Link `recomp-image-core::core`. Source consumers use `add_subdirectory`; no installed binary ABI is promised in this initial extraction.

## Interface

C++: `recomp_image/image_reader.hpp` provides instance-owned `ImageReader` with the extracted mount/sector/track and ISO9660 methods. `recomp_image/cue_sheet.hpp` exposes the reusable CUE parser. C: `recomp_image/image.h` provides opaque lifecycle, track, sector-view and identity operations with exception containment. One owner/thread per reader; different instances are independent. No global mount or writable disk handle exists.

Supported: cooked 2048-byte ISO; raw 2352-byte BIN/IMG; single-/multiple-BINARY-file CUE, INDEX00/01, virtual PREGAP/POSTGAP and CDDA; libchdr-supported CD CHD track modes; generated subchannel Q and SBI replacements inherited from the donor. Raw Mode1 user data starts at 16, Mode2 at 24. Additional views expose Mode2 body (2336) and Form2 payload (2324) when raw geometry permits. Raw audio follows the donor little-endian PCM contract; CHD CDDA is byte-swapped accordingly.

ISO9660 volume/root lookup, root files and one subdirectory level retain donor behavior. Rock Ridge, Joliet, deeper arbitrary paths, Opera filesystem, console authentication, disc-controller timing and console-specific filesystem semantics are not supplied. MODE2/2336 backing files and WAVE/MP3 CUE payloads are explicitly unsupported. Parent CHDs need unavailable parent data and are rejected by libchdr. Complete HLE console integration is a later consumer step.

Images can mount without ISO9660, which permits sector access to other console formats. Access outside the logical disc or an undersized explicit-view buffer fails. Directory reads reject invalid record names and geometry before allocation (64 MiB directory resource guard). CHD hunk storage is capped at 16 MiB. Partial trailing sectors remain unaddressable as in the donor. No inference from file length overrides declared CUE track geometry.

## Identity-v1

`Identity`/`recomp_image_identity` explicitly reads the complete mounted image; it is never an automatic mount cost. It computes a SHA-256 chain with domain `recomp-image-identity-v1`. Starting digest is SHA256(domain). Each append is SHA256(previous_digest || SHA256(bytes) || uint32_le(length)). First append 4-byte LE values individually: sector count, track count, then number/start/pregap/audio for each track. For every LBA append the raw 2352-byte view when available, otherwise its cooked 2048 bytes, then the 12-byte Q packet plus one validity byte. No path or modification-time trust cache contributes. Output is published only on complete success. This is a versioned mounted-media identity, not a standard whole-file SHA256 or a promise that cooked and raw containers have equal identity.

## Validation and limits

Synthetic source-only tests cover original reader geometry, mixed tracks, stored/virtual gaps, SBI Q, ISO9660 lookup, raw Mode1, Mode2 Form2, identity repeatability/change, unchanged source bytes, C interoperability, malformed directories, invalid indices and an uncompressed CHD-v5 fixture. These are component results, not console boot/gameplay or real-compressed-CHD acceptance. Preserve donor licenses and see `PROVENANCE.md`.
