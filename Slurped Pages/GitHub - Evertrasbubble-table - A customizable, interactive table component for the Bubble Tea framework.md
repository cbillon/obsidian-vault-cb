---
link: https://github.com/Evertras/bubble-table
byline: Evertras
site: GitHub
excerpt: A customizable, interactive table component for the Bubble Tea framework - Evertras/bubble-table
twitter: https://twitter.com/@github
slurped: 2025-11-26T11:27
title: "GitHub - Evertras/bubble-table: A customizable, interactive table component for the Bubble Tea framework"
tags:
  - go
  - bubble_tea
---

## Bubble-table

[](https://github.com/Evertras/bubble-table#bubble-table)

[![Latest Release](https://camo.githubusercontent.com/8f16f22763ff360a4ac117ba55149e0c28c693abe03ddd1b8bb3592c523f37e0/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f72656c656173652f45766572747261732f627562626c652d7461626c652e737667)](https://github.com/Evertras/bubble-table/releases) [![GoDoc](https://camo.githubusercontent.com/1f1b851f90e4f5260e8a530392d5e506c447009755b0a1fa9b4c67001d104223/68747470733a2f2f676f646f632e6f72672f6769746875622e636f6d2f676f6c616e672f6764646f3f7374617475732e737667)](https://pkg.go.dev/github.com/evertras/bubble-table/table?tab=doc) [![Coverage Status](https://camo.githubusercontent.com/1f4834d8bd70f8235b32e331e9052f20906dfcd4410db3e85ea3342756ac6499/68747470733a2f2f636f766572616c6c732e696f2f7265706f732f6769746875622f45766572747261732f627562626c652d7461626c652f62616467652e7376673f6272616e63683d6d61696e26686173683d616263)](https://coveralls.io/github/Evertras/bubble-table?branch=main) [![Go Report Card](https://camo.githubusercontent.com/0e6889ec6b225b91d67c9c3e53bae25013878e90e96d35e0b387f3c8ba68499d/68747470733a2f2f676f7265706f7274636172642e636f6d2f62616467652f6769746875622e636f6d2f65766572747261732f627562626c652d7461626c65)](https://goreportcard.com/report/github.com/evertras/bubble-table)

A customizable, interactive table component for the [Bubble Tea framework](https://github.com/charmbracelet/bubbletea).

[![Styled table](https://user-images.githubusercontent.com/5923958/188168029-0de392c8-dbb0-47da-93a0-d2a6e3d46838.png)](https://user-images.githubusercontent.com/5923958/188168029-0de392c8-dbb0-47da-93a0-d2a6e3d46838.png)

[View above sample source code](https://github.com/Evertras/bubble-table/blob/main/examples/pokemon)

## Contributing

[](https://github.com/Evertras/bubble-table#contributing)

Contributions welcome, please [check the contributions doc](https://github.com/Evertras/bubble-table/blob/main/CONTRIBUTING.md) for a few helpful tips!

## Features

[](https://github.com/Evertras/bubble-table#features)

For a code reference of most available features, please see the [full feature example](https://github.com/Evertras/bubble-table/blob/main/examples/features). If you want to get started with a simple default table, [check the simplest example](https://github.com/Evertras/bubble-table/blob/main/examples/simplest).

Displays a table with a header, rows, footer, and borders. The header can be hidden, and the footer can be set to automatically show page information, use custom text, or be hidden by default.

Columns can be fixed-width [or flexible width](https://github.com/Evertras/bubble-table/blob/main/examples/flex). A maximum width can be specified which enables [horizontal scrolling](https://github.com/Evertras/bubble-table/blob/main/examples/scrolling), and left-most columns can be frozen for easier reference.

Border shape is customizable with a basic thick square default. The color can be modified by applying a base style with `lipgloss.NewStyle().BorderForeground(...)`.

Styles can be applied globally and to columns, rows, and individual cells. The base style is applied first, then column, then row, then cell when determining overrides. The default base style is a basic right-alignment. [See the main feature example](https://github.com/Evertras/bubble-table/blob/main/examples/features) to see styles and how they override each other.

Styles can also be applied via a style function which can be used to apply zebra striping, data-specific formatting, etc.

Can be focused to highlight a row and navigate with up/down (and j/k). These keys can be customized with a KeyMap.

Can make rows selectable, and fetch the current selections.

Events can be checked for user interactions.

Pagination can be set with a given page size, which automatically generates a simple footer to show the current page and total pages.

Built-in filtering can be enabled by setting any columns as filterable, using a text box in the footer and `/` (customizable by keybind) to start filtering.

A missing indicator can be supplied to show missing data in rows.

Columns can be sorted in either ascending or descending order. Multiple columns can be specified in a row. If multiple columns are specified, first the table is sorted by the first specified column, then each group within that column is sorted in smaller and smaller groups. [See the sorting example](https://github.com/Evertras/bubble-table/blob/main/examples/sorting) for more information. If a column contains numbers (either ints or floats), the numbers will be sorted by numeric value. Otherwise rendered string values will be compared.

If a feature is confusing to use or could use a better example, please feel free to open an issue.

## Defining table data

[](https://github.com/Evertras/bubble-table#defining-table-data)

A table is defined by a list of `Column` values that define the columns in the table. Each `Column` is associated with a unique string key.

A table contains a list of `Row`s. Each `Row` contains a `RowData` object which is simply a map of string column IDs to arbitrary `any` data values. When the table is rendered, each `Row` is checked for each `Column` key. If the key exists in the `Row`'s `RowData`, it is rendered with `fmt.Sprintf("%v")`. If it does not exist, nothing is rendered.

Extra data in the `RowData` object is ignored. This can be helpful to simply dump data into `RowData` and create columns that select what is interesting to view, or to generate different columns based on view options on the fly (see the [metadata example](https://github.com/Evertras/bubble-table/blob/main/examples/metadata) for an example of using this).

An example is given below. For more detailed examples, see [the examples directory](https://github.com/Evertras/bubble-table/blob/main/examples).

// This makes it easier/safer to match against values, but isn't necessary
const (
  // This value isn't visible anywhere, so a simple lowercase is fine
  columnKeyID = "id"

  // It's just a string, so it can be whatever, really!  They only must be unique
  columnKeyName = "何?!"
)

// Note that there's nothing special about "ID" or "Name", these are completely
// arbitrary columns
columns := []table.Column{
  table.NewColumn(columnKeyID, "ID", 5),
  table.NewColumn(columnKeyName, "Name", 10),
}

rows := []table.Row{
  // This row contains both an ID and a name
  table.NewRow(table.RowData{
    columnKeyID:          "abc",
    columnKeyName:        "Hello",
  }),

  table.NewRow(table.RowData{
    columnKeyID:          "123",
    columnKeyName:        "Oh no",
    // This field exists in the row data but won't be visible
    "somethingelse": "Super bold!",
  }),

  table.NewRow(table.RowData{
    columnKeyID:          "def",
    // This row is missing the Name column, so it will use the supplied missing
    // indicator if supplied when creating the table using the following option:
    // .WithMissingDataIndicator("<ない>") (or .WithMissingDataIndicatorStyled!)
  }),

  // We can also apply styling to the row or to individual cells

  // This row has individual styling to make it bold
  table.NewRow(table.RowData{
    columnKeyID:          "bold",
    columnKeyName:        "Bolded",
  }).WithStyle(lipgloss.NewStyle().Bold(true).  ,

  // This row also has individual styling to make it bold
  table.NewRow(table.RowData{
    columnKeyID:          "alert",
    // This cell has styling applied on top of the bold
    columnKeyName:        table.NewStyledCell("Alert", lipgloss.NewStyle().Foreground(lipgloss.Color("#f88"))),
  }).WithStyle(lipgloss.NewStyle().Bold(true),
}

### A note on 'metadata'

[](https://github.com/Evertras/bubble-table#a-note-on-metadata)

There may be cases where you wish to reference some kind of data object in the table. For example, a table of users may display a user name, ID, etc., and you may wish to retrieve data about the user when the row is selected. This can be accomplished by attaching hidden 'metadata' to the row in the same way as any other data.

const (
  columnKeyID = "id"
  columnKeyName = "名前"
  columnKeyUserData = "userstuff"
)

// Notice there is no "userstuff" column, so it won't be displayed
columns := []table.Column{
  table.NewColumn(columnKeyID, "ID", 5),
  table.NewColumn(columnKeyName, "Name", 10),
}

// Just one user for this quick snippet, check the example for more
user := &SomeUser{
  ID:   3,
  Name: "Evertras",
}

rows := []table.Row{
  // This row contains both an ID and a name
  table.NewRow(table.RowData{
    columnKeyID:       user.ID,
    columnKeyName:     user.Name,

    // This isn't displayed, but it remains attached to the row
    columnKeyUserData: user,
  }),
}

For a more detailed demonstration of this idea in action, please see the [metadata example](https://github.com/Evertras/bubble-table/blob/main/examples/metadata).

## Demos

[](https://github.com/Evertras/bubble-table#demos)

Code examples are located in [the examples directory](https://github.com/Evertras/bubble-table/blob/main/examples). Run commands are added to the [Makefile](https://github.com/Evertras/bubble-table/blob/main/Makefile) for convenience but they should be as simple as `go run ./examples/features/main.go`, etc. You can also view what they look like by checking the example's directory in each README here on Github.

To run the examples, clone this repo and run:

# Run the pokemon demo for a general feel of common useful features
make

# Run dimensions example to see multiple sizes of simple tables in action
make example-dimensions

# Or run any of them directly
go run ./examples/pagination/main.go