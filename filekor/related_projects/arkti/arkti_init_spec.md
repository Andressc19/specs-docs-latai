@Author DAVID SANTANA
@Version 1.0
@LastReview 02-Apr-2026

I will call you Gpy or Ai, IA or AI each time I need something from you (artificial intelligence friend). WATCH OUT of my texts! We can make a good work together!.

Architecture Knowledge Terminal Interface 🤖 (ARKTI) is a Tool like a Swiss Knife that helps you via TUI to administer, find and understand faster and accurately your project files (within a project folder) quickly and do actions on that project folder (and subfolders).

From technical matter works like this

    AKTi TUI  (Termina User Interface)  ---- call ---->  AKTi API (Wrapper Engine ) ---- call ----> Tools/Engines (such us SideCardTagger, Exiftool, Unix commands, etc)

ARKTi somehow is like the evolution of Total Commander (of DOS) but 30 years later using the capabilities of the new tools we can find in Unix and Opensource. ARKTi is NOT Commercial, also not company spread, is just a tool to be used by the Author during his research tasks and projects from a NON Commercial perspective. Therefore it will not infringe any restriction of Private Property, allowing this tools to interact and reuse other tools which have a compatible LICENSE for it.


--------------------------------
1) ARKTi GENERALITIES
--------------------------------

ARKTi turns messy project folders into something you can actually navigate and understand, apply useful commands to erase duplicates, mark garbage files (corrupt or empty or too small), suggest names (based on the metadata extracted, such us filenames, creation date, tags/labels), rename masively (but with human authorization first), obfuscate (for contracts and GDPR sentitive texts), and have a Search Bar in a TUI that quickly locate the project files from a Terminal, so an architect like me can save ours finding and organizing project files. Also Arkti can run commands to reorganize the files based on labels or datatypes so the project folder is completely reorganize in a better structure. 

Arkti can also ZIP (make a backup) of a particular folder (or the whole project) to protect the changes done by the tool (if the user requests it) so it can have rollback capabilities. Additionally Arkti has always a LOG file explaining each renamem, erase, backup, summarize, etc. so the user can always know what he/she was doing in the project with Arkti and not blame Arkti for any misunderstanding.

The index of Arkti  has an hybrid strategy that if the whole project is copied into other path or machine, the index is already build or at least easy to rebuild (not needed to start from scratch). 

Arkti has a viewer for most of the files it generates (JSON indexes, summaries, logs or at least a way to convert into a format that a tool like visualStudio or SUblime can quickly highlight and be human readable) with that the architect not only see the index but he can edit it on purpose to increase the search capabilities and adjust the knowledge. Arkti (at least the search part) is based on the old school project of void tools named EVERYTHING that is super fast finding a particular file inside of the project (this will explain later on)

1B) Some USE CASES for your understanding
ARKti (A for simplicity) scans your selected project folder (ROOT and Subfoders) and:

- shows you what files you actually have (like a specialized DIR command) from an organized perspective
- runs in background something like INDEXING deamon (think of kind of Google Desktop) to evaluate which files are duplicated, poorInRelevance (Weak files % -> corrupt or size too small) and then extracts the metadata and relevant info of each file in hybrid mode (explained later on). 

- When the index is good enough ( parametric like INDEX_QUALITY > 80%) then we can operate the Arkti commands like this:

1. explains what those files contain
2. lets you search by meaning (labels and relevant metadata for naming), not just file name (which can be empty)
3. indicate the groups and labels files automatically
4. suggests the best name (or names) and the best label (or labels)
5. On-Demand summarizing, allows the user to choose a specific folder or group of files (text/doc files) to do the summary.
6. Execute renaming of selected files (files the user agrees on the suggestion), the user can even modifiy the suggestion and then apply the changes. 
7. Obfuscate information of a document (txt/docs) given a obfuscator_dictionary (set of rules to obfuscate) so that a representative TXT file is mimic that document but without sensitive data (GDPR protected)

8. High-level feature set ARKTi should support:

- Project scan
- scan root + subfolders
- inventory files
- collect metadata
- assess usable state
- Indexing
- maintain searchable local index
- support re-scan / refresh
- Search
- search by exact text
- search by filename
- search by labels/tags
- later search by meaning/semantic cues
- Preview
- display metadata
- display extracted content snippets
- display related files / labels
- Summarization
- on-demand summary for one file
- on-demand summary for folder/group of files
- no need to summarize everything automatically in MVP
- Classification / Label suggestion
- propose category + labels
- examples: contract, finance, architecture, config, notes, provider, legal, onboardin
- Rename suggestion
- propose better file names
- later support user-approved rename execution
- File assessment
- assess relevance/health/quality
- identify duplicates, broken files, temporary artifacts, low-value files
- Obfuscation
- create safe representative text output using obfuscation rules
- preserve meaning while masking sensitive data
- Operation safety
- log every action
- support preview
- support rollback for mutating actions in later phases


