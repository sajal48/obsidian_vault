---
Priority_Level: 
Status: 
Date_Created: 
Due_Date: 
connections: 
tags:
  - project
type: project_family
cssclasses:
  - hide-properties_editing
  - hide-properties_reading
---
# Components
**Select Connection:** `INPUT[inlineListSuggester(optionQuery(#area)):connections]` 
**Date Created:** `INPUT[dateTime(defaultValue(null)):Date_Created]`
**Due Date:** `INPUT[dateTime(defaultValue(null)):Due_Date]`
**Priority Level:** `INPUT[inlineSelect(option(1 Critical), option(2 High), option(3 Medium), option(4 Low)):Priority_Level]`
**Status:** `INPUT[inlineSelect(option(1 To Do), option(2 In Progress), option(3 Testing), option(4 Completed), option(5 Blocked)):Status]`
# Description


# Notes


# Definition of Done


<%* tp.hooks.on_all_templates_executed(async () => { const file = tp.file.find_tfile(tp.file.path(true)); const folder_name = tp.file.folder().toLowerCase().replace(/ /g, "_"); await app.fileManager.processFrontMatter(file, (frontmatter) => { frontmatter["tags"] = [`#project/${folder_name}`]; }); }); -%>
