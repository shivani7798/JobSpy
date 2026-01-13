# How to Run JobSpy for Job Searches in UK

This guide shows you how to use JobSpy to search for jobs in the United Kingdom, including ready-to-use scripts for **Data Engineer** and **Strategic Consultant** positions.

## Prerequisites

First, install JobSpy and required dependencies:

```bash
pip install -U python-jobspy pandas openpyxl
```

**Python version >= 3.10 required**

## Quick Start

### For Data Engineer Jobs
```bash
python run_data_eng.py
```
Output: `data_engineer/data_engineer_jobs_YYYY-MM-DD_HH-MM-SS.xlsx`

### For Strategic Consultant Jobs
```bash
python run_strategy_consultant.py
```
Output: `strategy_consultant/strategy_consultant_jobs_YYYY-MM-DD_HH-MM-SS.xlsx`

---

## Pre-built Scripts

### Data Engineer Jobs

File: `run_data_eng.py`

**Features**:
- 🔍 Searches Indeed, LinkedIn, and Glassdoor
- 📊 50 results per site (customizable)
- 📁 Output to `data_engineer/` folder
- 💅 Professional Excel format with multiple sheets
- 📅 Timestamped filenames

**Excel Output Structure**:
- **Summary** - Statistics (total jobs, companies, salary averages)
- **All Jobs** - Complete job listing
- **Indeed** - Jobs from Indeed
- **LinkedIn** - Jobs from LinkedIn

**To customize**:
```python
# Edit run_data_eng.py
RESULTS_PER_SITE = 50   # Change to 15, 20, 100+ as needed
SEARCH_TERM = "Data Engineer"
LOCATION = "United Kingdom"
SITES = ["indeed", "linkedin", "glassdoor"]
```

---

### Strategic Consultant Jobs

File: `run_strategy_consultant.py`

**Features**:
- 🔍 Searches Indeed, LinkedIn, and Glassdoor
- 📊 50 results per site (customizable)
- 📁 Output to `strategy_consultant/` folder
- 💅 Professional Excel format with multiple sheets
- 📅 Timestamped filenames

**Excel Output Structure**:
- **Summary** - Statistics (total jobs, companies, salary averages)
- **All Jobs** - Complete job listing
- **Indeed** - Jobs from Indeed
- **LinkedIn** - Jobs from LinkedIn

**To customize**:
```python
# Edit run_strategy_consultant.py
RESULTS_PER_SITE = 50
SEARCH_TERM = "Strategic Consultant"
LOCATION = "United Kingdom"
SITES = ["indeed", "linkedin", "glassdoor"]
```

---

## Generic Job Search Template (Any Role)

Create custom scripts for any job role using this template:

### Template: `run_JOB_NAME.py`