--------------------------------
2) MVP
--------------------------------
Ai, we are going to build the ARKTI in maaaany iterations with very short steps/increments so we will built useful software even from the very beginning even if the software starts with a Hello World message and a button (TUI), as an Architect I am against long planning phases and extensive code/testing generation at one shot. I hate project where we implement thousands of features each release and have to troubleshoot a lot, in here we are going to go by 2 or 3 features each time and build everything. IN that case not only all the SDLC process is faster but human understable.

for the MVP, we will do some iterations (maybe 3,4 or 5) until something decent is there to present.

I also like to Mockup first most of the CORE Engine interactions (specially with 3rd party tools and github projects) so we can build the rest using those Mockups (interfaces/contracts) and then simply replacing it for the real ones.

For example (Summarize), I will expect 1st iteration simply:

    a) ARKTI engine
        a1) API for Summarize => function summarize(inputFile, restrictions) returns textSummarize
        a2) TUI with a Menu that only has the summarize button and the select file feature
    b) Mock engine
        Tool Mock -> We simulate the elements we need to do for summarization via 3rd parties
            For example we simulate the AI analyse and AI summary from the file give the restrictions provided
                - We return a Fake/Mock summary from a Pool of Mock summaries that we build for this iteration

The approach for this development will be TDD, and Prototyping, it means we will create first all the Test Cases, test files (for iteration 1), Unit Test, Test inputs, Test expected, Results, Api, interfaces. Then we will mock it all and then we will code the proper code for that iteration. After that we will do the whole SDLC process using 'SKILLS' so that we can have a proper tool.

The languages that we will try to use is MAC shell (unix script), javascript / python (because many AI and tools are Python exposed) and Java and C++ (because it is the language that I know). In any case we will use a SKILL also here to have a professional software development ( good commments, examples, identation, best practice in naming conventions, etc). We will try to use the format of Java (even in other languages), it means the style of coding, grouping in classes, strong data typing, looping, etc so that I can read the code no matter the programming language. 

Ai, tell me the best way to get the proper SKILLS that i need for this development, Im sure there is a repository for SDLC that is more or less famous/standard, as well as a methodology to work with such SKILLS (for example Claude Guide how to manage skills, orchestrate, agents, etc.) 
    

--------------------------------
3) MVPIterations until is Presentable/Worth it
--------------------------------
Ai, we will define the SOW (Statement of Work) for each iteration but at the end of all the MVP iterations, we should have in our ARKTI at least the following elements:  

local-only
one project root
support of PDF only
scan files
extract text from text-based PDFs
build searchable metadata
TUI search + preview
on-demand summary
label suggestions
operation log
mock or simple real engine behind TUI

At this level (MVP) I am NOT expecting to see this elements:
- advance image handling
- OCR
- DOCX/PPTX/XLSX
- audio/video
- email processing
- SharePoint integration
- machine-wide daemon
- batch reorganization
- full rename engine
- advanced obfuscation across all formats
- per-folder manifest editing UI
- vacation/team availability utilities
- enterprise workflows

Reason:
- keep scope small
- prove value fast
- build TUI shell and engine contract first




--------------------------------
3) Extra considerations
--------------------------------
- The ARKTi project we will build in /Users/dsantana/develop/arkti/arkti where as I know we can see only:
drwxr-xr-x  6 dsantana   192 Apr  2 14:56 .
drwxr-xr-x  4 dsantana   128 Apr  2 14:56 ..
drwxr-xr-x 12 dsantana   384 Apr  2 14:57 .git
-rw-r--r--  1 dsantana  4688 Apr  2 14:56 .gitignore
-rw-r--r--  1 dsantana 11357 Apr  2 14:56 LICENSE
-rw-r--r--  1 dsantana   281 Apr  2 14:56 README.md


I want us to create folders for AI Specs, folders for TEST files, folders for code and all other good practice folder structure we need. Ai, Do not improvise or overengineer simply look for a good SKILL (or create one) where we use the best practices for software development.

As you can see this document (arkti_init_spec.md) is too big and dense. I expect that we handle better some folder or requirements inside the AI_Specs where we have the init spec but then we have designated MD files for the particular spec/planning of each of the main topics of the SDLC.

Ai, as the best Software Engineer you are, I expect you to challenge me, propose idea, ask me all that you need so we can structure all in a good detail. For example: what should be the structure of the index files, structure in our DB for centralized files, structure of the Summary document files, etc. I think JSON or even YAML are goog languages to represent many data in a human readable way (that the machine can also read).

I expect the tool to be sync with the idea of darInfoDEV/term/dainfo.sh, this means the tool can show a visual aid in a separate terminal window () log of which command its is running in background or executing so the architect can know whats happening behind.



--------------------------------
4) Extra considerations
--------------------------------
Because we can have many iteration before shipping something useful, lets groups multiple iterations in sth called PHASES, then in general lines we can have a Phase roadmap as for example (only suggestions):


Phase 0 — Mock + proof of concept
Goal: Prove TUI, Engine API , connection to mocks and concept and command architecture

