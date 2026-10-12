# One developer, one year, 1.8 billion rows: a point-in-time alternative-data platform

*Also published on [dev.to](https://dev.to/alexbsolodev/one-developer-one-year-18-billion-rows-a-point-in-time-alternative-data-platform-382l), 12 October 2026.*

In the last year I built, alone, a quantitative research data platform: about 1.8 billion rows across 764 tables of alternative data, SEC filings, government records and market data, engineered to be trusted in a backtest. I am an Army veteran and self-taught. The discipline came from the work itself: 206 architecture decision records and 680 research notes, one for every problem worth not solving twice.

The hard problems were not the ones anyone warns you about. They were data problems.

## The stack, in the terms the field uses

- **Point-in-time (as-of) data.** Every record carries when it became public, not just when it happened, and 103 tables are bitemporal (valid time and recorded time). That is the defence against look-ahead bias, the most common way a backtest flatters itself.
- **Survivorship-bias-free history.** Delisted and renamed companies stay in the data with their history, and a window in which a stock dies is scored with its delisting return instead of being dropped.
- **Entity resolution and a security master.** Record linkage across tickers, CIKs, LEIs and CUSIPs, with point-in-time symbology: tickers get renamed, reused and delisted, so "which company was this on that date" is answered as of the date.
- **Corporate actions.** Split and dividend adjustment validated against an independent source rather than against itself; the split-adjustment tool is open source.
- **Information extraction (NLP) from regulatory filings.** 8-K events, earnings guidance and reported results, press releases, IPO prospectuses, SEC comment letters, material contracts, Schedule 13D/13G beneficial ownership, insider Forms 3/4/5, foreign-issuer reports and XBRL financial statements, turned into structured, dated facts.
- **A medallion lakehouse.** Bronze, silver and gold layers on Parquet, PostgreSQL for the serving tables, about 456 data contracts guarding them, and data lineage that answers "what breaks downstream if this table changes" in one command.
- **MLOps and model validation.** Feature pipelines, walk-forward and purged cross-validation, backtesting and paper trading, because a model that cannot fail a test is a horoscope.
- **Domain-specific language models.** Small models fine-tuned for narrow financial extraction tasks on a desktop GPU.
- **Agentic workflows.** Model Context Protocol (MCP) servers that let AI agents query the data, and agent-to-agent orchestration around them.
- **All of it on one workstation,** with a resource broker that schedules heavy jobs so the machine does not eat itself.

By the numbers: 4,776 commits since 21 November 2025, 5,281 Python files, 738 of them tests.

## Three lessons in data quality

**1. Labels lie; specifications do not.** The SEC's fund-holdings data (Form N-PORT) has a field called "Ticker". Measured across every quarter the SEC has published, about 28% of its 17.4 million values are not stock tickers: mortgage-pool codes, bond issuers' symbols, futures contracts. One pool code, FG, is also an NYSE ticker, so a naive join turns Freddie Mac mortgage pools into shares of an insurance company. The fix was metadata management, not cleverer code: the SEC's own form instructions say the field is a fallback identifier. I now build a data dictionary (a codebook, in the ISO/IEC 11179 sense) from each agency's official documentation before touching its data. The first seven dictionaries turned up twelve defects in the SEC's own published files.

**2. A test that compares data with itself proves nothing.** One evening every data-quality check on prices passed. The next morning an independent reconciliation found 236 days of prices rewritten onto the wrong side of a stock split. Every check had been comparing the pipeline with its own output. Now a data-quality claim needs an external witness.

**3. Coverage is not precision.** The easy metric in entity resolution is match rate. The useful one is precision. A shortcut that raised one match rate by ten points was right only 38% of the time against an independent identifier. It never shipped.

## What is public

Two open-source releases so far:

- **split-adjustment-tool** - verify, repair and apply stock-split adjustments against your own as-traded price tape. Vendor-neutral Python (Polars), Apache 2.0, release 0.4.0: https://github.com/AlexBocio/split-adjustment-tool
- **us-gov-data-dictionary** - machine-readable data dictionaries for seven SEC data sets: 1,469 fields, each with the SEC's own definition and citation, official code lists, and a profile of what the published files actually contain. W3C CSVW and SKOS formats, CC BY 4.0: https://github.com/AlexBocio/us-gov-data-dictionary

More agencies are next. If you work with public financial data and it has ever quietly lied to you, I would like to hear the story.
