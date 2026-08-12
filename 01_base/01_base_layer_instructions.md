# Base Layer Instructions

The base layer is the layer of machine-generated LookML views that makes it simple to respond to upstream data schema changes. This layer doesn't contain any hand-editing or business logic.

With an isolated base layer, adding this column is a single button click: re-run the Create View From Table LookML generator to get the latest changes. For some databases, Create View From Table will even pull in column-level descriptions into the view definition, so youΓÇÖll even get updated documentation for free with this approach.

Folders
Organize your base folders to mirror how the source data resides in your database, and generate one .view.lkml file per raw table.

This folder hierarchy makes it clear which data powers your Looker models. This is especially helpful if you're referencing tables across different schemas and databases.