I expect this at least to have:
- project scan mock
- fake index status
- fake search results
- fake summaries
- fake labels
- one TUI screen
- engine API contract

Phase 1 — Usable MVP
Goal: real value on PDFs inside one project root

I expect this at least to have:

- file discovery
- PDF text extraction
- root index
- searchable metadata
- preview pane
- on-demand summaries
- label suggestions
- file assessment (relevance/health)
- duplicate detection (basic)
- operation log

Phase 2 — Controlled actions
Goal: support useful but safe operations

Include:

- rename suggestions
- user-approved rename
- backup to zip
- simple rollback strategy
- simple obfuscation (based maybe in simple rules/dictionary)
- optional reorganize on demand

Phase 3 — Richer media and better semantics
Include: images
- OCR
- xlsx/docx/pptx
- audio
- richer duplicate logic
- semantic cross-file linking
- stronger grouping / similarity


--------------------------------
5) Architecture overview (LAYERS)
--------------------------------
ARKTi should be a TUI orchestrator over a pluggable engine. But both are important. The key of the TUI is that abstract me as architect to know all commands and type all the time. I am not reeinventing the wheel I am just facilitating my work in a nice/fancy way. (like TotalCommander did for me 30y ago)

I can also suggest to work by Layers, for example:

Layer 1 — Orchestrator / TUI

Responsibilities:

- render terminal UI
- capture user commands
- dispatch actions
- preview suggestions
- request approval
- display progress and results

Layer 2 — Engine API

Responsibilities:

- expose stable application commands
- abstract underlying tools
- allow real or mock adapters

Layer 3 — Index + metadata

Responsibilities:

- file discovery
- metadata persistence
- summary/label persistence
- file assessment persistence
- duplicate info
- extracted text references

Layer 4 — Search

Responsibilities:

- exact search
- filename search
- label search
- fuzzy search
- later semantic search

Layer 5 — Actions

Responsibilities:

- summarize
- classify
- suggest rename
- apply rename
- obfuscate
- export
- reorganize (later)

Layer 6 — Safety

Responsibilities:

- backups
- logs
- rollback
- no destructive defaults

--------------------------------
5) Architecture Style
--------------------------------

Build ARKTi as:

- a thin TUI shell
- over a mockable engine contract
- backed by tool adapters
- with safe persistent local state
- Anti-pattern to avoid


But be careful, we need good architecture, specially when working with the end points (3rd tool parties and APIs) not:

- bind the TUI directly to sidecar-tagger internals (important to use indirection / abstraction NO direct connection)
- respect the best principles to go thru layers without jump or avoid the hierarchies (FrontEnd cannot open a Database cursor for example)
- bind search directly to one external command with no abstraction
- hardcode all real tools before the UI exists 
- build deep business logic inside the TUI layer

** Engine contract (important)
- Create a stable engine interface first.
- Suggested core engine methods
- scan(root_path)
- refresh(root_path)
- get_index_status(root_path)

- search(query, filters)
- get_file_details(path)
- get_preview(path)

- summarize(path_or_folder)
- suggest_labels(path)
- suggest_rename(path)

- assess_file(path)
- find_duplicates(scope)

- obfuscate(path, policy)

- list_operations()
- rollback(operation_id)
- backup(scope)


This allows:

- mock engine now
- real integrations later
- UI progress without backend dependency
- testing independent from tool availability

* Mock engine requirement

This is a deliberate strategy and should be implemented early. With Mocking and Interfaces:

TUI can be built immediately

no need to wait for sidecar-tagger integration

no need to wait for PDF parser decisions

behavior and UX can be tested first


* Mock engine should fake:
- search results
- summaries
- labels
- rename suggestions
- relevance/health assessments
- progress bars
- duplicate results
- preview snippets

* Mock engine output should be realistic
* Important that structure and data of Mocks(Synthetic data) is Not random nonsense but structured and representative, specially when we simulate the logs, the YAML files, input files, summaries, etc. It should resemble likely real project data.


* TUI design goals:  ARKTi is a TUI, not just a raw CLI. It should feel like:

modern shell tooling

Oh My Zsh / powerlevel10k inspired or OpenCode (so colorful but still intuitive, full of menus and hotkeys to handle commands and so on)

readable

colorful by meaning

keyboard-driven

visually structured

Visual principles
search bar / command input should be large and prominent

multiple panes

clear shortcuts bar

syntax-highlighted previews when possible

color indicates meaning, not decoration

Target layout
top bar: project path, mode, counts, warnings

large search/command input

left pane: results / tree

center pane: metadata / summary

right pane: preview / related / actions

bottom bar: shortcuts

UX inspiration sources
fzf (selection/fuzzy interaction)

broot (tree + search navigation)

bat (preview/highlight)

lazygit (panel-based TUI clarity)

k9s (watch/status mode ideas)

