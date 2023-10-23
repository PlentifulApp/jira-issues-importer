# JIRA issues importer

Python 3.x scripts for importing JIRA issues in XML format into an existing GitHub project.

## Background

This project is a fork of a few older projects:
* [hbrands/jira-issues-importer](https://github.com/hbrands/jira-issues-importer)
* [ultimategrandson/jira-issues-importer](https://github.com/ultimategrandson/jira-issues-importer)

There are few similar projects, but these were the least outdated, and the fact that they operate on a Jira XML export rather than on the Jira API directly decreases the complexity and should make it easier to maintain.

# Features

* Import JIRA milestones as Github milestones
* Import JIRA labels as Github labels
* Import JIRA components as Github labels
* Configure colour scheme for labelling on import
* Import multiple files to help overcome the export limit of 1000 (export multiple files by by using the JIRA key column as a range)
* Import JIRA issues as Github issues where
  * issue ids are mapped one by one, e.g. PROJECT-1 becomes GH-1 and PROJECT-4711 becomes GH-4711
  * both issue label and component assignments are mapped to Github labels
  * issue relationships like "depends on", "blocks" or "duplicates" are mapped to special Github comments
  * issue timestamps such as creation, close or update date are considered
  * issue states (open or closed) are considered
  * issue comments are mapped to Github comments
    * JIRA issue references in normal and relationship comments are replaced by references to the Github issue id  

## Caveats
* use these scripts at your own risk, no warranties for a correct and successful migration are given
* this project does not try to map JIRA users to GitHub users
  * the GitHub user which performs the import will appear as issue creator, the original JIRA issue reporter is noted in the first comment
  * the GitHub user which performs the import will also appear as comment creator, as the GitHub API doesn't support that (yet),
    the original JIRA commentator is noted in the comment text

## Assumptions and prerequisites

* you must have installed and authenticated with the `gh` CLI tool from GitHub before starting
* your target GitHub project should already exist with the issue tracker enabled
* it's recommended to test your issue migration first with a test project on GitHub
* input to the import script is the XML export file of your JIRA project, see below
* works with JIRA Cloud, as of March 2019

# Getting started

1. clone this repository
1. run `pip3 install -r requirements.txt`
1. export the desired JIRA issues of your project (see section below)
1. edit the `labelcolourselector.py` if you want to change the logic of how the colours are set on labels
1. to start the Github import, execute 'python main.py'
1. on startup it will ask for
   1. the JIRA XML export file name (use a semi-colon to enter multiple XML paths)
   1. the JIRA project name
   1. the `<statusCategoryId>` element's `id` attribute that signifies an issue as Done 
   1. the GitHub account name (user or organization)
   1. the target GitHub repository name
   1. the index at which to start from, enter 0 to begin, if you have a failure, enter the index number the import failed at. Entering a number higher than 0 will stop labels from re-importing and milestones will re-match to existing.
1. the import process will then
   1. read the JIRA XML export file and create an in-memory project representation of the xml file contents
   1. import the milestones with the regular [GitHub Milestone API](https://developer.github.com/v3/issues/milestones/)
   1. import the labels with the regular [GitHub Label API](https://developer.github.com/v3/issues/labels/)
   1. import the issues with comments with the [GitHub Import API](https://gist.github.com/jonmagic/5282384165e0f86ef105)
      1. references to issues in the comments are replaced with placeholders in this step
      1. the used import API will not run into abuse rate limits in contrast to the normal [GitHub Issues API](https://developer.github.com/v3/issues/)
   1. post-process all comments to replace the issue reference placeholders with the real GitHub issue ids using the [GitHub Comment API](https://developer.github.com/v3/issues/comments/)

## Export JIRA issues

1. Navigate to Issue search page for project. Issues --> Search for Issues
1. Select project you are interested in
1. Specify Query criteria, Sort as needed, if you have more than 1000 items use something like eg. `issuekey < PRO-1000 AND issuekey > PRO-2000` to select a range and export each set into separate XML files
1. From results page, click on Export icon at the top right of page
1. Select XML output and save file

