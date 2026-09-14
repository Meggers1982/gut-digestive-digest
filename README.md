# Gut & Digestive Health Research Digest

A GitHub Actions workflow that searches curated gastroenterology, the gut microbiome, digestive disease, nutritional science, and microbiology journals on PubMed, filters out widely covered stories, runs a single Claude pass for journalist-ready summaries and pitch angles, and publishes results to a GitHub Pages dashboard.

## How it works

1. **PubMed search** - Queries journals by ISSN for studies published in the past 7 days
2. **Title screening** - Prioritizes studies with novelty signals and excludes animal-only studies
3. **SERPAPI media filter** - Checks Google News and skips any study with 3+ news results
4. **Abstract fetch** - Retrieves full abstracts for shortlisted studies
5. **Claude pass** - Writes structured JSON: headline, summary, why it matters, caveats, relevance score, and pitch angles per publication type
6. **Artifact upload** - Saves JSON results as a GitHub Actions artifact
7. **Deploy job** - Downloads all job artifacts, merges and deduplicates by PMID, commits `data/results.json`, serves via GitHub Pages
8. **Email notification** - Sends a short email with study count and a dashboard link

## Dashboard

Features:
- Card view per study with headline, summary, caveats, fact-check notes
- Expandable pitch angles section for publications such as EatingWell, Well+Good, Healthline, Self, Prevention, Gastroenterology & Endoscopy News, and general health outlets
- Filter by category, groundbreaking type, status, date range, and score
- Search across all study text and pitches
- Status tracking (New / Saved / Pitched / Passed) saved to localStorage
- Deduplication across runs by PMID

## Schedule

Runs automatically every morning at 7:00 AM ET. All jobs run in parallel; the deploy job merges results and publishes the dashboard once complete.

Can also be triggered manually via **Actions -> Gut & Digestive Health Research Digest -> Run workflow**.

## Categories

| Category | Journals | Jobs |
|---|---:|---|
| Gastroenterology | 91 | 2 (chunks 1-2) |
| Nutritional Sciences | 62 | 2 (chunks 1-2) |
| Microbiology | 121 | 2 (chunks 1-2) |

Large categories are split into chunks to keep run times under 20 minutes.

Every row in a category CSV is searched with no topic filter, so adding a journal brings its entire weekly output into the digest. The CSVs in `data/` are now hand-maintained: `scripts/extract_journals.py` built them from a source workbook that no longer exists, and re-running it would wipe hand-added rows.

## Journal list audit (2026-09-14)

Method: pulled OpenAlex's top sources for this digest's subject areas (gastroenterology, nutrition, microbiology) over the prior year, diffed them against the three CSVs by ISSN and title, and kept only titles that PubMed indexes with recent articles. Each survivor was then checked for beat fit, English-language research content, and volume (at least 20 PubMed articles a year).

Added to `Gastroenterology.csv` (6):

| Journal | ISSN | PubMed articles/yr |
|---|---|---:|
| Journal of Hepatocellular Carcinoma | 2253-5969 | 276 |
| Hepatology Research | 1872-034X | 267 |
| Liver Cancer | 1664-5553 | 125 |
| Journal of Neurogastroenterology and Motility | 2093-0887 | 77 |
| iLIVER | 2772-9478 | 43 |
| Canadian Liver Journal | 2561-4444 | 31 |

The two liver-cancer titles sit alongside the existing GI-oncology rows (Gastric Cancer, Clinical Colorectal Cancer). None of the additions are currently MEDLINE-indexed; all reach PubMed through PMC.

Left out:
- **Sexually Transmitted Diseases** - off-beat (OpenAlex lumped it in via microbiology).
- **Cereal Chemistry, Journal of Cereal Science, Starch - Stärke** - food science, off-beat; also 0-1 PubMed articles a year.
- **Portal Hypertension & Cirrhosis** - in-beat but only 2 PubMed articles in the past year.
- **Not usable because PubMed doesn't carry them:** Sexually Transmitted Infections, Journal of Water Sanitation and Hygiene for Development, Egyptian Liver Journal, and Hepatoma Research (0 PubMed articles in the past year); Clínicas de gastroenterología de México (both editions), World Nutrition, and a Korean food-service journal (not in NCBI's journal list).

No mega-journals turned up among the candidates. Gastroenterology grew by about 7%, so the workflow chunking is unchanged.

## Manual Trigger

Go to **Actions -> Gut & Digestive Health Research Digest -> Run workflow**.

- Leave **category** blank to run all jobs
- Enter an exact category name, such as `Gastroenterology`, to run just that category

## GitHub Pages Setup

1. Go to **Settings -> Pages**
2. Set source to **Deploy from a branch**
3. Branch: `main`, folder: `/ (root)`
4. Save; GitHub will serve `index.html` at the dashboard URL

## Required Secrets

Add these in **Settings -> Secrets and variables -> Actions**:

| Secret | Description |
|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `SERPAPI_KEY` | SerpAPI key for Google News filtering |
| `SUPABASE_URL` | Supabase project URL (dashboard save/delete personalization) |
| `SUPABASE_KEY` | Supabase API key (read-only) |
| `DASHBOARD_REPO_TOKEN` | Token with push access to the shared `research-digest-dashboard` repo |

## Repo Structure

```text
.github/
  workflows/
    gut-digestive-digest.yml
scripts/
  gut_digestive_digest.py
  merge_results.py
  extract_journals.py
data/
  Gastroenterology.csv
  Nutritional Sciences.csv
  Microbiology.csv
  results.json
index.html
requirements.txt
```

## Dashboard Study Card Fields

Each study card shows:

- **Headline** - plain-language present-tense summary
- **Relevance score** - 1-10, weighted for gut health, the microbiome, diet, and digestive medicine journalism fit
- **Category & journal** - source metadata
- **Groundbreaking type** - counterintuitive, overturns prior research, first-in-class, or domain-relevant finding
- **Media coverage** - SERPAPI verification status
- **The study** - what was done, who participated, and the key finding
- **Why it matters** - real-world significance for the target audience
- **Caveats** - limitations flagged automatically
- **Fact-check note** - corrections made during the Claude pass
- **Pitch angles** - expandable publication-specific pitch blocks
- **Status** - New / Saved / Pitched / Passed, tracked in your browser