ARKTi should NOT be a pseudo-IDE, or so complicated the iterations takes forever (TUI as all other component should improve progressively thru iterations, I am not expecting to have all TUI final in 1 shot) therefore I dont expect:

a complex dashboard
a bloated platform UI
shell-theme cosplay only

* Example of UX commands (that will internally work with ARKTi command engine)
Search

> arkti --label "finance,2026" --search "provider costs"

Expected behavior:

show count
show ranked results
show labels
show summary snippets
allow preview
Summarize a folder


> arkti --summarize notes_meeting/
Expected behavior:

show progress
show resulting grouped summary
store output
allow open/export


> arkti --obfuscate contract.pdf --out safe_contract.txt
Expected behavior:

detect sensitive entities
replace using rules/policy
produce safe representative text
log action

* Search priorities
Search importance ranking
Search accuracy
Useful preview
Search speed
Search filters
Semantic search later
MVP search types
exact text search
filename search
label/tag search
extracted-content search
fuzzy result selection
Semantic search
Not required in MVP (Ai we could have more examples and discussions about this an other search approches for further interactions)

18. File assessment model (to identify low quality files for the project). For example a JPG that has 1x1 pixel even if its an icon its useless to have in the project or at least to look for it, summarize or add labels and so on. therefore I named this files as low quality files (Ai lets look for a way to name them ... trash maybe is too strong for that), but for example corrupted files, audios or videos of 1 second, documents of 1 page (totally empty), these are examples of some LOW quality file that we should mark with ARKTi and that we could use as candidate to erase.


We should check the quality of files quickly with some heuristics and/or command line commands, that for example if its an image or pdf and is less than 20Kb we can consider that "trash". 


For our file analysis we can use 2 axes:

RELEVANCE
- relevance_score
- relevance_status

HEALTH
- health_score
- health_status

Why two axes? A file can be:
valid but low relevance
broken but highly important
tiny but intentional
duplicate but still useful for audit

Recommended status values

relevance_status
core
useful
auxiliary
low_relevance
review

health_status
valid
degraded
broken
duplicate_candidate
temporary_candidate

Supporting fields
reasons[]

user_review_required

safe_action = label_only

This is better than one field called trash_score (lets find a proper name Ai)

* File assessment heuristics
Use heuristics, not hard absolutes.
Generic likely low-value / review-needed signals
zero bytes
unreadable or corrupt parse result
known temporary extension
broken partial download
invalid container structure
duplicate hash + lower quality variant

Images) Potential low-relevance or review:

width < 16 and height < 16
single-color placeholder
corrupt header
generated thumbnail/cache artifact
duplicate of larger original

Audio) Potential low-relevance or review:
duration < 3–5 sec
corrupt container
near silence only

PDF) Potential low-relevance or review:

zero pages
unreadable
one page with no text and near-empty render
parser claims page but nothing visible/useful

DOCX / PPTX / XLSX (future phases) -> Potential low-relevance or review:

broken ZIP/container
DOCX with no body
PPTX with zero slides or only empty placeholder slide
XLSX with empty sheets only
Text / JSON / XML

For this particular files like (TXT oriented like JSON, TXT, XML, etc) never mark low relevance only because <1 KB since many config files or keys can be handle in very small files.

Instead:
empty => review
invalid and non-useful => low relevance or broken
otherwise keep

Ignore mechanism
ARKTi should support an ignore file similar to .gitignore. so we dont need to process some folders or files (they are somehow excluded from ARKTi processing)

Example concept:
.arktiignore
.cache/
node_modules/
*.tmp
*.DS_Store
thumbnails/

20. Indexing strategy

Hybrid index architecture

Not:
one giant machine-global database OR
one JSON metadata file in every directory as the only truth

Instead:

one root project index
optional manifests / sidecars for portability/caching
one operation log
one cache area
Recommended root structure

projectABC/ ==> project folder 
  .arkti/  ==> hidden ARKti folder
    index.db    ==> central DB
    operations.log ==> Operations of ARKTi 
    config.yaml ==> Some parameters/configs/heuristics for ARKTi in that folder
    cache/
    manifests/

** The index is still in Draft , Ai give me some more ideas, explanations and examples of how do you understand the concepts of cache, manifest, sidecards, etc

The index should contain or reference:

- file inventory
- paths
- hashes
- timestamps
- labels
- summaries
- health/relevance scores
- duplicate relations
- extracted-text references
- operation history references

* I would say during phases/iterations we can also play with different tools/technologies for example:
SQLite for MVP
Some better DB (if needed) when the phases are more robus/complex

operations.log
Append-only operation log Contains:

- scans
- summaries
- rename suggestions
- applied changes
- backups
- rollbacks
- obfuscation actions

cache/
Temporary or regenerable artifacts.

Examples:

extracted text cache
thumbnails
preview cache
parser temp outputs
obfuscation intermediates

manifests/
Portable structured snapshots for folder subtrees.

Examples:
contracts.manifest.json
notes.manifest.json

