# CleanMyMac deleted my 4 GB music archive as "cache" — a forensic post-mortem

*How a Mac "cleaner" app classified a personal Desktop folder as application junk, deleted it without Trash protection, and how I proved it from the app's own logs. Published November 18, 2026. All quotes are verbatim from preserved logs and correspondence; the complete evidence set (application logs, statistics database, filesystem records, correspondence) is preserved and available to journalists on request.*

---

## TL;DR

On June 3, 2026, **CleanMyMac 5** permanently deleted `~/Desktop/TRANCE` — a folder containing roughly **4 GB of music, including irreplaceable sets I mixed myself** — during a routine cleanup, because its junk scanner classified the entire folder as *Spotify's application cache*. It did so because Spotify's preferences happened to list that folder as its "Offline Storage Location". The deletion bypassed the Trash. No warning identified the folder by name as my personal data. Nothing was recoverable: on a modern Mac (Apple Silicon, APFS, FileVault, TRIM), deleted files are gone within moments, and I didn't discover the loss until ten weeks later.

I found the proof in CleanMyMac's own application log:

```
2026-06-03 15:21:45.684 [ScanningCore] ▶️ [*FileRemoverTask<JunkCleanupItemNode<CacheFileModel, Node>>] Started
2026-06-03 15:21:45.883 [CleaningCore:FileRemover] File removed at /Users/~/Desktop/TRANCE
2026-06-03 15:21:48.043 [CleaningCore:FileRemover] File removed at /Users/~/Library/Group Containers/6N38VWS5BX.ru.keepcoder.Telegram/.../postbox/media
2026-06-03 15:21:48.043 [ScanningCore] ⏹️ [*FileRemoverTask<JunkCleanupItemNode<CacheFileModel, Node>>] Ended. Duration: 2.36s
```

`CacheFileModel`. A user-created, user-visible folder on my Desktop, full of MP3/WAV/M4A files the app did not create, deleted recursively as a *cache*, in the same two-second batch as Telegram's media cache.

I reported it to MacPaw on August 13 with complete forensic evidence. Over the following **three months**, across five channels — their support system, their payment provider Paddle, Trustpilot, X, and a final written settlement proposal — MacPaw never once substantively responded. The timeline is below, with receipts.

## What happened

My setup was ordinary: a MacBook Pro (M2 Pro), macOS Sonoma 14.5, FileVault on, Desktop synced via iCloud Drive, CleanMyMac 5 (Plus subscription, build 50504.0.2605191427) installed, Spotify installed.

At some point, Spotify's **Offline Storage Location** came to point at `~/Desktop/TRANCE` — my personal music archive. I don't remember setting it, and how it happened remains unknown; Spotify's settings UI buries this option, and a misclick in a folder picker is enough. Spotify then wrote its `index.dat` into the folder, next to my files.

That single preference value was the trap. CleanMyMac's junk scanner reads applications' preferences to locate their caches. It read Spotify's `storage.location`, concluded the folder was Spotify's cache, and presented its contents as removable junk. On June 3 at 15:21 local time, during a cleanup session, it deleted the folder — directly, not via the Trash. CleanMyMac's own `ExtendedStatistics.db` records the batch: **3 items, 4,470,796,288 bytes**, June 3 at 13:21 UTC. Subtracting the two Telegram cache folders (~0.5 GB), my archive was about 4 GB.

