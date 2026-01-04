# Disk Shadow Creator

A powerful Bash script for creating and maintaining symlink-based "shadow" copies of directory structures, with advanced support for macOS Finder aliases, batch processing, and safety features.

## Overview

Creates a mirror directory structure using symlinks instead of copying files. Perfect for:
- Creating lightweight shadows of large music/media libraries
- Organizing collections without duplicating data
- Following Finder aliases to their resolved locations
- Incremental updates of existing shadows

## Key Features

### 🔗 Symlink Shadow Creation
- Creates directory structures with symlinks pointing to original files
- Zero disk space usage for file content (only directory structure overhead)
- Changes to source files immediately reflect in shadow

### 🎯 Finder Alias Resolution
- **Standard mode**: Resolves aliases for symlink targets
- **Follow-alias mode** (`-followalias`): Resolves aliases and creates shadow at resolved path structure
- Batch alias resolution for 29x performance improvement
- Transparent handling of both regular files and aliases

### 🛡️ Safety Features
- Protects existing regular files from being overwritten by symlinks
- Detects dangerous path operations (target inside source, infinite recursion)
- Concurrent execution detection and prevention
- Validates paths before processing

### ⚡ Performance Optimizations
- Batch alias resolution: ~29x faster than individual resolution
- Skip mtime checks (`-skipmtime`): 20-30x faster incremental updates
- Only-update mode: Process only new or changed items
- Efficient path-based sync for large datasets

### 🎵 Music Library Specific
- Handles special characters in filenames (apostrophes, quotes, brackets)
- Excludes common system files (.DS_Store, .Trashes, etc.)
- Preserves album art and metadata files
- Letter-folder organization support via `-followalias`

## Installation
- comes with CoverFrame
- needs alias_resolve, what also comes with CoverFrame

## Usage

### Basic Syntax

```bash
./shadowcopy [options] <source_path(s)> <shadow_path>
```

### Common Options