(Ai this ideas of cache and manifest is still in DRAFT, I need your help. What is a manifest? 
A manifest is a compact, portable metadata snapshot of a folder or file set but IT IS NOT the full main index, 
the original file OR the only truth

It is:
a summary that speeds refresh
a transportable local state hint
a cache/distribution artifact

Example manifest
JSON

{
  "folder": "contracts",
  "scan_time": "2026-04-01T10:42:00Z",
  "file_count": 18,
  "folder_state_hash": "abc123xyz",
  "status": "usable",
  "labels": ["contract", "finance", "provider"],
  "files": [
    {
      "path": "contracts/price-contract.xlsx",
      "type": "xlsx",
      "relevance_status": "core",
      "health_status": "valid",
      "labels": ["finance", "contract", "provider"],
      "summary_ref": "cache/summaries/price-contract.md"
    },
    {
      "path": "contracts/old_tmp_copy.tmp",
      "type": "tmp",
      "relevance_status": "low_relevance",
      "health_status": "temporary_candidate",
      "reasons": ["temp_extension", "low_signal"]
    }
  ]
}

Why manifests help
if a subtree is copied, some intelligence can travel with it
folder workers can parallelize work
partial refresh becomes easier

Why manifests should not be sole truth
worse global search
duplication issues
schema migration harder
consistency harder

** Why hybrid indexing is preferred

Pros
global search stays coherent
local portability still possible
partial folder intelligence can travel
easier rollback and audit
simpler schema migration than fully distributed JSON-only approach
better foundation for TUI speed and ranking

Cons
slightly more complexity than pure DB-only
requires clear cache/manifest invalidation rules

* Decision For MVP:

use SQLite root index
allow optional manifests later
do not overbuild folder-manifest editing in UI

** Our initial target is macOS later on cross-platform, so we do not rely on Windows NTFS internals, 
do not design around MFT or USN Journal, keep filesystem watcher logic abstract

Useful principles to retain from fast search tools:

- fast initial scan
- incremental refresh
- search instantly, preview later
- But implementation should be:
- macOS-friendly first
- portable if possible

* Tool strategy
ARKTi should reuse as much as possible Unix commands, opensource and FREE 3rd parties and only rebuild when necessary from scratch (or code the logic of a functionality).

Rule: prefer external tool when
mature
scriptable
stable
narrow function well-covered
output parseable enough
compatible with license and deployment constraints

Rule: rebuild internally only when
orchestration impossible
output unstable/unstructured
dependency too heavy
license blocks usage
performance insufficient
security/compliance blocks deployment
Incorporation or Replace of the Mock turns very difficult
The effort of coding is maybe easier than the integration of command or tool


------------------
6. EXTERNAL TOOLS AND DEPENDENCIES
------------------

Here are some candidates to consider (specially sidecar-tagger from latai is our MAIN STRONG dependency)
- Ai, I dont know many Unix commands or useful tools, I need your help to understand not only which functionalities could we incorporate in ARKti but also which commands or tools do you think may be useful. DO NOT discard any tool by LICENSE, I will always find a way to use the code or the tool, for example if 

* sidecar-tagger

Role:
- metadata generation
- semantic tags
- suggesting of fileName
- summaries
- sidecar generation
- likely strongest metadata dependency candidate

License: Apache-2.0

Use:
- core metadata adapter candidate
- wrap behind ARKTi interface

Do not:
couple ARKTi to its internal design too tightly

* ripgrep
Role:

exact search fallback
preview snippet extraction
extracted-text search
text-search engine support

License: MIT & Unlicense

Use:
strong candidate for search backend fallback


* fzf
Role:
fuzzy picker behavior
quick select
command palette style UX

License: MIT

Use:
interaction inspiration or optional embedded behavior

* bat
Role:
syntax-highlighted preview rendering

Use:
preview layer inspiration or command integration

* broot
Role:
tree navigation patterns
search with hierarchy awareness

License: MIT

Use:
UX inspiration, possibly optional integration patterns

* lazygit
Role:
panel layout, action discoverability, keyboard UX

Use:

UI inspiration mostly

k9s
Role:

live refresh/status/watch patterns

keyboard-centric object navigation

License: Apache-2.0

Use:

UI/UX inspiration

** Inspiration-only / caution candidates
* File Sorter
Role:

categorization pipeline inspiration

rename suggestion UX

file-type aware processing inspiration

local-AI workflow inspiration

License: AGPL-3.0 (careful with this license - maybe better to study the code and create our own one)

Guidance:
study repo
copy ideas/concepts only
do not reuse code unless deliberately accepting AGPL implications

* Advanced Renamer
Role:
rename workflow inspiration
rollback ideas
advanced rename rule inspiration

License: proprietary/commercial for professional use

Guidance:
do not assume safe reuse in internal job context
better to borrow ideas, not depend on it, or even better Ai find some similar opensource software for do renaming massively.

27. License guidance for implementation
Safe/open enough for direct dependency or integration
MIT
Apache-2.0
MIT/Unlicense dual

This means these are generally good candidates:

sidecar-tagger
ripgrep
fzf
broot
k9s
bat (if used and license checked in actual implementation)

Caution / avoid direct code reuse
AGPL-3.0
  study architecture
  copy ideas
  do not copy code casually

Commercial/proprietary
  use only if vendor terms explicitly permit internal professional use

otherwise avoid as dependency => Ai you can suggest which tools have you found and discarded and the reason, maybe I human, can find a way to use them or decide what to do with them later on.

--------------------------------
7. OBFUSCATION CONCEPT
--------------------------------
ARKTi should eventually support:

- obfuscation of documents to produce safe representative text
dictionary/rule driven or policy-driven masking
preserve semantics but hide sensitive data

Example obfuscation targets
names
emails
phone numbers
addresses
customer ids
account numbers
IBAN
business references

* MVP
Not full feature.
Maybe mock only or phase 2.

--------------------------------
8. Rename / reorganize concept
--------------------------------
Principles
- suggest first
- preview
- allow user edits
- apply only selected changes
- log everything
- support backup and rollback later

* Not for MVP
No full batch rename engine in MVP.

* Future
rename selected files
backup before mutation
rollback action
optional reorganize by labels/type/date on demand

--------------------------------
9. Backup / rollback concept
--------------------------------
Backup
ARKTi may support:
- zip backup of project or folder before changes
  -- this can be improved to zip only the files that CANNOT be changed by ARKti (special careful backup mode - if ignored files are heavy why to back them up?)
- backup only on demand or before mutating operations
Rollback
Mutating operations should eventually support rollback using:
operation log
before/after state
rename maps
backup references

MVP
Only operation logging and perhaps mock rollback UI.

--------------------------------
10. Operation log requirements
--------------------------------
Every action should be logged.

Operation types
scan
refresh
summarize
classify
rename suggestion
rename apply
obfuscate
backup
rollback
reorganize

Each log entry should include
operation id
timestamp
actor/user
scope/path
action type
result
warnings/errors
affected files
summary text or refs
rollback possibility flag

--------------------------------
11. Search and ranking concept
--------------------------------
Search should consider
  filename
  path
  labels
  extracted text
  summary
  type
  folder context

Ranking can consider

  term match strength
  labels match
  file recency
  folder relevance
  summary relevance
  user flags/bookmarks later

Minimum useful result
Every result should ideally show:
  path
  type
  labels
  relevance/health hint
  short summary snippet

--------------------------------
12.Index status concept
--------------------------------
Avoid fake precision like “100% quality” as a business concept. 
Better states (Draft ideas)
  empty
  scanning
  usable
  degraded
  stale
  refreshing

Additional metrics
  files discovered
  files processed
  files with preview
  files with summary
  extraction failures
  duplicates found
  review-needed files

This is more meaningful than one arbitrary percent threshold or simply saying this is "trash"
--------------------------------
13. Config and ignore files
--------------------------------
.arkti/config.yaml

Potential settings:

root path
parsers enabled
summary policy
label policy
ignore rules path
obfuscation policy
cache settings
preview settings
external tools locations



.arktiignore
Purpose: ignore noise
skip known useless folders/files
user-tuned scope control

Examples:
.cache/
dist/
node_modules/
*.tmp
*.DS_Store
thumbnails/
coverage/


--------------------------------
14. Data model suggestions
--------------------------------
File record
Draft (Ai we need to think this carefully together):
JSON
{
  "path": "contracts/price-contract.pdf",
  "name": "price-contract.pdf",
  "extension": "pdf",
  "size_bytes": 120344,
  "modified_at": "2026-04-01T10:00:00Z",
  "hash_sha256": "abc...",
  "labels": ["finance", "contract", "provider"],
  "summary": "Commercial annex with provider pricing...",
  "relevance_score": 92,
  "relevance_status": "core",
  "health_score": 95,
  "health_status": "valid",
  "reasons": [],
  "duplicate_group_id": null,
  "preview_ref": "cache/previews/contracts_price-contract.txt",
  "extracted_text_ref": "cache/text/contracts_price-contract.txt",
  "parser_status": "ok"
}

Search result DTO
JSON
{
  "path": "contracts/price-contract.pdf",
  "score": 0.94,
  "labels": ["finance", "contract"],
  "summary_snippet": "Commercial annex with provider pricing...",
  "health_status": "valid",
  "relevance_status": "core"
}

File assessment DTO
JSON
{
  "path": "tmp/old_copy.tmp",
  "relevance_score": 20,
  "relevance_status": "low_relevance",
  "health_score": 40,
  "health_status": "temporary_candidate",
  "reasons": ["temp_extension", "no_useful_content_detected"],
  "user_review_required": true
}

--------------------------------
15. TUI pages / views suggested
--------------------------------
Page 0 - Credits and Quick FAQ (Manual for Users)


Page 1 — Search workspace
big command/search input
result list
metadata pane
preview pane
bottom shortcuts

Page 2 — Summary execution
folder/file summary action
progress view
output summary
open/export actions

Page 3 — Interactive search wrapper
more immersive full-screen search
command palette
fast panel switching
watch mode later

Page 4 — Obfuscation page
input file
sensitive entities found
masked preview
output path
export options
(Ai we should define the way to represent the Dictionary of Obfuscation)
Ai, it may be possible there is already some advance renaming (inside of files) tool to do basic obfuscation. Maybe research about it, we could find sth useful.

--------------------------------
16. Suggested keyboard actions
--------------------------------

Examples of Hotkeys or Commans in the TUI:

/ search

Enter preview

s summarize

l label

c classify

r rename suggestion

o obfuscate

f related files

g grep preview

q quit

--------------------------------
17. Suggested implementation stack (conceptual, not final)
--------------------------------

Ai, take in considerations the language I know and the language Ai works, important is that I can understand ALL the code, so I have to learn the languages but it would be easier if we document and program in a way I understand already.

Language
Not finalized in this conversation. A practical split could be:

fast prototype language for TUI/orchestration

shelling out to proven tools

SQLite for storage

Important design choice
Because the immediate goal is fast implementation and orchestration (specially MVP)>

do not start by building every low-level function yourself, lets mock and TDD first (also for TDD we can do iteratively NOT 10000 TDD cases to cover all possible variations)

build adapters to external tools

keep the engine interface stable

--------------------------------
18. Implementation style guidance
--------------------------------
Ai Remember our approach:
- mock engine first
- TDD
- Document all the functions and interfaces (even before start thinking/writing the implementation code) 
- TUI first enough to visualize workflows
- adapter pattern for tools
- root .arkti/ store
- SQLite root index
- logs and cache from day one
- safe suggestion model

Do not do this:
- hardwire all functionality into one giant script
- use AGPL code as copy-paste source
- overbuild per-folder metadata editing
- expand file types too early
- treat ARKTi as enterprise platform in v1


--------------------------------
19. Technical decisions (Drafts almost in final thinking)
--------------------------------
Chosen / preferred Decisions:
 
  Index storage: hybrid with SQLite root + optional manifests later
  First parser: PDF text only
  Metadata engine: evaluate sidecar-tagger first
  Local-only: yes
  TUI: yes
  Mock engine: yes, before full real integration
  Search accuracy over file understanding: yes, as priority order

Draft Decisions (we need to discuss with the Ai):

  final language/runtime
  final PDF pars
  final exact search adapter integration details
  semantic search timing
  manifest generation timing

--------------------------------
20. Open questions or still not clear or in Draft
--------------------------------

These are few elements known but deferred because we havent research good enough

What is a “good file” in full detail?
What is a “useful summary” exactly?
What labels taxonomy should exist?
What is the minimum useful search ranking strategy?
Which file types after PDF enter phase 2 first?
How much sidecar-tagger should be wrapped vs reused directly?
Which parts should be mocks vs real in first working TUI demo?

--------------------------------
21. Recommended implementation order
--------------------------------
Step 1
  Define the engine API and YAML & JSON (DTOs, structure, etc).
  Define the Data Model and Classes/Structures 

Step 2
  Implement a mock engine that returns realistic fake responses.

Step 3
  a) Build the ARKti engine (and mock it)
  b) Build the TUI against that mock engine:
    search page
    result details
    preview
    summary action page

