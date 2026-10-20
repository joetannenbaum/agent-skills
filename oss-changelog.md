---
name: oss-changelog
---

We're assembling a changelog of all repos across the Laravel org, just surfacing interesting or impactful changes and additions to the ecosystem.

## Step 1

cd ~/Dev/oss-bot
`php artisan changelogs:generate YYYY-MM` (for the date specified)

The command will output the directory of MD files.

Make your changes in the `summary` directory of the dated directory.

## Step 2

Go through each file and divide the items into the three following categories. Do not change _anything_ about the title of each item or the h1 of the document, leave them exactly as they are and just sort them into the following categories. If there are no items in a category, you may exclude it entirely.

### Definitely

Useful additions or features that many developers would be impacted by.

### Maybe

Enhancements that are on the fence. Interesting, but might have a huge impact.

### No

Bug fixes, doc fixes, typo fixes.