```python
import pandas as pd
from openpyxl.styles import PatternFill, Font, Alignment
from jobspy import scrape_jobs
from datetime import datetime
import os

# ========== CONFIGURATION ==========
RESULTS_PER_SITE = 50  # Can be: 15, 20, 50, 100+
JOB_NAME = "Product Manager"  # ← CHANGE THIS
LOCATION = "United Kingdom"
SITES = ["indeed", "linkedin", "glassdoor"]
OUTPUT_FOLDER = "product_manager"  # ← CHANGE THIS (match JOB_NAME)

# Create output folder if it doesn't exist
os.makedirs(OUTPUT_FOLDER, exist_ok=True)

# ========== SCRAPE JOBS ==========
print(f"🔍 Searching for {JOB_NAME} jobs in {LOCATION}...")
print(f"📊 Results per site: {RESULTS_PER_SITE}")

jobs = scrape_jobs(
    site_name=SITES,
    search_term=JOB_NAME,
    location=LOCATION,
    results_wanted=RESULTS_PER_SITE,
    country_indeed='UK',
)

print(f"Found {len(jobs)} jobs")

# ========== GENERATE EXCEL ==========
timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
output_file = f'{OUTPUT_FOLDER}/{JOB_NAME.lower().replace(" ", "_")}_jobs_{timestamp}.xlsx'

with pd.ExcelWriter(output_file, engine='openpyxl') as writer:
    workbook = writer.book
    
    # Define styling
    header_fill = PatternFill(start_color="4CAF50", end_color="4CAF50", fill_type="solid")
    header_font = Font(bold=True, color="FFFFFF")
    
    # Sheet 1: Summary Statistics
    summary_data = {
        'Metric': [
            'Total Jobs Found',
            'Unique Job Sites',
            'Unique Companies',
            'Remote Jobs',
            'Average Min Salary (£)',
            'Average Max Salary (£)',
            'Most Common Job Type'
        ],
        'Value': [
            len(jobs),
            jobs['site'].nunique(),
            jobs['company'].nunique(),
            jobs['is_remote'].sum() if 'is_remote' in jobs.columns else 0,
            f"£{jobs['min_amount'].mean():,.2f}" if jobs['min_amount'].notna().any() else "N/A",
            f"£{jobs['max_amount'].mean():,.2f}" if jobs['max_amount'].notna().any() else "N/A",
            jobs['job_type'].mode()[0] if 'job_type' in jobs.columns and len(jobs['job_type'].mode()) > 0 else "N/A"
        ]
    }
    summary_df = pd.DataFrame(summary_data)
    summary_df.to_excel(writer, sheet_name='Summary', index=False)
    
    # Format summary sheet
    summary_sheet = writer.sheets['Summary']
    for cell in summary_sheet[1]:
        cell.fill = header_fill
        cell.font = header_font
    
    # Sheet 2: All Jobs
    jobs_display = jobs[[col for col in [
        'site', 'title', 'company', 'location', 'job_type', 
        'min_amount', 'max_amount', 'interval', 'is_remote',
        'date_posted', 'job_level', 'company_industry'
    ] if col in jobs.columns]].copy()
    
    jobs_display.to_excel(writer, sheet_name='All Jobs', index=False)
    
    # Format all jobs sheet
    all_jobs_sheet = writer.sheets['All Jobs']
    for cell in all_jobs_sheet[1]:
        cell.fill = header_fill
        cell.font = header_font
        cell.alignment = Alignment(horizontal='center', vertical='center')
    
    # Auto-adjust column widths
    for column in all_jobs_sheet.columns:
        max_length = 0
        column_letter = column[0].column_letter
        for cell in column:
            try:
                if len(str(cell.value)) > max_length:
                    max_length = len(str(cell.value))
            except:
                pass
        adjusted_width = min(max_length + 2, 50)
        all_jobs_sheet.column_dimensions[column_letter].width = adjusted_width
    
    # Sheet 3+: Jobs by Site
    for site in sorted(jobs['site'].unique()):
        site_jobs = jobs[jobs['site'] == site]
        site_sheet_name = site.capitalize()[:31]
        
        site_jobs.to_excel(writer, sheet_name=site_sheet_name, index=False)
        
        # Format site-specific sheet
        site_sheet = writer.sheets[site_sheet_name]
        for cell in site_sheet[1]:
            cell.fill = PatternFill(start_color="2196F3", end_color="2196F3", fill_type="solid")
            cell.font = Font(bold=True, color="FFFFFF")
            cell.alignment = Alignment(horizontal='center', vertical='center')
        
        # Auto-adjust column widths
        for column in site_sheet.columns:
            max_length = 0
            column_letter = column[0].column_letter
            for cell in column:
                try:
                    if len(str(cell.value)) > max_length:
                        max_length = len(str(cell.value))
                except:
                    pass
            adjusted_width = min(max_length + 2, 50)
            site_sheet.column_dimensions[column_letter].width = adjusted_width

print(f"\n✅ Excel file created successfully!")
print(f"📁 File: {output_file}")
print(f"📊 Sheets created:")
print(f"   - Summary (statistics)")
print(f"   - All Jobs ({len(jobs)} jobs)")
for site in sorted(jobs['site'].unique()):
    site_count = len(jobs[jobs['site'] == site])
    print(f"   - {site.capitalize()} ({site_count} jobs)")
```

**To use this template**:

1. Copy and save as `run_software_engineer.py` (or any role name)
2. Change the configuration at the top:
   ```python
   JOB_NAME = "Software Engineer"
   OUTPUT_FOLDER = "software_engineer"
   ```
3. Create the output folder: `mkdir software_engineer`
4. Run it:
   ```bash
   python run_software_engineer.py
   ```

---

## Manual Code Examples

### Basic Single Search

```python
from jobspy import scrape_jobs

jobs = scrape_jobs(
    site_name=["indeed", "linkedin", "glassdoor"],
    search_term="Software Engineer",
    location="United Kingdom",
    results_wanted=50,
    country_indeed='UK',
)

print(f"Found {len(jobs)} jobs")
print(jobs.head())
```

### Search with Job Type Filter

```python
from jobspy import scrape_jobs

jobs = scrape_jobs(
    site_name=["indeed", "linkedin"],
    search_term="Data Engineer",
    location="United Kingdom",
    job_type="fulltime",  # or "parttime", "internship", "contract"
    results_wanted=50,
    country_indeed='UK',
)
```

### Search for Remote Positions