Step 4
  Create the root .arkti/ project structure and SQLite schema.

Step 5
  Integrate real PDF discovery + text extraction.

Step 6
  Integrate real search backend (likely exact + label + extracted text).

Step 7
  Evaluate sidecar-tagger integration as metadata generation layer.

Step 8
  Add file assessment model (relevance + health).

Step 9
  Add logs, backup stubs, rollback stubs.

Step 10
  Add controlled action adapters (summarize, rename suggest, obfuscate).

--------------------------------
22. Summary for AI agents
--------------------------------

Ai (you and all your agents) treat ARKTi as a local-first project-folder intelligence TUI
for architect productivity with search-first design using a mock-first engine strategy
backed by SQLite root state + optional manifest snapshots integrating existing tools where useful
avoiding license-unfriendly code reuse starting with PDF-only MVP
and keeping all actions safe, suggestive, and user-approved 

The critical success criterion is not fancy AI or horrible spaguetti tool or rewrite all the functions from scratch.

The critical success criterion is:
  make it easy to find and understand the right project file fast using all the tools the Ai knows, that are easy to have/install and that can teach commands to the architect (facilitating this process by using a TUI)

--------------------------------
23. Short implementation mantras 
-------------------------------

## 1. One-line definition

ARKTi is a **local-first TUI/CLI orchestrator** for project folders that helps an architect **find, understand, classify, summarize, assess, and safely act on project files** without requiring the architect to know or remember every shell command or underlying tool (PDFTools, Exiftool, SideCar-tagger,etc)