Spotify, finding its storage root missing, quietly recreated the folder that evening with a fresh `index.dat` (its creation timestamp — 2026-06-03 17:20:47 UTC — was recorded in the file's metadata before that file, too, was destroyed). So the folder still *existed* on my Desktop, apparently intact, with my files silently gone. I didn't look inside for ten weeks.

## Why nothing was recoverable

By the time I discovered the loss in mid-August, every recovery door had closed, each for a documented reason:

- **Trash:** empty — CleanMyMac's FileRemover deletes directly.
- **iCloud "Recently Deleted":** empty — its window is 30 days; the deletion was on June 3, the window expired around July 3, and I first looked in August.
- **Time Machine / APFS snapshots:** none covering the period (`diskutil apfs listSnapshots` → "No snapshots").
- **Raw recovery:** a full physical-disk scan with Disk Drill from macOS Recovery (698,448 files / 778 GB identified) surfaced only 1,099 reconstructed audio fragments totaling 132 MB — cache debris, no music. On Apple Silicon with APFS, FileVault and TRIM, deleted blocks are invalidated at NAND level within moments, and the storage controller lives inside the SoC behind the Secure Enclave. No lab can bring that back. R-Studio couldn't even open the volume ("hardware encryption module").

The combination deserves emphasis, because it's the part most people don't realize: **on a modern Mac, a deletion you don't notice within days is a deletion forever.** The window where "undelete" tools help effectively no longer exists on this hardware.

## The investigation

Because the folder had been Spotify's "storage location," Spotify was the obvious suspect — the folder even died completely when I later changed Spotify's storage setting (the client deletes its old storage root on migration; destructive behavior in its own right). A first analysis pointed that way. The breakthrough came from preserving and parsing the right artifacts before they rotted: the FSEvents journal, iCloud's CloudDocs databases (`client.db` / `server.db`), the Spotlight store, and — decisively — CleanMyMac's own logs and `ExtendedStatistics.db`, where the deletion was recorded explicitly, timestamped, categorized, and size-accounted. The iCloud server database also confirmed the loss end-to-end: the folder's record reduced to a single child (`index.dat`), the deletion long since propagated.

Two details made the case airtight. The log line names the exact path. And the statistics database independently records the same batch with byte-level totals matching the folder's size. There is no alternative reading: the app's cache cleanup deleted a personal Desktop folder.

## Three months of trying to be heard

Everything below is preserved in writing.

| Date | Event |
|---|---|
| **Aug 13** | Reported to MacPaw support with the log excerpt, statistics record, and full description. Agent replied within the hour asking for their Support Tool diagnostic report — I sent it the same day. Their written reply: *"I've shared it with our engineering team… They'll review it and reach out to you soon."* **No one ever did.** |
| **Aug 27** | Follow-up with a Sept 3 deadline. **No response.** |
| **Sep 8** | Escalated via **Paddle** (MacPaw's merchant of record), who formally forwarded the dispute to MacPaw the same day. **MacPaw never answered Paddle's forwarding.** (Credit where due: Paddle themselves same-day refunded a duplicate subscription and cancelled renewals — the one part of this story where a process worked.) Also discovered MacPaw's account page returned a **500 error** on the cancel-subscription action; I had to cancel by email. Posted publicly on X the same day, tagging @MacPaw. **No response.** |
| **Sep 9** | Posted a factual one-star Trustpilot review. |
| **Sep 11** | MacPaw replied publicly on Trustpilot: *"We would like to review your case thoroughly… please use the 'Provide more information' option… This will allow us to locate the issue and continue working with you directly."* |
| **Sep 23** | Submitted the requested details via Trustpilot's form. **No contact followed.** |
| **Sep 30** | Hours after my review update documenting the silence, MacPaw emailed — a template: *"we would like to check in and ask if your issue is still relevant."* |
| **Oct 3** | Replied in full: complete case summary, three requests (engineering registration in writing, refund, compensation proposal), deadline Oct 10. **No response.** |
| **Oct 22** | Final settlement proposal: 12,000 SEK (~€1,050) covering the destroyed data, documented recovery expenses (dedicated recovery SSD, recovery software), and ~8 hours of diagnostic work. Deadline Nov 3. **No response.** |
| **Nov 18** | This post. |

## Lessons — for users

1. **Audit where apps point their storage.** Open Spotify → Settings → Storage *today* and check the "Offline storage location." If any app's cache/storage path points inside Desktop, Documents, or any folder you care about, move it. That one setting cost me the archive.
2. **"Cleaner" apps deserve zero trust with personal folders.** Any tool that deletes by *classification* rather than by explicit path will eventually misclassify. If you run one, exclude your personal folders explicitly — or don't run one.
3. **Deletions on modern Macs are final, fast.** APFS + TRIM + hardware encryption means recovery tools are largely a relic. Your protection is backups and the Trash — which is why software that bypasses the Trash is removing your last safety net.
4. **iCloud sync is not a backup.** It replicated the deletion within moments, and its undo window (30 days) was shorter than my discovery time. Time Machine on an external disk would have saved everything. Mine wasn't running. That part is on me.
5. **If it happens to you, preserve first, investigate second.** The artifacts that solved this case rot quickly: `~/Library/Group Containers/*CleanMyMac*/Library/Logs/` (the app logs every removal), `ExtendedStatistics.db` (byte totals per cleanup), `/System/Volumes/Data/.fseventsd`, and `~/Library/Application Support/CloudDocs/session/db/`. Copy them to an external drive before anything else.

## Lessons — for developers of cleanup tools

A cache cleaner must never assume that everything inside a third-party app's configured storage path is disposable. The folder it's pointed at can be — as here — a user's home for their own files, with the app's cache merely squatting in it. Minimum bar: verify the files being deleted were *created by* the app whose cache is claimed (Spotify's cache files have a known structure; my WAVs did not match it), route every removal through the Trash, and show users the actual paths of what's about to be destroyed, not a category total. Any of those three would have prevented this.

## On MacPaw

I want to be precise rather than angry. The deletion was a design flaw, not malice — a classification heuristic that will bite anyone whose folders end up in an app's preference file. Software has bugs; what defines a company is what happens next. What happened next, in my case, was: a written promise that engineering would follow up (broken), a formal escalation through their own payment provider (ignored), a public commitment on Trustpilot (not followed up), and a reasonable, itemized settlement proposal (never answered). Three months, five channels, zero substantive response — while the product continues to be marketed as safe.

Draw your own conclusions. Mine is on my Desktop: a folder named TRANCE, recreated by Spotify, forever empty.

---

*Written by the affected user (Sweden). Case ticket MacPaw #1ZE21P-X42KE · Paddle order #33347125. Full evidence preserved: CleanMyMac application logs and ExtendedStatistics.db, FSEvents journal, iCloud CloudDocs databases, Spotlight stores, Disk Drill scan results, and all correspondence. Verification material and identity available to journalists and researchers on request via the repository owner.*
