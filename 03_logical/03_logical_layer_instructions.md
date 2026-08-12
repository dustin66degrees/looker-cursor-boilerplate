Explores define the relationship between tables; Starting point for self-service & data exploration

# Logical Layer Instructions

The logical layer is the layer of Explore definitions based on standard layer views.
The include statement will reference the standard layer so that we get the refined fields as well as the base fields.

The .explore extension clarifies that this file contains an Explore definition. Separating each explore into its own file solves the problem of frequent merge conflicts among developers working on different Explores simultaneously. It also allows more granular control of exposing explores for different models.

What belongs in the standard layer vs. the logical layer?
The line here can be a bit blurry, but generally you should put low-level, global changes like renaming columns, hiding columns, adding descriptions, and simple measures, into the standard layer and more complex changes in the logical layer (e.g. cross-view calculations, which weΓÇÖll explain next).

Where should we define ΓÇ£cross-view calculationsΓÇ¥
Cross-view calculations are fields that use fields from more than one view file. In the traditional setup, youΓÇÖd pick one of those view files and define it there with an include statement for the other view. But this will often throw warnings and/or errors when you re-use this view file in an Explore that doesnΓÇÖt have the other views joined in.

The solution to this problem is LookerΓÇÖs best unofficially documented feature, called field-only views (FOVs). FOVs are view definitions without a sql_table_name parameter. They're a great tool for defining cross-view calculations.

