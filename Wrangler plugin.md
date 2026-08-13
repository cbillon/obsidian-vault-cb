---
tags:
  - plugin
---
# Obsidian Tag Wrangler Plugin

> NEW in 0.6.0
> 
> - Open or create a tag page by alt/opt clicking a tag in any note (or the tags view)
> - Use the Tag Wrangler context menu for tags in the body of a note (editor or preview mode)
> - Drag-and-drop tags to rename/reorganize them
> - Drag tags from the tags view or a note preview to an editor pane to insert them as text.

This plugin adds a context menu for tags in theÂ [Obsidian.md](https://obsidian.md/)Â tags view, with the following actions available:

![Image of tag wrangler's context menu](https://raw.githubusercontent.com/pjeby/tag-wrangler/master/contextmenu.png)

- Open or create aÂ [Tag Page](app://obsidian.md/index.html#tag-pages)Â (**NEW in 0.5.0**)
- [Rename the tag](app://obsidian.md/index.html#renaming-tags)Â (and all its subtags)
- Start a new search for the tag (similar to a plain click)
- Add the tag as a requirement (`tag:#whatever`) to the current search
- Add an exclusion for the tag (`-tag:#whatever`) to the current search
- Open a random note with that tag (if you have theÂ [Smart Random Note](https://github.com/erichalldev/obsidian-smart-random-note/)Â plugin installed and enabled)
- Collapse all tags at the same level in the tags view
- Expand all tags at the same level in the tags view

Depending on the current state of the search and tag views, some actions may not be available. (e.g. expand and collapse are only available when the tags view is showing tags in a hierarchy.)

**Please note**: renaming a tag is potentially an irreversible operation: you may wish to back up your data before beginning a rename. See the section onÂ [Renaming tags](app://obsidian.md/index.html#renaming-tags)Â below for more information.

## Installation

Search for "tag wrangler" in Obsidian's Community Plugins interface, orÂ [click here](https://obsidian-plugins.peak-dev.org/show/tag-wrangler)Â to open it in your most-recently-used vault. (Then select "Install" and "Enable".)

Also, make sure that you have enabled the "Tags View" plugin, in the "Core Plugins" section of your vault's configuration. This plugin adds a context menu to the pane provided by that plugin, and does not have any UI of its own.

## Tag Pages

People often debate the merits of using tags vs. page links to organize your notes. With tag pages, you can combine the best of both worlds: the visibility and fluid entry of tags, plus the centralized content and outbound linking of a page.

To create a tag page, just right click any tag in the tags view, then select "Create Tag Page". A new note will be created with an alias of the selected tag. You can rename the note or move it anywhere you like in the vault, as long as it retains the alias linking it to the tag. (Renaming a tag associated with a tag page (see "Renaming Tags", below) will automatically update the alias.)

To open or create a tag page, you can Alt-click (Option-click on Mac) any tag in the tags view or any note, whether in editing or reading view. Ctrl/Cmd-click or middle click plus Alt/Option will open the tag page in a new pane. (Note: if no tag page exists, you'll be prompted for whether you want to create it. If you cancel, the normal click behavior of globally searching for the tag will apply.)

Or, you can enter the tag's name in the Obsidian "quick switcher" (default hotkey: Ctrl/Cmd-O) to open the page from the keyboard. You can also hover-preview any tag in the tags view or any markdown views to pop up a preview of the tag page.

(If you're not familiar with hover-previewing, the basic idea is that by holding the Ctrl/Cmd key while moving the mouse pointer over an item in Obsidian, a popup will often appear with a small version of the relevant page. You can also go into the settings for the built-in "Page Preview" plugin and selectively disable the need for using the Ctrl/Cmd key, if you prefer to just hover without it. Tag Wrangler respects your existing settings for hovering links in Editor and Preview views, and adds an extra setting for "Tags View" that controls whether it will require the Ctrl/Cmd key when hovering tags in the tags view.)

Tag Wrangler does not (yet) support automatic conversion of tag references to page links or vice versa, though it may in a future version. In the meantime, however, you can use Obsidian's backlinks to find and change such references from the tag page. Specifically, viewing a tag page's "unlinked mentions" will show you all the locations where the tag was used in the body of a note, and "link" buttons you can use to convert them to page links. (Converting page links to tags requires hand editing.)

(Note: to see "unlinked mentions", you have to click the part of the backlinks pane that says "unlinked mentions" to perform the search, as it's not done by default. Also, the Obsidian unlinked mentions feature does not support the tagsÂ _property_, only tags used in the note body.)

### Manually Creating and Managing Tag Pages

You do not have to use the "Create Tag Page" menu command to create a tag page: any page (even Kanban boards or Excalidraw drawings!) can be a tag page as long as it has a tag asÂ **a valid Obsidian alias**. You can validate whether a particular page has a valid alias in one of two ways:

- Open the Obsidian Quick Switcher (Ctrl/Cmd-O by default) and typeÂ `#`Â followed by the tag name: the page should show up with the tag as an alias
- Switch the note to preview, and see if the "Aliases" in the metadata box at the top of the note has a shape containing the tag (with an arrow and aÂ `#`Â at the front)

If it does not show up in either of the above places, it's likely that your note's metadata is syntactically incorrect in some way. Obsidian recognizes the first field namedÂ `Alias`Â orÂ `Aliases`Â (case-insensitive) and expects it to be either a YAML list of strings or a single string of aliases separated by commas. To be recognized as a tag alias, the string or stringsÂ _must_Â be quoted. Here are some valid examples of an alias or aliases field that contains a tag, each of which would cause the note to be considered the tag page forÂ `#some/tag`:

```yaml
---
Alias: "#some/tag"
---
Aliases: [ "#some/tag", "another alias" ]
---
alias:
  - some alias
  - "#some/tag"
  - another alias
---
aliases: "some alias, #some/tag, another alias"
---
```

Notice that either each item must be quoted and be in a valid YAML list (`[ ]`Â or lines withÂ `-`), or else theÂ _entire_Â alias collection should be quoted and comma-separated. A tag alias must also not contain any whitespace or other text.

When a tag is renamed, Tag Wrangler will automatically update the alias. You can also manually edit or remove the aliases to disconnect a tag page, add additional tags (in case you want more than one tag sharing the same page), or change what tag the page is for.

Finally, note that nothing prevents you from adding the same alias to more than one note, but in such a case Tag Wrangler will select the tag page at random from the available options. To fix this, use the quick switcher to search for notes with that alias (i.e., by typingÂ `#`Â and the tag name), then remove the alias(es) from the note(s) you don't want to use as tag pages.

## Renaming Tags

Just like renaming notes, renaming tags involves making substitutions in all the files that reference them. In order to ensure only actual tags are renamed, Tag Wrangler uses Obsidian's own parse data to identify the tag locations. That way, if you have a markdown link likeÂ `[foo](#bar)`, it won't consider theÂ `#bar`Â to be an instance of aÂ `#bar`Â tag.

Because tags exist only in the files that contain them, certain renaming operations areÂ _not reversible_. For example, if you renameÂ `#foo`Â toÂ `#bar`, and you already have aÂ `#bar`Â tag, then afterwards there will be no way to tell which files originally hadÂ `#foo`Â and which hadÂ `#bar`Â any more, without consulting a backup or revision control of some kind.

For this reason, Tag Wrangler checks ahead of time if you are renaming tags in a way that will merge any tags with existing tags, and ask for an additional confirmation, warning about the lack of undo. (Not thatÂ _any_Â renames are really undoable, it's just that if no merging takes place, you can generally rename the tag back to its old name!)

If you are using some type of background sync (e.g. Dropbox, GDrive, Resilio, etc.), and it causes any files to be changedÂ _while_Â Tag Wrangler is doing a rename, Tag Wrangler may skip the changed file(s), resulting in a partial rename. Generally, repeating the same rename option should work to finish the process, but in case of merges you may have to be more careful. (It's probably best you make sure any sync operations are completed before beginning any rename operations.)

If many files need to be changed, or if renaming proceeds slowly, a progress dialog will be displayed, giving you the option to abort the renaming process. This will not undo changes made prior to that point, only stop further changes from occurring.

### Tags With Child Tags

Obsidian allows hierarchical tags of the formÂ `#x/y/z`. When you rename a parent tag (likeÂ `#x/y`), all of its child tags will be renamed as well.

So for example, if you renameÂ `#x/y`Â toÂ `#a/b`, then a tag that was previously namedÂ `#x/y/z`Â will be renamed toÂ `#a/b/z`. (There is no way to rename only the parent tag, except by individually renaming all its child tags to move them under another parent.)

If you want to refactor your tag hierarchy, note that you can rename a tag and its children to have either more or fewer path parts than it did before. That is, you can renameÂ `#x/y`Â to justÂ `#x`, and then yourÂ `#x/y/z`Â tag will becomeÂ `#x/z`. Or conversely, you can renameÂ `#x/y`Â toÂ `#letters/x/y`, which will moveÂ `#x/y/z`Â toÂ `#letters/x/y/z`.

Many possibilities are available for refactoring your tags. Just be sure to make a backup before you start, keep track of what you're changing, and check on the results before you sync or commit your changes.

### Metadata / Front Matter

Obsidian allows tags to be specified as part of a note's metadata via YAML front matter. Tag Wrangler will attempt to rename these as well as those found in a note's body.

In most cases, this will not cause any issues. However, you are using advanced YAML features to specify your tags (YAML aliases or block scalars), there are two points you should be aware of.

First, if you are using YAML block scalars (`<`Â orÂ `|`) to specify your tags, a renamed tagÂ [may affect the indentation, spacing, or wrapping](https://github.com/eemeli/yaml/issues/349)Â of the tags field. Second, if you are using YAML aliases (`*`) in your tag list, renaming an aliased tag willÂ [expand the alias in place, rather than changing the anchor that defined it](https://github.com/pjeby/tag-wrangler/issues/13#issuecomment-826264213).

The indentation issue has been reported to the upstream library, and hopefully there will be some sort of solution in the future. For aliases, the current behavior is a deliberate choice to avoid changing non-tag values in your metadata. So if you are using either of these more-advanced YAML features, you should probably experiment with some fake tags in a scratch note before doing any vault-wide renames.

### Case Insensitivity

Tag Wrangler uses the same case-insensitive comparison as Obsidian when matching tags to change, and checking for clashes. Please note, however, that because Obsidian uses theÂ _first_Â occurrence of a tag to determine how it is displayed in the tags view, renaming tags without consistent upper/lowercase usage may result inÂ _apparent_Â changes to the names of "other" tags in the tags view.

Let's say you have a tag namedÂ `#foo/bar`Â and you renameÂ `#foo`Â toÂ `#Bar/baz`. But in the meantime, you alreadyÂ _had_Â a tag calledÂ `#bar/bell`. ThisÂ _might_Â cause you to now see that tag displayed in the tags view asÂ `#Bar/bell`, even though Tag Wrangler did not actually replace any existingÂ `#bar/bell`Â tags in your text! (As you will see if you search for them.)

Rather, this kind of thing will happen if theÂ `#Bar/baz`Â tag is the first tag beginning with some variant ofÂ `bar`Â that Obsidian encounters when generating the tags view. Obsidian just uses the first-encountered string of a particular case as the "display name" for the tag, and then counts all subsequent occurrences as the same tag.

This is just how Obsidian tags work, and not something that Tag Wrangler can work around. But you can easily fix the problem by renaming anything that's in the "wrong" case to the "right" case. It just means that (as is already the case in Obsidian) you can't have more than one casing of the same tag name displayed in the tags view, and that now you can easily rename tags to a consistent casing, if desired.

### Canvas Support

Please note that tag renaming is not supported for tags in Obsidian Canvas files yet, as Obsidian itself doesn't fully support such tags yet either. (That is, tags in canvas text doÂ _not_Â appear in the tags view counts or in Obsidian's internal indexes, so from Tag Wrangler's perspective they aren't findable and don't exist.) If some future version of Obsidian addresses this, this limitationÂ _may_Â be removable then, depending on how the issue is addressed.

## Developer Notes

Tag Wrangler triggers the following events on theÂ `app.workspace`Â that may be useful for integration with other plugins:

### `tag-wrangler:contextmenu`

This event allows other plugins to add menu items to the Tag Wrangler context menus. You can register a callback like this:

```typescript
type menuInfo = {
  query?: string           // the current global search query
  isHierarchy: boolean     // true if the tag is a child tag in the tags view (sorted by hierarchy)
  tagPage: TFile|undefined // the tag page for the note, if it exists
};

this.registerEvent(
  app.workspace.on("tag-wrangler:contextmenu", (menu: Menu, tagName: string, info: menuInfo) => {
    // add items to menu here
  })
);
```

### `tag-page:will-create`

This event allows other plugins to take over creation of tag pages. You can register a callback like so:

```typescript
type tagPageEvent = {
  tag: string
  file?: TFile | Promise<TFile>
}

this.registerEvent(app.workspace.on("tag-page:will-create", (evt: tagPageEvent) => {
  if (!evt.file) {
    // create the file here, then save it in the event
    evt.file = someAsynFunctionReturningaTFilePromise();
  }
}));
```

You can set the file to the result of calling an async function (as shown), but the event handler itself mustÂ **not**Â be async nor should it await anything. If it's async, you will most likely end up with multiple files created because Tag Wrangler and any other event handlers for the event will think no other plugin has created one.

Note that ifÂ `evt.file`Â is anything butÂ `undefined`, your callbackÂ _must not do anything_, as the file has already been created or is in the process of being created. If you want to modify an already-created tag page file, use theÂ `tag-page:did-create`Â event instead. (See below.)

For users of Quickadd and other plugins that allow user-defined Javascript, note that theÂ `this.registerEvent()`Â call may need to be replaced with something likeÂ `app.plugins.plugins['quickadd'].registerEvent()`. You should also only run this codeÂ _once_, when the plugin is initialized, and not as part of a command or template.

### `tag-page:did-create`

This event allows other plugins to modify or rename a newly-created tag page. It has the same callback signature asÂ `tag-page:will-create`, except theÂ `file`Â field will always contain a TFile. (The one created by Tag Wrangler or by a callback toÂ `tag-page:will-create`.) You should use theÂ `app.vault.process()`Â method to do any changes, to prevent accidental file overwrites and data loss. (It should also be safe toÂ `app.vault.rename()`Â it to change its name or location.)

Unlike event handlers forÂ `will-create`, the handler forÂ `did-create`Â can be asynchronous.