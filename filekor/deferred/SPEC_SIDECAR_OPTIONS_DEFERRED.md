# Sidecar Location Options (Deferred - NOT IMPLEMENTED)

> **Status:** These options are **NOT IMPLEMENTED** and will not be developed.
> Kept only for historical reference and potential future consideration.

---

## Option A: Per File

**Location:** `archivo.kor` alongside each file

| Pros | Cons |
|------|------|
| Maximum portability | "Clutter" in folders |
| Intelligence travels with file | Many `.kor` files in folder |

**Use case:** When maximum portability is required, files moved individually.

---

## Option B: Centralized

**Location:** `.filekor/sidecars/{hash}.kor`

| Pros | Cons |
|------|------|
| Clean (no clutter in folders) | Lost when moving files |
| Single location for all sidecars | Requires reconstruction on move |

**Use case:** When files stay in place and cleanliness is prioritized.

---

## Current Implementation

**Option C (Default):** Subfolder as `.kor.json`

Located at `carpeta/.kor.json` alongside the folder.

See `SPEC_SIDECAR.md` for current specification.