```python
from jobspy import scrape_jobs

jobs = scrape_jobs(
    site_name=["indeed", "linkedin", "glassdoor"],
    search_term="Strategic Consultant",
    location="United Kingdom",
    is_remote=True,
    results_wanted=50,
    country_indeed='UK',
)
```

### Filter Jobs Posted Recently

```python
from jobspy import scrape_jobs

jobs = scrape_jobs(
    site_name=["indeed", "linkedin"],
    search_term="Data Engineer",
    location="United Kingdom",
    hours_old=168,  # Posted in last 7 days
    results_wanted=50,
    country_indeed='UK',
)
```

### Fetch Full Job Descriptions (LinkedIn)

```python
from jobspy import scrape_jobs

jobs = scrape_jobs(
    site_name=["linkedin"],
    search_term="Data Engineer",
    location="United Kingdom",
    linkedin_fetch_description=True,
    results_wanted=50,
    country_indeed='UK',
)
```

---

## Output Columns

| Column | Description |
|--------|-------------|
| SITE | Job board source (indeed, linkedin, glassdoor, etc.) |
| TITLE | Job title |
| COMPANY | Company name |
| LOCATION | Job location |
| JOB_TYPE | Employment type (fulltime, parttime, internship, contract) |
| MIN_AMOUNT | Minimum salary |
| MAX_AMOUNT | Maximum salary |
| INTERVAL | Salary interval (yearly, hourly, monthly) |
| IS_REMOTE | Remote position flag |
| DATE_POSTED | Job posting date |
| JOB_LEVEL | Seniority level |
| COMPANY_INDUSTRY | Industry classification |

---

## Supported Job Boards

- **LinkedIn** - Global support
- **Indeed** - Supports UK (use `country_indeed='UK'`)
- **Glassdoor** - Supports UK (use `country_indeed='UK'`)
- **ZipRecruiter** - US/Canada only
- **Google Jobs** - Global support
- **Bayt** - Middle East focus
- **Naukri** - India focus
- **BDJobs** - Bangladesh focus

---

## Tips & Best Practices

1. **Start with 20-50 results** to test your search before scaling up
2. **Use specific locations**: "London", "Manchester" instead of just "United Kingdom"
3. **Combine multiple sites** for comprehensive results
4. **Save in folders** to keep outputs organized by role
5. **Use timestamped filenames** to track searches over time
6. **Check salary data** - not all jobs include it
7. **Create separate folders** for each job role:
   ```bash
   mkdir data_engineer strategy_consultant software_engineer
   ```

---

## Common Issues

**Issue**: Getting blocked by a job board
- **Solution**: Use proxies or wait before retrying

**Issue**: No salary information
- **Solution**: Not all jobs include salary; depends on job board and posting

**Issue**: Folder permission errors
- **Solution**: Ensure output folders exist and are writable

**Issue**: "country_indeed parameter required"
- **Solution**: Always include `country_indeed='UK'` for UK searches on Indeed/Glassdoor

---

## Project Structure

```
JobSpy/
├── run_data_eng.py                      # Data Engineer search
├── run_strategy_consultant.py           # Strategic Consultant search
├── how_to_run.md                        # This file
│
├── data_engineer/                       # Data Engineer outputs
│   ├── data_engineer_jobs_2026-01-13_04-24-54.xlsx
│   └── data_engineer_jobs_2026-01-13_04-25-12.xlsx
│
└── strategy_consultant/                 # Strategic Consultant outputs
    ├── strategy_consultant_jobs_2026-01-13_04-27-27.xlsx
    └── strategy_consultant_jobs_2026-01-13_04-28-15.xlsx
```

---

## Full Parameter Reference

```python
scrape_jobs(
    site_name=["indeed", "linkedin", "glassdoor"],  # Job boards
    search_term="Data Engineer",                     # Job title
    location="United Kingdom",                       # Location
    distance=50,                                     # Search radius (miles)
    is_remote=False,                                 # Remote jobs only
    job_type="fulltime",                            # fulltime, parttime, internship, contract
    results_wanted=50,                              # Results per site
    country_indeed='UK',                            # Country code
    hours_old=168,                                  # Posted within hours
    easy_apply=None,                                # Easy apply filter
    verbose=1,                                      # 0=errors, 1=warnings, 2=all
)
```

---

## Quick Start Examples

### Search for Data Engineers
```bash
python run_data_eng.py
```

### Search for Strategy Consultants
```bash
python run_strategy_consultant.py
```

### Search for Product Managers (custom)
```bash
# 1. Create folder
mkdir product_manager

# 2. Copy template to run_product_manager.py
# 3. Change JOB_NAME = "Product Manager"
# 4. Change OUTPUT_FOLDER = "product_manager"

python run_product_manager.py
```

---

Happy job hunting! 🎯
