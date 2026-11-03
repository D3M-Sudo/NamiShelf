# NamiShelf — Domain Data and Persistence Architecture

**Status:** SQLite embedded is owner-approved as the local persistence technology under ADR-013. Driver, SQL access layer and physical schema remain deferred until after M0.  
**Scope:** Local data ownership, identity, persistence, migrations and lifecycle.

## 1. Design principles

- Domain identity must not depend on database row IDs, Mihon numeric source IDs, provider object names, or UI list positions.
- Local operation must remain available when GitHub, WebDAV, trackers or sources are offline.
- Database schema is an implementation detail behind repositories and application services.
- Downloaded page data and cache files are not ordinary backup/sync records.
- Secrets and device-local state are not portable data.
- All schema changes require forward migration, failure handling and a tested recovery path.

## 2. Logical bounded contexts

| Context | Core records | Owner |
|---|---|---|
| Catalog / Sources | Source identity, source metadata, source capability profile | Source subsystem |
| Library | Library entry, source reference, favorite state, category membership | Library domain |
| Reading | Chapter reference, read state, bookmark, progress, last-read data | Reading domain |
| Reader | Reading session and device-local reader preferences | Reader subsystem |
| History | Reading event/history record | History domain |
| Tracking | Tracker association and sync status | Tracking subsystem |
| Downloads | Download job, chapter file manifest, local file state | Download subsystem |
| Extensions | Installed extension metadata, repository provenance, granted profile | Extension host |
| Backup | Backup job, format version and restore report; not a second canonical library | Backup subsystem |
| Synchronization | Device state, operation journal, tombstones, key version metadata | Sync subsystem |
| Preferences | Typed settings split into portable and device-local categories | Settings subsystem |

The backup and sync contexts serialize selected domain data; they must not become alternative owners of the canonical entities.

## 3. Stable identities

Use opaque stable identifiers for NamiShelf-owned entities. Preserve external identifiers as namespaced references:

- SourceRef = source namespace + source-native identifier.
- EntryRef = NamiShelf entry ID + source reference.
- ChapterRef = entry ID + source-native chapter key.
- TrackerRef = tracker namespace + tracker-native record identifier.
- ExtensionRef = repository identity + package identity + artifact version/digest.
- DeviceRef = random per-installation identity, not a hardware fingerprint.

Never assume IDs from two source repositories or installations share a namespace. URL normalization must be source-aware; URLs are not globally unique identifiers.

## 4. Relationship model

Conceptual relationships:
- A LibraryEntry references exactly one SourceRef and may have many ChapterRefs.
- A LibraryEntry may belong to multiple Categories.
- ChapterState belongs to one ChapterRef and contains read/bookmark/progress state.
- History contains ordered reading events and references entries/chapters without owning their metadata.
- Tracking associations are zero-to-many per library entry and reference a tracker namespace.
- Download records reference chapters and one or more local file artifacts through opaque managed storage IDs.
- Extension installation records are separate from source catalog records; uninstalling an extension must not silently delete library entries created from that source.
- Sync journal records mutations to sync-eligible entities; it is not the primary entity store.

## 5. Persistence recommendation

SQLite embedded is the approved reference technology for transactional library, chapter, category, history, tracking and job metadata. M0 must verify the downstream candidate's current SQLite stack, JVM driver compatibility, concurrency behavior, licensing and native packaging. If the existing stack is unsuitable, document evidence and alternatives for explicit owner review; do not silently substitute another database.

Do not lock a new ORM, JDBC/native driver or query framework before inspecting the candidate's SQLDelight/AndroidX SQLite adaptations and JVM runtime. Prefer explicit migrations and observable SQL over an abstraction that obscures query behavior.

Large chapter/image files belong in managed filesystem storage, not database BLOBs by default. Cache files are disposable and stored separately from durable downloads. Database stores metadata and managed artifact references.

## 6. XDG storage layout

Proposed application-owned layout:

    $XDG_CONFIG_HOME/nami-shelf/
      settings.json (only if small settings are not in DB)
    $XDG_DATA_HOME/nami-shelf/
      database/
      downloads/
      backups/ (only if user selects an app-managed destination)
      extensions/ (trusted metadata/artifacts; execution isolated)
      migrations/
    $XDG_CACHE_HOME/nami-shelf/
      covers/
      pages/
      thumbnails/
      http/
    $XDG_STATE_HOME/nami-shelf/
      logs/
      sessions/
      operation-state/

Use the platform's XDG defaults when variables are unset. Do not place durable data in AppImage mount paths or Flatpak installation paths. Flatpak paths must resolve through its app-specific XDG sandbox. Exact directories may be adjusted after the candidate's existing path abstraction is inspected.

## 7. Portable versus device-local settings

Portable candidates, selected by category:
- application behavior preferences that are not tied to hardware;
- reader defaults that can be interpreted on another device;
- source preferences that do not contain credentials or cookies;
- user-created category and display preferences where stable semantics exist.

Device-local exclusions:
- window geometry and monitor identifiers;
- absolute paths and mount points;
- cache and temporary state;
- hardware/GPU rendering choices;
- OS credential references and secrets;
- provider tokens, tracker credentials, cookies, E2EE keys;
- active session state and process IDs;
- extension binaries and arbitrary executable artifacts.

A setting is portable only when its meaning survives transfer to a different OS account, display, filesystem and device.

## 8. Migration and recovery

- Use ordered, versioned migrations with a schema-version table.
- Before destructive or non-reversible migration, create a verified database checkpoint or backup.
- Migration failure must preserve the original database and provide a recovery report.
- Clean NamiShelf profile is the default.
- Mihon Desktop profile import is explicit opt-in, read-only at source, and reports unsupported/ambiguous records.
- Never point NamiShelf at the old profile and mutate it in place.
- Migration of downloaded files is a separate operation with checksum and relinking semantics.

## 9. Data retention and deletion

Define retention per context. Removing an extension must preserve user library metadata. Removing a library entry must define behavior for history, tracking and downloaded files separately. Cache eviction must never delete durable downloads. Sync tombstones must outlive the maximum supported offline interval or require explicit stale-device recovery.

## 10. Open technical decisions

- SQLite JVM driver/native distribution and SQL access layer.
- Exact entity schema and indexes.
- Whether history is event-based, state-based or hybrid.
- Download artifact layout and checksum policy.
- Portable settings allowlist.
- Retention and delete semantics for linked records.
- Migration/import mapping from the candidate profile.

These are to be resolved from candidate inspection and domain fixtures before schema freeze. SQLite itself is not open for re-selection absent new evidence and an explicit owner decision.

No schema migration, database access or implementation has been performed.
