# Rich Text Composer — User Manual (English)

This manual is for users who have a **built, ready-to-use plugin** and work in the Unreal Editor (Blueprints, levels, UI, etc.).

**Engine compatibility:** Use the **Fab / marketplace listing** and **`EngineVersion`** in **`RichTextComposer.uplugin`** in your package (when set).

**中文:** [`User-Manual.zh.md`](./User-Manual.zh.md).

---

## Contents

- [1. Installation and enabling](#1-installation-and-enabling)
- [2. Project settings (Edit → Project Settings → Editor → Rich Text Composer)](#2-project-settings-edit--project-settings--editor--rich-text-composer)
- [3. Quick start: Display and document data](#3-quick-start-display-and-document-data)
- [4. Ways to edit rich text](#4-ways-to-edit-rich-text)
- [5. Modal editor: supported and unsupported](#5-modal-editor-supported-and-unsupported)
- [6. Runtime widget (Rich Text Composer Display)](#6-runtime-widget-rich-text-composer-display)
- [7. Typewriter](#7-typewriter)
- [8. Function chips: concepts, candidates, runtime binding](#8-function-chips-concepts-candidates-runtime-binding)
- [9. Subsystem and click events](#9-subsystem-and-click-events)
- [10. Localization template (REF) and `FText`](#10-localization-template-ref-and-ftext)
- [11. FAQ](#11-faq)

---

## 1. Installation and enabling

### 1.1 From Epic / Fab / Launcher (typical)

1. In **Epic Games Launcher**, under the **Unreal Engine** tab, install the engine version you need.
2. Under **Library / Vault** (or your Fab purchase), **install the product to the engine** (exact button text depends on the launcher).
3. Open your project, go to **Edit → Plugins**, search for **Rich Text Composer** (or `Rich Text`), enable **Enabled**, and **Restart** the editor if prompted.

**Why this section still says “enable in Plugins”:** Products **installed to the engine** via the launcher/Fab are usually **enabled per project** the first time. That flow is separate from `EnabledByDefault` in `.uplugin` when you drop the plugin under the project’s `Plugins` folder. Engine-managed installs **do not** require copying the plugin into `YourProject/Plugins` unless you want a project-local copy (see below).

### 1.2 Manual copy into `Plugins` (source / ZIP / project-only)

Use this when you **modify plugin source**, need the plugin in **one project only**, or received a **`RichTextComposer`** folder (repo / zip).

1. Copy the entire **`RichTextComposer`** folder (including `RichTextComposer.uplugin`, `Source`, etc.) into **`YourProject/Plugins/`** (create `Plugins` if needed).
2. If the folder came from another machine, delete **`Binaries`** and **`Intermediate`** inside the plugin, then open the project on **this** machine and **compile** when prompted.
3. This plugin sets **`"EnabledByDefault": true`** in `RichTextComposer.uplugin`. After a successful compile, it is **usually enabled by default** and you **typically do not** need to enable it again under **Edit → Plugins**. **The Plugins window is the source of truth** if you or source control previously disabled it.

### 1.3 How to verify it works

- **Plugins** lists **Rich Text Composer** and it is enabled (for launcher installs, or after you checked manually).
- On assets with a **`Rich Text Composer Document`** property, **Edit** appears in Details; or after adding **Rich Text Composer Display** in UMG, **Document** and related properties are editable.

---

## 2. Project settings (Edit → Project Settings → Editor → Rich Text Composer)

All settings below live under **`Edit → Project Settings → Editor`**, on the page titled **`Rich Text Composer`**. Most take effect the next time you open the modal or refresh the preview.

### 2.1 Toolbar

| Setting | Purpose |
|---------|---------|
| **Quick Color Palette** | Quick colors on the modal toolbar; used with the color picker for **body / selection** coloring. |
| **Font Size Presets** | Integer list for the **font size** dropdown in the modal (e.g. 10, 12, 14). |
| **Font Families** | `FSoftObjectPath` entries to `UFont` assets for the **font family** dropdown. If empty, the plugin falls back to built-in default fonts. |

### 2.2 Modal

| Setting | Purpose |
|---------|---------|
| **Edit Surface Background Color** | Background color of the **editing surface** in the modal window. |

### 2.3 Preview

| Setting | Purpose |
|---------|---------|
| **Preview Wrap Text** | Whether the **inline preview** of `Rich Text Composer Document` in **Details** **wraps** text. |

### 2.4 Caret

| Setting | Purpose |
|---------|---------|
| **Caret Width Pixels** | Caret line width in the modal (pixels; clamped min/max). |
| **Caret Color** | Caret color in the modal. |

### 2.5 Function chips (scan lists)

| Setting | Purpose |
|---------|---------|
| **Global Blueprint Function Libraries For Chip Scan** | Project-wide list of **Blueprint function library** classes for **Library** chips; one source of `[Library]` candidates in the insert dialog. |
| **Global Instance Classes For Chip Scan** | Project-wide **Instance** candidates (**do not** put Blueprint function libraries here); one source of `[Instance]` candidates. |

Merge rules with per-document **Bind Blueprint Function Libraries** / **Bind Instance Classes** are in [Section 8.2](#82-which-classes-appear-in-the-insert-dialog).

---

## 3. Quick start: Display and document data

1. Open a **Widget Blueprint**, drag **`Rich Text Composer Display`** from **Palette** into **Hierarchy**.
2. Select the widget. In **Details**, edit **`Document`** directly (same data as a `Rich Text Composer Document` property elsewhere), or click **Edit** next to **`Document`** to open the **modal editor** (Sections 4–5).
3. At runtime you will often call **`Set Document`** (Blueprint node name is usually **Set Document**) to replace the whole document from variables or network data.
4. For **Instance** function chips, register live objects at runtime using the nodes in **Section 8.4**; otherwise those chips may not appear.

---

## 4. Ways to edit rich text

| Mode | Description |
|------|-------------|
| **A. Document property on any UObject / Blueprint** | Property type **`Rich Text Composer Document`**; **Edit** in Details opens the modal. On the same Details panel you can set **Bind Blueprint Function Libraries** / **Bind Instance Classes** (chip candidates for **this** document only). |
| **B. Rich Text Composer Display in UMG** | Edit **`Document`** in the designer **Details**, or **Edit** to open the same modal—good when copy lives with the widget. |
| **C. Runtime** | Call **`Set Document`** on the Display, or update a stored document variable then **Set Document**, for quests, chat, etc. |

All three edit the **same struct type** (`FRichTextComposerDocument` semantics) but **not necessarily the same instance**: e.g. a DataAsset field and a Display’s **`Document`** are **two separate copies** by default. To keep UI and asset in sync, **`Set Document`** at runtime, or keep a **single source of truth** in your game code and push to the Display.

---

## 5. Modal editor: supported and unsupported

**Supported in the current modal** (toolbar-related options come from [Section 2](#2-project-settings-edit--project-settings--editor--rich-text-composer)):

- Type and edit body text; **bold / italic**; **font size** from **Font Size Presets**; **font family** from **Font Families** (plus built-in fallbacks); **text color** (including **Quick Color Palette**).
- Insert and edit **hyperlinks**, **images**, and **function chips**.
- **Inline spacing** between text and inline chips for the current selection (toolbar control affecting paragraphs touched by the selection).

**Not exposed in the modal UI today:**

- **Paragraph alignment** (left/center/right/justify): the data model has alignment fields, but **this plugin’s modal does not provide alignment controls**. To set alignment you need another path that can write `FRichTextComposerDocument` (custom tool, C++, etc.); this manual does not cover that.

---

## 6. Runtime widget (Rich Text Composer Display)

### 6.1 Designer / Details properties

| Property | Description |
|----------|-------------|
| **Document** | Document shown in the widget; editable in the UMG designer or replaced at runtime with **`Set Document`**. |
| **Base Font Size** | Default pixel size when a span’s font size is **0** (≥ 1). |
| **Use Internal Scroll Box** | If true, the widget provides its own scroll box; if false, place it inside **your own** `ScrollBox` (or similar). |
| **Always Show Vertical Scrollbar** | When internal scroll is on and content scrolls, keep the vertical scrollbar **always visible**. |
| **Auto Scroll To End On Document Change** | After **`Set Document`**, scroll to the **bottom** on the next frame (chat/log style). Requires internal scroll. |
| **Auto Scroll Only If Near Bottom** | When used with the above: if true, auto-scroll only if the user was **already near the bottom**. |

There is also a **`Set Use Internal Scroll Box`** Blueprint node to toggle internal scrolling at runtime.

### 6.2 Blueprint-callable functions (names as shown in editor)

| Node / function | Purpose |
|-----------------|--------|
| **Set Use Internal Scroll Box** | Toggle internal scroll at runtime. |
| **Set Document** | Replace the displayed document. |
| **Get Document** | Get a copy of the current document. |
| **Set Base Font Size** | Change the default font size at runtime. |
| **Set Line Height Layout Reference** | During typewriter reveal, stabilize line heights using a **reference document**. |
| **Clear Line Height Layout Reference** | Clear that reference. |
| **Scroll To End** | Scroll to the end when internal scroll is enabled. |
| **Get Scroll Progress** | Returns scroll ratio **0–1** with **internal** scrolling; **0** when internal scroll is **off** or no valid scroll box yet. |
| **Get Rich Text Content Geometry** | `FGeometry` of the inner rich-text area (for aligning other UI). |
| **Get Document Tail Local** | Caret and last segment bounds at the document “tail” in **inner** space. |
| **Get Document Tail Absolute** | Same in **absolute** screen space. |
| **Bind Runtime Object to Chip Class (Widget)** | Register a **runtime object** for an **Instance** chip **bind class** on **this** widget only. |
| **Unbind Runtime Object for Chip Class (Widget)** | Remove registration for that bind class on this widget. |
| **Clear All Chip-Class Objects (Widget)** | Clear **all** Instance registrations on this widget. |
| **Find Runtime Object for Chip Class (Widget)** | Look up on this widget only (**not** the subsystem fallback). |

---

## 7. Typewriter

The typewriter **reveals** a document step-by-step into a **Rich Text Composer Display**. Two forms: a standalone **`URichTextComposerTypewriter`** object, and an **Actor component** that owns a typewriter and forwards the API.

### 7.1 `URichTextComposerTypewriter` (object)

Typical use: create an instance in Blueprints (**Construct Object from Class**), store it in a variable, tune **Interval**, **Target Display**, etc. in **Details**; or drive everything from the event graph with **Play From*** nodes.

**Properties (Blueprint / Details)**

| Property | Description |
|----------|-------------|
| **Interval** | Seconds between reveal steps. |
| **Chars Per Tick** | Characters (or equivalent units) per step (≥ 1). |
| **Reveal Hyperlink By Character** | If true, hyperlink visible text reveals **character by character**; if false, each link appears in **one** step. |
| **Typing Sound** | Sound asset for typing clicks. |
| **Typing Sound Volume Multiplier** | Volume multiplier. |
| **Play Typing Sound** | Whether to play typing sounds. |
| **Typing Sound Min Interval** | Minimum seconds between sounds; **0** allows a sound on every step. |
| **Target Display** | **Rich Text Composer Display** that receives the reveal; also supplies **World** (and similar) for chip resolution. |

**Delegates (Assignable)**

| Delegate | Description |
|----------|-------------|
| **On Reveal** | Fired each step; parameter is the **current partially revealed** `FRichTextComposerDocument`. |
| **On Finished** | Fired when the reveal **fully completes**. |

**Blueprint-callable / query functions**

| Function | Description |
|----------|-------------|
| **Play From Document** | Start from a **`Rich Text Composer Document`** using the configured **Target Display**. |
| **Play From String** | Build temporary content from **`FString`** and play. |
| **Play From Text** | Same from **`FText`**. |
| **Pause** | Pause stepping (timer paused). |
| **Resume** | Resume from pause. |
| **Skip To End** | Jump **instantly** to the full document; fires **On Finished**. |
| **Stop** | Stop the timer and reset internal state. If **Target Display** is set, clears line-height reference and sets that Display’s document to **empty** (UI cleared). **Does not** fire **On Finished** (unlike **Skip To End**). |
| **Set Interval** | Changes **Interval**; if **playing and not paused**, the timer is **rebuilt immediately** with the new interval. |
| **Is Playing** | Whether a reveal is in progress. |
| **Is Paused** | Whether paused. |
| **Get Total Units** | Internal total reveal units (for progress UI). |
| **Get Revealed Units** | Units revealed so far. |

### 7.2 `URichTextComposerTypewriterComponent` (Actor component)

In **Details**, configure **Interval** (label **Typing Interval (seconds)**), **Characters Per Step**, **Reveal Hyperlink By Character**, and the four sound fields. **`Target Display` is not meant to be edited as a default property**; use **`Set Target Display`**, or pass a widget into the **optional Display** pin on **`Play From Document` / `Play From String` / `Play From Text`** (same effect as **Set Target Display**).

**Delegates:** **On Reveal**, **On Finished** (same meaning as 7.1).

**Blueprint API**

| Function | Description |
|----------|-------------|
| **Set Target Display** | Choose the **Rich Text Composer Display** to drive. |
| **Get Target Display** | Returns the current target. |
| **Play From Document** | Document plus **optional** Display (if non-null, also sets the target for this play). |
| **Play From String** | Same, from string. |
| **Play From Text** | Same, from `FText`. |
| **Stop** | Same semantics as **Stop** in 7.1 (clears target document, line-height reference, etc.). |
| **Skip To End** | Same as 7.1: instant end + **On Finished**. |
| **Pause** / **Resume** | Pause / resume. |
| **Set Interval** | Writes the component’s **Interval**; if the internal typewriter exists, it is **synced** (if playing and not paused, timer rebuild matches 7.1). If no **Play*** has run yet, the internal object may not exist; the new interval applies on the **first Play***. |
| **Is Playing** / **Is Paused** | State queries. |
| **Get Total Units** / **Get Revealed Units** | Progress queries. |
| **Get Typewriter** | Returns the internal **`URichTextComposerTypewriter`** (for advanced debugging). |

---

## 8. Function chips: concepts, candidates, runtime binding

### 8.1 Library vs Instance

| Kind | When to use | Recommendation |
|------|-------------|----------------|
| **Library** | Logic does **not** need a specific level object’s **member state**; implement on a **`UBlueprintFunctionLibrary` subclass** (**Blueprint** or **C++** library), e.g. table-driven text, formatting. | Prefer Library when possible—simpler preview and deployment. |
| **Instance** | You need **instance state** (HUD context, player component, etc.). The document stores **class + function name** only; you **must bind** a live object at runtime (Section 8.4). | Use only when Library is insufficient. |

### 8.2 Which classes appear in the insert dialog

Four sources are **merged** into the class list; each **`UClass` appears only once** (first occurrence wins). Order:

1. Project **Global Blueprint Function Libraries For Chip Scan**  
2. Project **Global Instance Classes For Chip Scan**  
3. Document **Bind Blueprint Function Libraries**  
4. Document **Bind Instance Classes**  

Pick a **class** (with **`[Library]`** / **`[Instance]`** tag), then a **function**.

### 8.3 UFUNCTION requirements

Functions must be **`BlueprintCallable` or `BlueprintPure`** and satisfy the plugin’s **return type** and **parameter** rules. Invalid signatures may be missing from the list or fail at runtime. Use the insert dialog as the primary sanity check.

### 8.4 Runtime binding: full node list and guidance

**Resolution order (Instance chips):** look up on **this Display** first; if missing, use **`Rich Text Composer Subsystem`** **Game Fallback**. **Library** chips **do not** use these bindings.

#### On `Rich Text Composer Display` (best for “this UI only”)

| Blueprint display name | Purpose |
|------------------------|---------|
| **Bind Runtime Object to Chip Class (Widget)** | Register a **runtime object** for a chip **bind class** on this widget. In Blueprints, **None** on **Runtime object** (unconnected pin) **removes** registration for that class on this widget. |
| **Unbind Runtime Object for Chip Class (Widget)** | Remove registration for that bind class on this widget. |
| **Clear All Chip-Class Objects (Widget)** | Clear **all** Instance registrations on this widget. |
| **Find Runtime Object for Chip Class (Widget)** | Query this widget only (**not** the subsystem fallback). |

#### On `Rich Text Composer Subsystem` (global / singleton-style context)

| Blueprint display name | Purpose |
|------------------------|---------|
| **Bind Runtime Object to Chip Class (Game Fallback)** | **Game Instance** fallback when the Display has **no** registration for that bind class. **None** removes subsystem registration for that class. |
| **Unbind Runtime Object for Chip Class (Game Fallback)** | Remove that class from the subsystem table. |
| **Clear All Chip-Class Objects (Game Fallback)** | Clear **all** Instance registrations on the subsystem (**does not** clear Display registrations). |
| **Find Runtime Object for Chip Class (Game Fallback)** | Query the subsystem table only. |

#### Blueprint library `Rich Text Composer Binding Blueprint Library`

| Blueprint display name | Purpose |
|------------------------|---------|
| **Clear All Chip-Class Objects (Game Fallback Only)** | Clears the **subsystem** only. |
| **Clear All Chip-Class Objects (Widget Only)** | Clears the given **Display** only. |
| **Clear All Chip-Class Objects (Game Fallback + Widget)** | Clears subsystem first, then the Display if provided; **if Display is null, only the subsystem is cleared**. |

**Guidance:** Widget-specific context → prefer **Display (Widget)**; shared game-wide context → **Game Fallback**. You **may** register **both** for the same bind class: **Display always wins**, subsystem fills the gap (“global default + per-widget override”). If you do not need layering, pick **one** place for clarity.

### 8.5 Editor preview vs play

Details/modal previews lack a full game world and your runtime bindings; **Instance** chips often show placeholders or are omitted. **Library** chips usually preview fine.

---

## 9. Subsystem and click events

**`Rich Text Composer Subsystem`** is a **Game Instance Subsystem**. In Blueprints, search for **Subsystem** / **Game Instance** and use **`Get Game Instance Subsystem`** (some engine versions label it **Get Subsystem from Game Instance**, etc.) with class **`Rich Text Composer Subsystem`**.

**Multicast delegates you can bind**

| Delegate | Description |
|----------|-------------|
| **On Hyperlink Clicked** | Hyperlink clicked; includes display text, target string, `EHyperlinkTargetType`, etc. |
| **On Image Clicked** | Related to clickable image chips; includes **Gameplay Tag**, texture path, etc. |

Subsystem binding APIs for Instance chips are in [Section 8.4](#84-runtime-binding-full-node-list-and-guidance).

---

## 10. Localization template (REF) and `FText`

This section is the **user-facing** description of localization for `Rich Text Composer Document`. Use it with Unreal’s **Gather Text** / PO workflow on the **`LocalizedMarkupSource`** property when **`Use Localized Markup`** is enabled.

### 10.1 What the two fields do

| Field | Purpose |
|-------|---------|
| **Use Localized Markup** | When **true**, **Rich Text Composer Display** and the **Typewriter** take the current-culture string from **`LocalizedMarkupSource`** and merge it into a **copy** of the document: **plain text** for included segments comes from that `FText`; **layout, styles, chips, images, hyperlinks, and function chips** still come from **`Paragraphs`**. When **false**, only **`Paragraphs`** are used (no merge). |
| **Localized Markup Source** | A single **`FText`** string built when you **OK** the modal editor. It is **sparse**: one **`{n:i}…{/n:i}`** segment per **plain Text** run that counts toward localization (see §10.2), including **empty** Text runs used for spacing at style/chip boundaries. Suitable for **Gather** as one entry per document property (per culture after import). |

**Gather (Localization Dashboard):** Unreal only collects `FText` values that have a localization **identity** (namespace + key). The plugin assigns one when **Use Localized Markup** is on (stable per document field via an internal GUID). Your target’s **Gather from Packages → Include Path Wildcards** must cover the folder that contains the Blueprint or widget asset (for example `/Game/...`); otherwise nothing from that asset is gathered. After upgrading the plugin or enabling markup for the first time, **OK** the modal, **save** the asset, then run **Gather** again.

### 10.2 What appears in the string (sparse template)

The plugin walks your document in order (all paragraphs and runs). It **adds a segment** for each run that is:

- **`Text`** type, and  
- **Not** a **structural empty** anchor (the invisible runs the editor inserts for caret placement around chips; those never carry visible translated text).

This **includes** runs whose **`TextContent` is empty** but are **not** structural anchors—those slots often represent **gaps** between colors or before/after inline chips. Translators may leave the body empty or insert **spaces** in a given culture so spacing matches that language’s typography.

**Not** written into this string (they stay only in **`Paragraphs`**): **line breaks**, **images**, **hyperlinks**, **function chips**, and **structural empty** text anchors. **Dynamic text** returned by chips at runtime is localized **elsewhere** (your game data / other `FText` keys), not inside this field.

### 10.3 Wire format (for translators)

Segments are concatenated in order, each shaped like:

**`{n:0}`** *body* **`{/n:0}`** **`{n:1}`** *body* **`{/n:1}`** …

- **`i`** is the slot index: **0, 1, 2, …** counting **only** the `Text` runs that participate in the template (not every run in the document). A segment may have an **empty** body between **`{n:i}`** and **`{/n:i}`** when the authored document used an empty spacing run; you may fill it with spaces for a locale if needed.
- **Body** is the translatable text. To put a literal **`{`** or **`}`** inside the body, write **`{{`** or **`}}`** (same idea as escaping braces in other formats).
- There are **no** extra newline characters between paragraphs inside this string; paragraph breaks remain in the structured document.

**Translators:** edit **only** the text **between** **`{n:i}`** and **`{/n:i}`**. Do **not** change the tags or the numbers **`i`**, and do not reorder segments, or the merge will fail and the UI will **fall back** to the authored **`Paragraphs`** text until the string is fixed or regenerated.

### 10.4 Editor: Details and preview

In **Details**, enable or disable **Use Localized Markup**, and expand **Localized template (REF v1, read-only)** to inspect the generated string. The **inline preview** uses the same merge path as runtime when the option is on, so changing the editor **Preview Language** (when available) should match in-game culture for this field.

### 10.5 If something goes wrong

If the stored string **does not parse** (wrong indices, extra characters, missing closing tags), the plugin **ignores** it for layout and uses **`Paragraphs`** until you open the **modal**, make a small edit if needed, and **OK** to **regenerate** a valid template.

---

## 11. FAQ

**Q: Compile fails after copying into the project, or the editor asks to compile the plugin?**  
A: Delete **`Binaries`** and **`Intermediate`** inside the plugin folder and reopen the project. Install **Visual Studio** with the C++ workload required by your engine. The engine **major** line must match the plugin listing (Fab / product page; `.uplugin` may omit **`EngineVersion`**).

**Q: I installed from the marketplace but cannot find the plugin.**  
A: Ensure the product is installed for the **same engine version** the project uses; search **`Rich Text`** or **`Composer`** in the Plugins window.

**Q: Instance chips do not show in game.**  
A: Ensure you called **Bind Runtime Object to Chip Class (Widget)** or **(Game Fallback)** for the **bind class stored on the chip**; the object must be an **instance of** that class (or subclass).

**Q: Why is there no paragraph alignment in the modal?**  
A: The current modal **does not expose alignment UI**; see [Section 5](#5-modal-editor-supported-and-unsupported).

**Q: Where do font lists come from?**  
A: Toolbar fonts come from **Project Settings → Rich Text Composer → Font Families** (plus built-in fallbacks); sizes from **Font Size Presets**. See [Section 2](#2-project-settings-edit--project-settings--editor--rich-text-composer).
