# JobSpy Structured Output Options

Here are several options to create more structured and useful output from JobSpy data:

## Option 1: JSON Output (Most Flexible)
**Best for**: APIs, web applications, data pipelines

```python
import json
from jobspy import scrape_jobs

jobs = scrape_jobs(
    site_name=["indeed", "linkedin", "glassdoor"],
    search_term="Data Engineer",
    location="United Kingdom",
    results_wanted=20,
    country_indeed='UK',
)

# Convert to JSON with proper formatting
jobs_json = jobs.to_json(orient='records', indent=2)
with open('data_engineer_jobs.json', 'w') as f:
    f.write(jobs_json)

print("Saved to data_engineer_jobs.json")
```

**Advantages**:
- Hierarchical structure
- Easily parseable by other tools
- Preserves data types
- Can handle nested data

---

## Option 2: Excel with Multiple Sheets
**Best for**: Data analysis, Excel users, non-technical stakeholders

```python
from jobspy import scrape_jobs

jobs = scrape_jobs(
    site_name=["indeed", "linkedin", "glassdoor"],
    search_term="Data Engineer",
    location="United Kingdom",
    results_wanted=20,
    country_indeed='UK',
)

# Create Excel with multiple sheets
with pd.ExcelWriter('data_engineer_jobs.xlsx', engine='openpyxl') as writer:
    # All jobs on main sheet
    jobs.to_excel(writer, sheet_name='All Jobs', index=False)
    
    # Group by site
    for site in jobs['site'].unique():
        site_jobs = jobs[jobs['site'] == site]
        sheet_name = site.capitalize()[:31]  # Excel sheet name limit
        site_jobs.to_excel(writer, sheet_name=sheet_name, index=False)
    
    # Summary statistics
    summary = pd.DataFrame({
        'Total Jobs': [len(jobs)],
        'Sites': [jobs['site'].nunique()],
        'Companies': [jobs['company'].nunique()],
        'Avg Min Salary': [jobs['min_amount'].mean()],
        'Avg Max Salary': [jobs['max_amount'].mean()],
    })
    summary.to_excel(writer, sheet_name='Summary', index=False)

print("Saved to data_engineer_jobs.xlsx")
```

**Advantages**:
- Formatted and filterable
- Multiple sheets for organization
- Summary statistics
- Easy to share

---

## Option 3: Parquet (Cloud-Optimized)
**Best for**: Data warehousing, big data pipelines, cloud storage

```python
from jobspy import scrape_jobs

jobs = scrape_jobs(
    site_name=["indeed", "linkedin", "glassdoor"],
    search_term="Data Engineer",
    location="United Kingdom",
    results_wanted=20,
    country_indeed='UK',
)

# Save as Parquet
jobs.to_parquet('data_engineer_jobs.parquet')

print("Saved to data_engineer_jobs.parquet")
```

**Advantages**:
- Compressed file size
- Preserves data types
- Fast querying
- Industry standard for data lakes

---

## Option 4: Structured CSV with Better Organization
**Best for**: Data analysis, simple distribution

```python
import csv
from jobspy import scrape_jobs
import pandas as pd

jobs = scrape_jobs(
    site_name=["indeed", "linkedin", "glassdoor"],
    search_term="Data Engineer",
    location="United Kingdom",
    results_wanted=20,
    country_indeed='UK',
)

# Reorder and select key columns
key_columns = [
    'site', 'title', 'company', 'location', 'job_type', 
    'min_amount', 'max_amount', 'interval', 'currency',
    'is_remote', 'date_posted', 'job_url', 'job_level',
    'company_industry', 'description'
]

jobs_structured = jobs[[col for col in key_columns if col in jobs.columns]]

# Save with better formatting
jobs_structured.to_csv(
    'data_engineer_jobs_structured.csv',
    quoting=csv.QUOTE_NONNUMERIC,
    escapechar="\\",
    index=False,
    encoding='utf-8'
)

print(f"Saved {len(jobs_structured)} jobs to data_engineer_jobs_structured.csv")
```

---

## Option 5: Database (SQLite)
**Best for**: Long-term storage, querying, comparisons over time

```python
import sqlite3
from jobspy import scrape_jobs
import pandas as pd

jobs = scrape_jobs(
    site_name=["indeed", "linkedin", "glassdoor"],
    search_term="Data Engineer",
    location="United Kingdom",
    results_wanted=20,
    country_indeed='UK',
)

# Save to SQLite database
conn = sqlite3.connect('jobs.db')
jobs.to_sql('data_engineer_jobs', conn, if_exists='append', index=False)

# Example queries
df = pd.read_sql_query(
    "SELECT site, COUNT(*) as count FROM data_engineer_jobs GROUP BY site",
    conn
)
print("\nJobs by site:")
print(df)

conn.close()
```

**Advantages**:
- Persistent storage
- Query capabilities
- Can store multiple searches
- Track jobs over time

---

## Option 6: Filtered & Grouped HTML Report
**Best for**: Sharing with non-technical stakeholders, presentations

