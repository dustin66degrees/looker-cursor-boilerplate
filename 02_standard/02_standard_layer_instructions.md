One per table; defines the dimension/measures within the table

# Standard Layer Instructions

The standard layer is made up of hand-written LookML refinements that preserve your column changes when you update the base layer.

The standard layer makes use of an underrated feature called refinements (notice the + symbol in the view definition above). Refinements are similar to extends. Extends make a copy of the extended view file, but refinements make the edits in place. For low-level changes that youΓÇÖre comfortable applying across your entire project, refinements are a more concise way to add business logic to a view without editing the base file.

Some examples of business logic you should add to the standard layer include:

Renaming columns to be more human-readable
Defining primary keys or composite primary keys
Adding descriptions and labels
Adding measures

Folders
Similar to the base layer, organize your standard layer folders according to your database structure, and create one .layer.lkml file per raw table.

The new .layer file extension might seem strange at first glance, but choosing "Create Generic LookML File" will allow you to create a LookML file with any arbitrary file extension before .lkml. Adding this extension clarifies that the file contains hand-written code and not machine-generated code.

To recap, for each table in your database there should be one machine-generated base view called <table>.view.lkml and one hand-written standard layer called <table>.layer.lkml.