---

## 2. Core product idea

ARKTi is **not** another generic file indexer.  
ARKTi is a **project-folder intelligence workbench**.

It should:
- scan an existing project root and subfolders
- build an index and metadata layer
- extract meaningful information from supported files
- allow very fast search and preview
- generate useful labels, summaries, and rename suggestions
- assess file quality/relevance/health
- optionally execute safe user-approved actions
- stay **local-only** for the next years
- be especially useful for **Software Architects** working on messy delivery/implementation projects

Primary value:
- **search accuracy first**
- **file understanding second**
- **controlled suggestions third**
- **on-demand actions fourth**
- **local-only safety always**

---

## 3. Primary user and scope

### Primary user
- One architect / technical consultant / senior engineer
- Initial user: the owner/creator only
- Internal developer-phase tool, not production enterprise rollout

### Usage scope
- Local project folders only
- No SharePoint/email/cloud integration in MVP (yet)
- No machine-wide indexing
- No full-disk indexing
- Typical project size expected:
  - often well below 500 MB
  - many nested folders
  - mixed quality file naming and structure

---

## 4. Main problems ARKTi solves

### Problem A — Search pain
The architect knows a file or concept exists somewhere in the project, but:
- filename is poor
- folder structure is messy
- document is old
- document is not clearly categorized
- the user must open many files manually