```python
from jobspy import scrape_jobs
import pandas as pd

jobs = scrape_jobs(
    site_name=["indeed", "linkedin", "glassdoor"],
    search_term="Data Engineer",
    location="United Kingdom",
    results_wanted=20,
    country_indeed='UK',
)

# Create HTML report
html = """
<html>
<head>
    <style>
        body { font-family: Arial; margin: 20px; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #4CAF50; color: white; }
        tr:nth-child(even) { background-color: #f2f2f2; }
        .salary { color: green; font-weight: bold; }
        .remote { background-color: #e8f5e9; }
    </style>
</head>
<body>
    <h1>Data Engineer Jobs - UK</h1>
    <p>Found <strong>{}</strong> jobs</p>
    
    <h2>Summary</h2>
    <p>Sites: {} | Companies: {} | Remote: {}</p>
    
    <h2>Jobs</h2>
    {}
</body>
</html>
""".format(
    len(jobs),
    jobs['site'].nunique(),
    jobs['company'].nunique(),
    jobs['is_remote'].sum(),
    jobs[['site', 'title', 'company', 'location', 'min_amount', 'max_amount', 'is_remote']].to_html(index=False)
)

with open('jobs_report.html', 'w') as f:
    f.write(html)

print("Saved to jobs_report.html")
```

---

## Option 7: Combination Script (Recommended)
**Best for**: Production use - saves in multiple formats

```python
import csv
import json
from jobspy import scrape_jobs
import pandas as pd

def save_jobs_all_formats(search_term, location, country='UK', results_wanted=20):
    """Save job results in multiple formats for different use cases"""
    
    jobs = scrape_jobs(
        site_name=["indeed", "linkedin", "glassdoor"],
        search_term=search_term,
        location=location,
        results_wanted=results_wanted,
        country_indeed=country,
    )
    
    base_filename = search_term.lower().replace(' ', '_')
    
    # 1. CSV (standard)
    jobs.to_csv(
        f'{base_filename}.csv',
        quoting=csv.QUOTE_NONNUMERIC,
        escapechar="\\",
        index=False
    )
    print(f"✅ Saved {base_filename}.csv")
    
    # 2. JSON (for APIs/webapps)
    jobs_dict = jobs.to_dict('records')
    with open(f'{base_filename}.json', 'w') as f:
        json.dump(jobs_dict, f, indent=2, default=str)
    print(f"✅ Saved {base_filename}.json")
    
    # 3. Excel (for analysis)
    with pd.ExcelWriter(f'{base_filename}.xlsx', engine='openpyxl') as writer:
        jobs.to_excel(writer, sheet_name='Jobs', index=False)
        
        # Summary stats
        summary = pd.DataFrame({
            'Metric': ['Total Jobs', 'Unique Companies', 'Sites', 'Avg Salary (Min)', 'Avg Salary (Max)'],
            'Value': [
                len(jobs),
                jobs['company'].nunique(),
                jobs['site'].nunique(),
                f"${jobs['min_amount'].mean():.2f}" if jobs['min_amount'].notna().any() else "N/A",
                f"${jobs['max_amount'].mean():.2f}" if jobs['max_amount'].notna().any() else "N/A",
            ]
        })
        summary.to_excel(writer, sheet_name='Summary', index=False)
    print(f"✅ Saved {base_filename}.xlsx")
    
    # 4. Parquet (for data pipelines)
    jobs.to_parquet(f'{base_filename}.parquet')
    print(f"✅ Saved {base_filename}.parquet")
    
    print(f"\n📊 Total jobs saved: {len(jobs)}")
    return jobs

# Usage
jobs = save_jobs_all_formats(
    search_term="Data Engineer",
    location="United Kingdom",
    results_wanted=20
)
```

---

## Comparison Table

| Format | Use Case | Accessibility | File Size | Data Types |
|--------|----------|---------------|-----------|-----------|
| **CSV** | General use | Very High | Large | Basic |
| **JSON** | APIs/Apps | High | Medium | Rich |
| **Excel** | Analysis/Sharing | High | Medium | Good |
| **Parquet** | Data Pipelines | Low | Small | Rich |
| **SQLite** | Long-term Storage | Medium | Small | Rich |
| **HTML** | Reports/Web | Very High | Large | Limited |

---

## My Recommendations

1. **For daily use**: Use **Option 7** (Combination) - saves in CSV, JSON, Excel, and Parquet
2. **For sharing with team**: Use **Option 2** (Excel with multiple sheets)
3. **For data analysis**: Use **Option 3** (Parquet) or **Option 5** (SQLite)
4. **For APIs/integrations**: Use **Option 1** (JSON)
5. **For reports**: Use **Option 6** (HTML) or **Option 2** (Excel)

---

## Implementation

Which format(s) would you like me to implement? I can:
- Create a new script using any of these options
- Modify `run_data_eng.py` to use one or more formats
- Create a configuration system to choose formats dynamically
