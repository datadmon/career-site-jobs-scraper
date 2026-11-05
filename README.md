# Workday, Greenhouse, Lever & Ashby Jobs Scraper (Python): career-site jobs to JSON and CSV

[![Run on Apify](https://img.shields.io/badge/Run%20on-Apify-00A67E?logo=apify&logoColor=white)](https://apify.com/data_daemon/career-site-jobs-scraper)
![Output](https://img.shields.io/badge/output-JSON%20%7C%20CSV%20%7C%20Excel-blue)
![License: MIT](https://img.shields.io/badge/license-MIT-green)

Scrape **jobs from company career sites** on **Workday** (myworkdayjobs.com), **Greenhouse**, **Lever**, **Ashby** and **SmartRecruiters** into one JSON/CSV schema from Python: company, title, locations, country, workplace type, department, **salary range** where the employer publishes it, posting date, apply URL and the **full description**. Give career-site URLs, or search **10,000+ company boards at once** by title, location and posting date.

[Workday, Greenhouse, Lever and Ashby](https://www.myworkdayjobs.com) is the applicant-tracking systems behind most company career sites (with SmartRecruiters). This repository is a small, tested Python client for the hosted
**[Career Site Jobs API – Workday, Greenhouse, Lever, Ashby](https://apify.com/data_daemon/career-site-jobs-scraper)** on Apify: no browser, proxy, login or scraper maintenance on your side. Price:
**$1 per 1,000 jobs, full description included, plus $0.001 per run**; Apify's free plan ($5 of credit every month) covers about 5,000 jobs.

## Quick start

```bash
pip install apify-client
export APIFY_TOKEN=your_token      # Apify Console > Settings > Integrations
python scrape.py --keyword "data engineer" --max 50
```

`scrape.py` starts the Actor, waits for it and writes `career-jobs.json` (every field) and `career-jobs.csv`
(the main columns). For any other option, put the Actor input in a file and pass `--input`:

```bash
python scrape.py --input examples/input.json
```

```json
{
  "platforms": [
    "greenhouse",
    "lever",
    "ashby"
  ],
  "keywords": [
    "data engineer"
  ],
  "postedWithinDays": 7,
  "maxJobs": 5
}
```

## Sample output

A real run of [`examples/input.json`](examples/input.json) on 6 November 2026
([full JSON](examples/sample-output.json), [CSV](examples/sample-output.csv); descriptions shortened):

| platform | company | title | locations | workplaceType | postedAt |
|---|---|---|---|---|---|
| ashby | Clera | Lead Data Platform Engineer | Berlin | onsite | 2026-11-03 |
| greenhouse | SpaceX | Software Engineer, Full Stack (Data & Video Analysis) | Hawthorne, CA |  | 2026-11-03 |
| greenhouse | Encora | Senior Data Engineer AWS | Peru |  | 2026-11-03 |
| greenhouse | Stripe | Senior Staff Engineer, Big Data Infrastructure | San Francsico |  | 2026-11-05 |
| greenhouse | Coupang | Data Engineering Lv.4 | Bengaluru |  | 2026-11-03 |

## Output fields

| Field | Description |
|---|---|
| `id` | Stable id: platform:company:jobId. |
| `platform` | workday, greenhouse, lever, ashby or smartrecruiters. |
| `company` | Company (board name, or the career-site slug). |
| `legalEntity` | Workday: the legal entity posting the job. |
| `careerSite` | The career-site URL you gave. |
| `jobId` | The platform's job id. |
| `requisitionId` | Employer's requisition / reference number. |
| `title` | Job title. |
| `url` | Job page. |
| `applyUrl` | Apply page. |
| `locations` | Locations as the employer lists them. |
| `country` | Country when the platform states it. |
| `workplaceType` | remote, hybrid or onsite (as stated). |
| `department` | Department or job family. |
| `team` | Team. |
| `employmentType` | Full-time, Part-time, Contract, Internship... |
| `experienceLevel` | SmartRecruiters experience level. |
| `salaryMin` | Minimum pay (number). |
| `salaryMax` | Maximum pay (number). |
| `salaryCurrency` | ISO currency of the pay range. |
| `salaryPeriod` | year, month, hour... |
| `salaryText` | Pay as the employer wrote it. |
| `postedAt` | First published (ISO 8601). |
| `postedText` | Workday's relative posting text, e.g. 'Posted 3 Days Ago'. |
| `description` | Full description as text (descriptions on). |
| `scrapedAt` | When the job was read. |
| `organization` | Compatible key (compatibleOutput): same as company. |
| `date_posted` | Compatible key (compatibleOutput): same as postedAt. |
| `locations_derived` | Compatible key (compatibleOutput): same as locations. |
| `ai_work_arrangement` | Compatible key (compatibleOutput): Remote OK, Hybrid or On-site, from workplaceType (as the employer states it, not AI-derived). |
| `ai_salary_min_value` | Compatible key (compatibleOutput): same as salaryMin. |
| `ai_salary_max_value` | Compatible key (compatibleOutput): same as salaryMax. |
| `ai_salary_currency` | Compatible key (compatibleOutput): same as salaryCurrency. |
| `ai_salary_unit_text` | Compatible key (compatibleOutput): YEAR, MONTH, WEEK, DAY or HOUR, from salaryPeriod. |
| `source` | Compatible key (compatibleOutput): the platform (greenhouse, lever, ashby, workday, smartrecruiters). |

## Daily feed of only new jobs

```bash
python scrape.py --keyword "data engineer" --only-new career-daily --max 0
```

With `--only-new <name>` the Actor remembers which jobs it has delivered under that name and returns only new ones;
skipped jobs are not charged. Run it from cron, a GitHub Actions schedule or an Apify schedule:

```yaml
# .github/workflows/career-daily.yml
on:
  schedule: [{cron: "30 2 * * *"}]
  workflow_dispatch:
jobs:
  scrape:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install apify-client && python scrape.py --keyword "data engineer" --only-new career-daily --max 0
        env: {APIFY_TOKEN: "${{ secrets.APIFY_TOKEN }}"}
      - uses: actions/upload-artifact@v4
        with: {name: career-jobs, path: "career-jobs.*"}
```

## Other ways to call it

**curl**, one request that runs the Actor and returns the rows:

```bash
curl -X POST "https://api.apify.com/v2/acts/data_daemon~career-site-jobs-scraper/run-sync-get-dataset-items" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"platforms": ["greenhouse", "lever", "ashby"], "keywords": ["data engineer"], "postedWithinDays": 7, "maxJobs": 5}'
```

**Apify CLI:** `apify call data_daemon/career-site-jobs-scraper --input-file examples/input.json`

**No code:** open the [Actor page](https://apify.com/data_daemon/career-site-jobs-scraper), fill the form and export to CSV/Excel, or connect it to Google Sheets,
Zapier, Make or n8n. AI agents can call it through the Apify MCP server (`https://mcp.apify.com`).

## FAQ

### Do Workday, Greenhouse, Lever and Ashby have job APIs?

Greenhouse, Lever and Ashby publish per-company job-board APIs, and Workday career sites load their jobs from a JSON endpoint; none of them offers one search across all companies. The hosted Actor behind this repo reads those public endpoints for the sites you give, or searches its built-in list of 10,000+ company boards, and returns one schema.

### Can I search all companies for a job title?

Yes: leave the career-site URLs empty and set `keywords`, `locations` and `postedWithinDays` (default 7 days in this mode). Narrow it with `platforms` (`workday`, `greenhouse`, `lever`, `ashby`, `smartrecruiters`). It is a live search over a fixed list of boards, so a company that is not on the list needs its URL in `careerSiteUrls`.

### Does it accept the input field names other career-site job APIs use?

Yes: `titleSearch`, `locationSearch`, `timeRange` (for example `7d`) and `limit` are read as aliases, and `compatibleOutput: true` adds `organization`, `date_posted`, `locations_derived` and `description_text` to each row.

### Is it legal to scrape Workday, Greenhouse, Lever and Ashby?

The Actor reads only public job listings, without logging in, at a polite request rate, and returns job and company
data, not personal profiles. Check the rules that apply to your use of the data in your jurisdiction.

## Related

- [Oracle Jobs Scraper – Oracle HCM & Recruiting Cloud Career Site](https://apify.com/data_daemon/oracle-hcm-jobs-scraper)
- [Welcome to the Jungle Jobs Scraper | WTTJ API & New Job Feed](https://apify.com/data_daemon/welcome-to-the-jungle-jobs-scraper)
- [India Jobs Feed | Naukri + 9 Indian Job Boards in One API](https://apify.com/data_daemon/india-jobs-feed)
- [India Job Alerts](https://github.com/datadmon/india-job-alerts): new jobs from seven Indian boards every morning via GitHub Actions
- [All Daemon Data Actors](https://apify.com/data_daemon)

Unofficial project, not affiliated with Workday, Greenhouse, Lever and Ashby. Issues and questions: open an issue here or on the
[Actor's Issues tab](https://apify.com/data_daemon/career-site-jobs-scraper/issues).
