---
layout: page
title: qdb Dash - Web Interface
permalink: /guidebook/qdb-dash
parent: /guidebook/
---

The qdb Dash application provides a web-based interface for exploring and analyzing the Antelope Quantity Database (qdb). It allows you to search for flowables, contexts, and LCIA quantities, examine their details, and build custom selections for analysis.

## Contents
{:.no_toc}

* TOC
{:toc}

## Accessing QDB Dash

The qdb Dash application is currently available in demo mode. Navigate to:

```
https://bk.vault.lc/qdb.dash/
```
(note the trailing slash)

> The demo version of qdb is running on a server with limited resources, so please be patient!
{: .prompt-tip }


[![qdb landing page](/assets/img/qdb-landing-full.png)](https://bk.vault.lc/qdb.dash/)

The interface consists of three main pages:
- **Search/Landing Page**: Search and browse flowables, contexts, and quantities
- **Detail Pages**: View detailed information about individual entities
- **Analysis Page**: Analyze your current selection and export results

## Search Interface

### Basic Search

The search interface is the home page of QDB Dash. It displays a search bar at the top and three columns below for results:

- **Flowables**: Flows of materials or energy (e.g., "carbon dioxide", "electricity")
- **Contexts**: Environmental compartments (e.g., "air", "water, ground-")
- **Quantities**: LCIA methods and impact categories (e.g., "Global Warming", "Acidification")

To perform a search:

1. Enter a search term in the search box
2. Press Enter or click the "Search" button
3. Results appear in the three columns below

Each result shows:
- The entity name and ID
- Three action buttons:
  - **Eye icon**: View detailed information
  - **Plus/Minus icon**: Add to or remove from selection
  - **X icon**: Remove from current search results

### Progressive Search Narrowing

QDB Dash supports progressive search refinement. After performing an initial search, you can enter additional terms to narrow down your results:

1. Perform an initial search (e.g., "carbon")
2. Results are displayed showing all matching entities
3. Enter a second search term (e.g., "dioxide")
4. Results are filtered to show only items matching both terms

The current search queries are displayed as tags above the search box. This allows you to track how you've refined your search.

**Example workflow:**
```
Search 1: "climate" → 150 results
Search 2: "ipcc"    → 45 results (from previous 150)
Search 3: "2013"    → 12 results (from previous 45)
```

To start a new search from scratch, click the "Clear Search" button.

### Bulk Actions

Each column has an "Add All" button at the top right. This adds all visible results in that column to your selection in one click. This is useful when you've narrowed down your search and want to select everything that matches.

### Managing Search Results

You can remove individual items from your search results by clicking the X icon on each result card. This doesn't affect your selection, only the visible search results.

To clear all search results and start fresh, click the "Clear Search" button at the top of the page.

## Detail Pages

Click the eye icon on any search result to view detailed information about that entity. Detail pages show:

### Flowable Details
- Full flowable name
- Associated terms and synonyms
- Number of characterization factors available
- List of quantities that characterize this flowable

### Context Details
- Full context path (e.g., "emission/air/urban air close to ground")
- Parent contexts
- Subcontexts
- Associated flowables

### Quantity Details
- Full LCIA method name
- Impact category
- Reference unit
- Indicator information
- Characterization factors

Each detail page includes an "Add to Selection" button that adds the entity to your current selection. If the entity is already selected, the button shows "Remove from Selection" instead.

Use the back button at the top left to return to the search page.

## Selection Management

The banner at the top of every page shows your current selection count. Your selection persists across page navigation and is stored in your browser session.

### Building a Selection

Add entities to your selection by:
- Clicking the plus icon on search results
- Clicking "Add All" on a results column
- Clicking "Add to Selection" on detail pages

Toggle entities by clicking the plus/minus icon again to remove them.

### Viewing Your Selection

Navigate to the Analysis page to see the full list of items in your selection. They're organized in three columns:
- Flowables
- Contexts
- Quantities

Each item has a remove button (X icon) to take it out of your selection.

### Clearing Your Selection

Click the "Clear" button in the top banner to remove all items from your selection at once.

## Analysis Page

Click the "Analyze" button in the top banner to view and analyze your selection.

### Running Analysis

The Analysis page shows:
1. **Current Selection**: Lists all selected flowables, contexts, and quantities
2. **Analysis Options**: Choose the type of analysis to run
   - Summary Table: View data in tabular format
   - Chart View: Visualize data (future feature)
   - Detailed Report: Generate comprehensive report (future feature)
   - Debug: View raw data for troubleshooting
3. **Results Display**: Shows analysis output after clicking "Run Analysis"

### Analysis Types

**Summary Table**: Displays characterization factors in a searchable, sortable table. Each row shows:
- Flowable name
- Context
- Quantity/LCIA method
- Characterization factor value
- Unit

The table supports:
- Sorting by clicking column headers
- Filtering using the search boxes above each column
- Pagination for large result sets

**Debug Mode**: Shows detailed information about factors and coverage for selected flowables. Useful for understanding data availability and troubleshooting missing factors.

### Exporting Results

Select an export format from the dropdown:
- **CSV**: Plain text, opens in Excel or any text editor
- **Excel**: Native .xlsx format with formatting
- **JSON**: Machine-readable format for programmatic use

Click "Export Results" to download your analysis data. (Note: Export functionality requires implementation in your deployment)

## Workflows

### Finding Climate Change Methods

A common task is finding and comparing different climate change LCIA methods:

1. Search for "climate change" or "global warming"
2. Review the results in the Quantities column
3. Click "Add All" to select all climate methods
4. Navigate to the Analyze page
5. Run a Summary Table analysis to see all characterization factors
6. Filter by specific flowables (e.g., "carbon dioxide", "methane")
7. Export to Excel for further comparison

### Exploring Flowable Synonyms

To understand what terms are associated with a flowable:

1. Search for a flowable (e.g., "CO2")
2. Click the eye icon to view details
3. Review the list of associated terms
4. Try searching with different synonyms to see coverage differences

### Building Custom Selections

For specialized analysis (e.g., water impacts in agriculture):

1. Search for water-related flowables:
   - Search: "water"
   - Click "Add All" in Flowables column
2. Refine contexts:
   - Clear search
   - Search: "water" (to find water compartments)
   - Select relevant contexts from Contexts column
3. Select water impact categories:
   - Clear search
   - Search: "water use" or "water scarcity"
   - Add relevant quantities
4. Navigate to Analyze page
5. Run analysis to see water characterization factors

### Debugging Missing Factors

If you expect characterization factors but don't see them:

1. Build a selection with:
   - Specific flowable(s)
   - Relevant context(s)
   - Expected LCIA method(s)
2. Go to the Analysis page
3. Select "Debug" from Analysis Type dropdown
4. Click "Run Analysis"
5. Review the debug output showing:
   - Which flowables have factors
   - Which quantities cover each flowable
   - Missing combinations

## Tips and Tricks

### Search Strategies

- **Start broad, then narrow**: Begin with general terms, then refine with progressive search
- **Try synonyms**: Different databases use different names (e.g., "CO2" vs "carbon dioxide")
- **Use partial matches**: Searching "phosph" will match "phosphorus", "phosphate", "phosphoric", etc.
- **Case-insensitive**: All searches ignore case

### Selection Management

- Use the selection count in the banner to track your progress
- Your selection persists while browsing, so you can search multiple times and build it up
- Visit the Analyze page periodically to review what you've selected
- Clear your selection between different analyses to avoid confusion

### Navigation

- The logos in the banner are clickable:
  - Left logo returns to search page
  - Right logo links to Antelope documentation
- Use the browser back button to return to previous pages (your selection is preserved)
- The Analyze and Debug buttons provide quick access without leaving your current page

## Technical Details

### Data Sources

QDB Dash queries the backend QDB API, which aggregates data from multiple sources:
- LCIA method databases (e.g., openLCA, TRACI)
- Flow lists (e.g., Federal Commons flow list)
- Context hierarchies from various databases

### Session Storage

Your selection and search results are stored in your browser session using sessionStorage. This means:
- Your selection persists across page navigation
- Closing the tab or browser window clears your selection
- Opening multiple tabs creates independent sessions

### Performance

- Initial searches may take a few seconds depending on database size
- Progressive narrowing is faster since it filters existing results
- Detail pages load on demand
- Analysis can take time for large selections (100+ items)

## Troubleshooting

### Search returns no results

- Verify spelling of search terms
- Try broader terms or partial words
- Check that the database is properly loaded
- Look for error messages in the browser console (F12)

### Selection not updating

- Check the selection count in the banner
- Navigate to the Analyze page to verify the selection contents
- Try clearing your selection and starting fresh
- Refresh the page if the UI appears stuck

### Analysis not running

- Ensure you have items in your selection
- Check the browser console for errors
- Verify the backend API is responding
- Try the Debug analysis type for more detailed error information

### Export not working

- Verify export functionality is configured in your deployment
- Check browser download settings
- Look for error messages in the browser console

## Related Resources

- [QDB Overview](/guidebook/qdb) - Learn about the Quantity Database
- [Antelope Quickstart](/guidebook/quickstart) - Get started with Antelope
- [Glossary](/guidebook/glossary) - Definitions of key terms

[guidebook home](/guidebook)
