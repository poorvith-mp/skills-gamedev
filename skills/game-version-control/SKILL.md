---
name: game-version-control
group: Engineering
description: >-
  Set up Git LFS for binary assets, handle unmergeable scene and prefab conflicts, and know when
  Perforce beats Git. Use when configuring Perforce, Git LFS, or binary asset branching.
---

# game-version-control

## Core Philosophy
Game development source control is fundamentally different from traditional web or software engineering. While code is text and merges easily, 80% of a modern game repository consists of multi-gigabyte monolithic binary assets: 4K textures, 3D FBX meshes, WAV audio banks, and binary scene graphs. Merging binary conflicts is mathematically impossible—a concurrent edit on a binary scene or prefab invariably destroys someone's work. Robust game version control requires strict locking paradigms, Git LFS (Large File Storage) or Perforce (Helix Core), and rigorous branching strategies tailored for cross-functional artist and programmer collaboration.

---

## 4-Step Game Version Control Architecture

### Step 1: Git LFS vs Perforce (Helix Core) Decision Matrix
1. **The Git LFS Model (Indie & AA Studios, < 30 Devs)**:
   - *Strengths*: Cheap hosting (GitHub, GitLab), familiar developer tooling, decentralized branching.
   - *Weaknesses*: Clunky file locking mechanism, slow clone times for 100GB+ repos, pointer sync overhead.
2. **The Perforce Helix Core Model (AAA Studios & Large Art Teams)**:
   - *Strengths*: Centralized server authority, native exclusive file locking (`p4 lock` / `p4 edit`), instant handling of multi-terabyte depots, granular directory permissions, visual artist UI (P4V).
   - *Rule of Thumb*: If your project repo exceeds 100GB or your team has more than 10 dedicated 3D/audio artists simultaneously editing shared scenes, migrate to Perforce.

### Step 2: Git LFS Configuration & Binary Attribute Tracking
1. **`.gitattributes` Locking Rules**:
   - Every non-mergeable binary asset must have the `lockable` attribute enabled so artists must explicitly obtain a lock before editing:
     ```gitattributes
     # 3D Assets
     *.fbx filter=lfs diff=lfs merge=lfs -text lockable
     *.obj filter=lfs diff=lfs merge=lfs -text lockable
     *.blend filter=lfs diff=lfs merge=lfs -text lockable

     # Textures & Images
     *.png filter=lfs diff=lfs merge=lfs -text
     *.tga filter=lfs diff=lfs merge=lfs -text
     *.psd filter=lfs diff=lfs merge=lfs -text lockable
     *.exr filter=lfs diff=lfs merge=lfs -text

     # Audio
     *.wav filter=lfs diff=lfs merge=lfs -text
     *.ogg filter=lfs diff=lfs merge=lfs -text

     # Unity / Unreal Engine Binaries
     *.unity filter=lfs diff=lfs merge=lfs -text lockable
     *.prefab filter=lfs diff=lfs merge=lfs -text lockable
     *.uasset filter=lfs diff=lfs merge=lfs -text lockable
     *.umap filter=lfs diff=lfs merge=lfs -text lockable
     ```
2. **Locking Workflow**:
   - `git lfs lock Assets/Scenes/MainWorld.unity`
   - Edit, save, and commit.
   - `git push && git lfs unlock Assets/Scenes/MainWorld.unity`

### Step 3: Scene Decomposition & Conflict Prevention
1. **Sub-Scene & Level Streaming Architecture**:
   - Never allow artists and programmers to work in a single monolithic `Main.unity` or `PersistentLevel.umap`.
   - Break levels into additive sub-scenes:
     - `Lighting_Subscene` (Lighting artists only).
     - `Environment_Art_Subscene` (Environment artists only).
     - `Gameplay_Logic_Subscene` (Level designers and programmers).
     - `Audio_Triggers_Subscene` (Sound designers).
2. **Prefab / Blueprint Chunking**:
   - Deconstruct mega-prefabs into modular nested prefabs.

### Step 4: Repository Hygiene & Large File Garbage Collection
1. **Preventing Binary Leaks**:
   - Add automated pre-commit hooks that scan commit payloads. Reject any raw file over 20MB that is not tracked by Git LFS.
2. **LFS Pruning**:
   - Run `git lfs prune --recent` locally to free cached binary pointers without breaking remote history.

---

## Deliverable Format: Production `.gitattributes` & Pre-Commit Hook

```bash
#!/bin/sh
# .git/hooks/pre-commit: Reject un-tracked binary files > 20MB
MAX_SIZE=20971520 # 20MB in bytes

for file in $(git diff --cached --name-only); do
    if [ -f "$file" ]; then
        filesize=$(wc -c < "$file")
        if [ "$filesize" -gt "$MAX_SIZE" ]; then
            # Verify if tracked by LFS
            if ! git check-attr filter "$file" | grep -q "lfs"; then
                echo "[ERROR] File '$file' is $(($filesize/1048576))MB and NOT tracked by Git LFS!"
                echo "Please run: git lfs track '$file' before committing."
                exit 1
            fi
        fi
    fi
done
```

---

## Worked Example: Studio Migration to Sub-Scenes & Git LFS Locks

- **Context**: An 18-person indie studio experienced 3 lost workdays per month due to concurrent edits on Unity's binary `Hub_World.unity` scene.
- **Implementation**: Decomposed `Hub_World` into 4 additive scenes (`Hub_Geometry`, `Hub_Lighting`, `Hub_Gameplay`, `Hub_Props`). Enforced `git lfs lockable` on `.unity` files and enabled Unity Smart Merge (`UnityYAMLMerge`).
- **Result**: Scene merge conflicts dropped from 14/month to 0. Zero hours of lost art progress over a 6-month vertical slice sprint.

---

## Verification Checklist

- [ ] `.gitattributes` covers all binary extensions (`.fbx`, `.png`, `.psd`, `.wav`, `.uasset`, `.unity`).
- [ ] Exclusive file locking (`lockable`) is active on non-mergeable scene and prefab formats.
- [ ] Large levels are split additively into multi-disciplinary sub-scenes.
- [ ] CI / pre-commit hooks prevent binary files > 20MB from leaking into standard Git blobs.
- [ ] Unity Smart Merge (`UnityYAMLMerge`) or Unreal Merge Tool configured for text-based asset resolution.

---

## Anti-Patterns

- **Committing 4K Video/Raw Textures to Git History**: Pushing raw 2GB MP4s or PSDs directly to git without LFS, permanently bloating the `.git` directory forever.
- **Concurrent Monolithic Scene Editing**: Two developers working on the same persistent master scene simultaneously on separate branches without sub-scene isolation.
- **Ignoring Git LFS Locks**: Modifying a locked file locally without acquiring the lock from the remote server first.