### Problem B — File understanding pain
Even when a file is found, it is hard to know:
- what it contains
- whether it is relevant
- whether it is outdated
- whether it is duplicate/auxiliary/temporary/broken
- how it relates to other files

### Problem C — Folder entropy
Project folders collect:
- contracts
- diagrams
- PDFs
- screenshots
- notes
- exports
- drafts
- configurations
- documents from different team members

Result:
- hidden knowledge
- duplicated work
- slower delivery
- strong dependency on people memory

### Problem D — Safe actions
The user wants help to:
- summarize
- classify
- label
- suggest better names
- mark low-value files
- obfuscate sensitive data
- optionally reorganize/rename later

But the tool must **not mutate destructively by default**.

---

## 5. Product principles

1. **Local-first**
   - no data leaves the machine by default
   - suitable for GDPR-sensitive environments

2. **Suggest first, apply later**
   - ARKTi does not enforce changes automatically
   - it proposes, previews, and then executes only on user approval

3. **Search before automation**
   - instant search + preview is more important than fancy actions

4. **Understand before rename**
   - file understanding must come before reorganization

5. **Composable architecture**
   - orchestrate existing tools when possible
   - avoid reinventing proven low-level utilities

6. **Mock-first implementation**
   - build TUI and engine contract before fully integrating real backends
   - mock everything to have clarity on the code implementation later on
   - TDD approach
   - build or get in internet real data to play with

---

## 6. Product positioning

ARKTi should be described as:

> A local-first terminal workbench that turns messy project folders into searchable, understandable, actionable project knowledge.

Avoid describing it as:
- a huge platform
- a replacement for OS search
- a general enterprise knowledge system
- a simple Google Desktop clone
- a simple Everything clone

---

## 7. Product name

**ARKTi — Architecture Knowledge Terminal Interface**

Command name:
```bash
arkti
Possible UX wording:

arkti scan .

arkti search "provider costs"

arkti summarize contracts/

arkti inspect file.pdf

arkti preview file.pdf


13. Engine contract (important)
Create a stable engine interface first.

Suggested core engine methods

scan(root_path)
refresh(root_path)
get_index_status(root_path)

search(query, filters)
get_file_details(path)
get_preview(path)

summarize(path_or_folder)
suggest_labels(path)
suggest_rename(path)

assess_file(path)
find_duplicates(scope)

obfuscate(path, policy)

list_operations()
rollback(operation_id)
backup(scope)
Why
This allows:

mock engine now

real integrations later

UI progress without backend dependency

testing independent from tool availability

14. Mock engine requirement
This is a deliberate strategy and should be implemented early.

Why a mock engine
TUI can be built immediately

no need to wait for sidecar-tagger integration

no need to wait for PDF parser decisions

behavior and UX can be tested first

Mock engine should fake:
search results

summaries

labels

rename suggestions

relevance/health assessments

progress bars

duplicate results

preview snippets

Mock engine output should be realistic
Not random nonsense. It should resemble likely real project data.

15. TUI design goals
General
ARKTi is a TUI, not just a raw CLI.
The TUI should feel like:

modern shell tooling

Oh My Zsh / powerlevel10k inspired

readable

colorful by meaning

keyboard-driven

visually structured

Visual principles
search bar / command input should be large and prominent

multiple panes

clear shortcuts bar

syntax-highlighted previews when possible

color indicates meaning, not decoration

Target layout
top bar: project path, mode, counts, warnings

large search/command input

left pane: results / tree

center pane: metadata / summary

right pane: preview / related / actions

bottom bar: shortcuts

UX inspiration sources
fzf (selection/fuzzy interaction)

broot (tree + search navigation)

bat (preview/highlight)

lazygit (panel-based TUI clarity)

k9s (watch/status mode ideas)