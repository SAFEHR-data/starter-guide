#CONTRIBUTING

Guide for authors contributing documentation.

This is an intermediate, low maintenance solution that we may change.

You can contribute either :
1. Markdown (.md) documents directly to the root folder
1. Dynamic documents in R markdown (.Rmd) or Python Jupyter notebooks (.ipynb) to the dynamic-docs folder, then these can either be knitted manually or through the github actions to create Markdown (.md) documents in the root folder. (When doing this manually for .Rmd files you may want to delete the YAML header from the knitted .md so that it doesn't render to Github & Slab where these are synched).   

Add a link and row to the table in the [Readme](README.md).