| Option | Description |
|--------|-------------|
| `-dryrun` | Preview what would be done without making changes |
| `-followalias` | Resolve aliases and create shadow at resolved path structure |
| `-noalias` | Don't resolve Finder aliases at all |
| `-onlyupdate` | Only update symlinks if source is newer or missing |
| `-skipmtime` | Skip mtime checks (20-30x faster, safe for path-based sync) |
| `-mirror` | Mirror source contents directly (don't create source directory name) |
| `-force` | Allow overwriting existing regular files (⚠️ DANGEROUS) |
| `-allowdanger` | Allow potentially dangerous path operations (⚠️ VERY DANGEROUS) |
| `-maxdepth N` | Limit recursion depth |
| `-nosizecheck` | Don't check folder sizes |

## Examples

### Example 1: Basic Shadow Copy

Create a shadow of a music collection:

```bash
./shadowcopy \
  "/Volumes/FW300RAID/-A-" \
  "/Users/username/Sites/music/FW300SHADOW/"
```

**Result:**
```
Source: /Volumes/FW300RAID/-A-/Adagio/2009-Archangels in Black/1 Vamphyri.mp3
Shadow: /Users/username/Sites/music/FW300SHADOW/-A-/Adagio/2009-Archangels in Black/1 Vamphyri.mp3 → [symlink]
```

### Example 2: Import Folder with Aliases (Standard Mode)

Process an import folder containing Finder aliases:

```bash
./shadowcopy \
  "/Volumes/FW300RAID/ Import to iTunes/Nov 2025" \
  "/Users/username/Sites/music/FW300SHADOW/"
```

**Result:**
```
Source structure:
/Volumes/FW300RAID/ Import to iTunes/Nov 2025/
  └─ Adagio-Archangels in Black {Power Metal} [alias → /Volumes/FW300RAID/-A-/Adagio/2009-Archangels in Black]

Shadow structure (preserves import folder structure):
/Users/username/Sites/music/FW300SHADOW/Nov 2025/
  └─ Adagio-Archangels in Black {Power Metal}/
      ├─ 1 Vamphyri.mp3 → /Volumes/FW300RAID/-A-/Adagio/2009-Archangels in Black/1 Vamphyri.mp3
      ├─ 2 The Astral Pathway.mp3 → /Volumes/FW300RAID/-A-/Adagio/2009-Archangels in Black/2 The Astral Pathway.mp3
      └─ cover.jpg → /Volumes/FW300RAID/-A-/Adagio/2009-Archangels in Black/cover.jpg
```

### Example 3: Import Folder with Aliases (Follow-Alias Mode) ⭐

Process import folder and create shadow at resolved locations:

```bash
./shadowcopy -followalias \
  "/Volumes/FW300RAID/ Import to iTunes/Nov 2025" \
  "/Users/username/Sites/music/FW300SHADOW/"
```

**Result:**
```
Source structure:
/Volumes/FW300RAID/ Import to iTunes/Nov 2025/
  ├─ Adagio-Archangels in Black {Power Metal} [alias → /-A-/Adagio/2009-Archangels in Black]
  ├─ Avatar-Don't Go In The Forest {Heavy Metal} [alias → /-A-/Avatar/2025-Don't Go In The Forest]
  └─ BB & The Bullets-High Tide {Blues Rock} [alias → /-B-/BB & The Bullets/2025-High Tide]

Shadow structure (follows resolved paths):
/Users/username/Sites/music/FW300SHADOW/
  ├─ -A-/
  │   ├─ Adagio/2009-Archangels in Black/
  │   │   ├─ 1 Vamphyri.mp3 → /Volumes/FW300RAID/-A-/Adagio/2009-Archangels in Black/1 Vamphyri.mp3
  │   │   └─ cover.jpg → /Volumes/FW300RAID/-A-/Adagio/2009-Archangels in Black/cover.jpg
  │   └─ Avatar/2025-Don't Go In The Forest/
  │       └─ 01 Tonight We Must Be Warriors.mp3 → [symlink]
  └─ -B-/
      └─ BB & The Bullets/2025-High Tide/
          └─ 01 Something In The Water.mp3 → [symlink]
```

**Use Case:** Perfect for organizing imported albums into your main library structure while keeping the import folder clean.

### Example 4: Dry Run Preview

Preview changes before executing:

```bash
./shadowcopy -dryrun -followalias \
  "/Volumes/FW300RAID/ Import to iTunes/Nov 2025" \
  "/Users/username/Sites/music/FW300SHADOW/"
```

**Output:**
```
=== CHECKING FOR CONCURRENT EXECUTION ===
✅ No other instances detected

Source path(s):
  /Volumes/FW300RAID/ Import to iTunes/Nov 2025
Found 1 source path(s)

=== SAFETY CHECK: Analyzing paths for dangerous operations ===
✅ Path safety check passed

DRY RUN MODE - No changes will be made
Follow-aliases mode:
  Original: /Volumes/FW300RAID/ Import to iTunes/Nov 2025/Adagio-Archangels in Black {Power Metal}
  Resolved: /Volumes/FW300RAID/-A-/Adagio/2009-Archangels in Black
  Relative: -A-/Adagio/2009-Archangels in Black
  New target: /Users/username/Sites/music/FW300SHADOW/-A-/Adagio/2009-Archangels in Black

Would create symlink: /Users/username/Sites/music/FW300SHADOW/-A-/Adagio/2009-Archangels in Black/1 Vamphyri.mp3
  → /Volumes/FW300RAID/-A-/Adagio/2009-Archangels in Black/1 Vamphyri.mp3
```

### Example 5: Fast Incremental Update

Update existing shadow with new albums only (ultra-fast):

```bash
./shadowcopy -followalias -onlyupdate -skipmtime \
  "/Volumes/FW300RAID/ Import to iTunes/Nov 2025" \
  "/Users/username/Sites/music/FW300SHADOW/"
```

**Performance:**
- Skips existing correct symlinks instantly (path-based check)
- Only processes new or changed items
- 20-30x faster than full mtime checks
- Processes 100+ files/sec vs 3 files/sec

### Example 6: Mirror Entire Volume

Mirror entire disk structure directly (no top-level folder):

```bash
./shadowcopy -mirror \
  "/Volumes/FW300RAID/" \
  "/Users/username/Sites/music/FW300SHADOW/"
```

**Result:**
```
Source: /Volumes/FW300RAID/-A-/Artist/Album/
Shadow: /Users/username/Sites/music/FW300SHADOW/-A-/Artist/Album/  (no "FW300RAID" folder)
```

### Example 7: Multiple Source Paths

Process multiple directories at once:

```bash
./shadowcopy \
  "/Volumes/FW300RAID/-A-" \
  "/Volumes/FW300RAID/-B-" \
  "/Volumes/FW300RAID/-C-" \
  "/Users/username/Sites/music/FW300SHADOW/"
```

### Example 8: Glob Patterns

Use shell glob patterns for batch processing:

```bash
./shadowcopy \
  /Volumes/FW300RAID/-* \
  "/Users/username/Sites/music/FW300SHADOW/"
```

Processes all letter folders: `-A-`, `-B-`, `-C-`, etc.

## Workflow Examples

### Workflow 1: Music Import Pipeline

```bash
# 1. Add new albums as Finder aliases to import folder
#    /Volumes/FW300RAID/ Import to iTunes/Nov 2025/
#      ├─ New-Album-1 [alias]
#      └─ New-Album-2 [alias]

# 2. Preview what will be created
./shadowcopy -dryrun -followalias \
  "/Volumes/FW300RAID/ Import to iTunes/Nov 2025" \
  "/Users/username/Sites/music/FW300SHADOW/"

# 3. Create shadows at resolved locations
./shadowcopy -followalias \
  "/Volumes/FW300RAID/ Import to iTunes/Nov 2025" \
  "/Users/username/Sites/music/FW300SHADOW/"

# 4. Later: Quick incremental update for new additions
./shadowcopy -followalias -onlyupdate -skipmtime \
  "/Volumes/FW300RAID/ Import to iTunes/Nov 2025" \
  "/Users/username/Sites/music/FW300SHADOW/"
```

### Workflow 2: Full Library Shadow Maintenance

```bash
# Initial full shadow creation
./shadowcopy -mirror \
  "/Volumes/FW300RAID/" \
  "/Users/username/Sites/music/FW300SHADOW/"

# Regular incremental updates (fast)
./shadowcopy -mirror -onlyupdate -skipmtime \
  "/Volumes/FW300RAID/" \
  "/Users/username/Sites/music/FW300SHADOW/"
```

## Understanding Modes

### Standard Mode (default)
- Resolves aliases for symlink targets
- Preserves source directory structure in shadow
- Symlinks point to resolved alias targets

### Follow-Alias Mode (`-followalias`)
- Resolves aliases and extracts resolved path structure
- Creates shadow matching the resolved location's organization
- Perfect for import folders with aliases to organized library

### Mirror Mode (`-mirror`)
- Mirrors source contents directly into target
- Doesn't create source directory name in shadow
- Useful for whole-disk shadows

## Performance Statistics

Typical output from script execution:

```
=== Summary ===
Directories created:      45
Symlinks created:         678
Skipped (up-to-date):     0
Protected files:          0
Finder aliases resolved:  12
Batch resolve time:       2s
Errors:                   0
Total duration:           8 seconds
Resolution rate:          6/sec
Processing rate:          84 files/sec
```

With `-skipmtime` on existing shadow:
```
Processing rate:          120 files/sec  (vs 3-4 files/sec without)
```

## Safety Features

### Protected File Prevention

By default, the script **refuses to overwrite regular files** with symlinks:

```
PROTECTED: Refusing to overwrite regular file: /path/to/file.mp3
 Use -force to override (not recommended)
```

### Dangerous Path Detection

Automatically detects and prevents:
- Target inside source (would destroy source data)
- Source inside target (infinite recursion)
- Same source and target paths
- Critical system directories

```
🚨🚨🚨 CRITICAL DANGER DETECTED! 🚨🚨🚨
The requested operation would likely cause DATA LOSS!

OPERATION ABORTED FOR YOUR SAFETY!
```

### Concurrent Execution Prevention

Detects and warns about multiple instances:

```
⚠️ CONCURRENT EXECUTION DETECTED!
Found separate running instances of this script:
  PID 12345: ./shadowcopy /Volumes/FW300RAID/ ...

Options:
  1. Press Ctrl+C to abort this instance
  2. Kill other instances and continue
  3. Continue anyway (dry-run mode only)
```

## Exclusion Patterns

Automatically skips:
- `.DS_Store` (macOS metadata)
- `.Trashes` (trash folders)
- `.fseventsd` (filesystem events)
- `.Spotlight-V100` (Spotlight index)
- `*Cache*` (cache directories)
- `node_modules` (Node.js dependencies)

## Requirements

- **Bash 3.2+** (compatible with macOS default)
- **Optional:** `alias_resolve` tool for optimal Finder alias performance
  - Install: `brew install al45tair/alias/alias`
  - Without it: falls back to slower individual resolution

## Exit Codes

- `0` - Success
- `1` - Errors occurred (count shown in summary)

## Tips & Best Practices

1. **Always use `-dryrun` first** to preview changes
2. **Use `-followalias`** for import folder workflows with aliases
3. **Use `-onlyupdate -skipmtime`** for fast incremental updates
4. **Use `-mirror`** for whole-disk shadows
5. **Monitor the summary** for errors and statistics
6. **Don't use `-force`** unless absolutely necessary
7. **Check concurrent execution warnings** to avoid conflicts

## Limitations

- Text-only symlink targets (no binary data in symlinks themselves)
- Requires read access to source
- Requires write access to target
- Finder alias resolution requires `alias_resolve` for best performance
- Rate limited by filesystem operations

## Troubleshooting

### "Batch resolution failed"
- Install `alias_resolve`: `brew install al45tair/alias/alias`
- Script falls back to individual resolution (slower but works)

### "PROTECTED: Refusing to overwrite"
- A regular file exists where symlink would be created
- Use `-force` to override (⚠️ use with caution)
- Or manually remove/move the conflicting file

### Slow performance
- Use `-skipmtime` with `-onlyupdate` for 20-30x speedup
- Ensure `alias_resolve` is installed for batch processing
- Check if filesystem is on slow network/USB connection

### "CONCURRENT EXECUTION DETECTED"
- Another instance is already running
- Wait for it to complete, or kill it as prompted
- Safe to continue in `-dryrun` mode

## Version History

- **V17c** - Added `-followalias` mode for resolved path structure
- **V17b** - Added concurrent execution detection
- **V17a** - Added `-skipmtime` flag for fast updates
- **V17** - Batch alias resolution (29x speedup)

## License

Free to use and modify. No warranty provided.

---

**Author:** Enhanced for music library management with safety-first design.
