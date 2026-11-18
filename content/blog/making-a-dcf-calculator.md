+++
title = "Making a DCF Calculator"
date = "2026-11-18"
tags = [
    "development",
    "project",
    "Python",
    "finance",
]
+++

In the fall semester of my Masters of Science in Business Analytics (MSBA) degree program at Cornell University, I took an introductory course in financial analytics. It was not very long, only half a semester, but I found the topics presented to be very interesting.

The takeaway I took from that course is:

> "A dollar today is worth more than a dollar tomorrow."

That is the core idea behind **Time Value of Money (TVM)**, which states that money today has a higher value than money in the future, due to earning potential, inflation, and uncertainty.

Upon finishing the course, I felt the spark of inspiration to make a Discounted Cash Flow model, applying the TVM formula using Python instead of Excel spreadsheets.

### Schemas and Valuation Logic

To follow along, you can check out the repository [here](https://github.com/vkumar825/dcf-calculator).

The first thing I did after creating the project scaffolding using `uv init` was to plan out the schema in `dcf_schemas.py`.

I used `pydantic`, which is a Python library that helps to define strict data validation for the schema and enforce type safety for deterministic values.

The first class, `AnnualFinancials`, acts as a container class that will be composed of other classes. The idea is that annual financials contain an income statement, cash flow statement, and a balance sheet for a given company and fiscal year.

There are three financial metrics classes: `IncomeStatement`, `CashFlowStatement`, and `BalanceSheet`. These are for storing the float values, retrieved from the company's 10-K filing statements, and used in the various DCF formulas.

The `ValuationAssumptions` class is for storing user inputs. Like the financial metric properties, they, too, will be used in the various formulas.

Finally, we have `dcf.py`, which is where all the formulaic calculations happen, and the steps that build out the DCF model.

The crucial function in this file is the runner function shown below:

```python
    def run_valuation(self) -> dict[str, Any]:
        return {
            "effective_tax_rate": self.calculate_effective_tax_rate(),
            "pre_tax_cost_of_debt": self.calculate_pre_tax_cost_of_debt(),
            "cost_of_equity": self.calculate_cost_of_equity(),
            "wacc": self.calculate_wacc(),
            "base_ufcf": self.calculate_base_ufcf(),
            "projection_schedule": self.generate_projection_schedule(),
            "present_values": self.calculate_present_values(),
            "pv_of_fcf": self.calculate_pv_of_fcf(),
            "terminal_value": self.calculate_terminal_value(),
            "pv_of_terminal_value": self.calculate_pv_of_terminal_value(),
            "enterprise_value": self.calculate_enterprise_value(),
            "equity_value": self.calculate_equity_value(),
            "implied_share_price": self.calculate_implied_share_price(),
        }
```

This function runs the calculation functions in order, closely mirroring the steps you would take to do a DCF model in Excel spreadsheets, after you retrieve the values from a firm's 10-K filing:

1. **Calculate the Cost of Equity via the Capital Asset Pricing Model (CAPM)**
2. **Find the discount rate using Weighted Average Cost of Capital (WACC)**
3. **Estimate Terminal Value (TV)**
4. **Discount both Cash Flows and Terminal Value to get Enterprise Value**
5. **Bridge Present Values (PV) of Cash Flows to Equity Value**
6. **Calculate the Implied Share Price and its upside/downside.**

These functions help answer the following question: What is the intrinsic value of the company today based on the amount of cash it is expected to generate in the future? The future, in this case, is the "fiscal horizon", and in this project, set to five or ten years. 

### Testing the Engine with Pytest

To verify that the DCF functions are doing the job, unit tests were mandatory. This is where **Google Gemini** came in handy, through the **Antigravity CLI**. I used **Gemini Flash 3.8 (Medium)** and tasked it with writing tests for each of the functions. The tests were done with the `pytest` framework.

The unit tests range from verifying Pydantic type coercion (e.g., `string` to `int`) to checking if the output of the calculation functions is correct and checking the edge cases to handle them accordingly.

Needless to say, after running the command `uv run pytest`, all of the tests passed with flying colors.

### Ingesting 10-K Filings via EDGAR

My initial plan was to utilize a local LLM, and use a library like `sec-parser`, which would extract specific information, and store them into the dictionary for income statement, cash flow statement, and balance sheet. However, this proved to be a rather convoluted approach, and it runs into the risks of hallucination, missing context, consistency in units, etc.

**Why reinvent the wheel?** That was my immediate thought when I came across this Python package [edgartools](https://github.com/dgunning/edgartools).

This led to the creation of the `fetch_edgar.py` file. It uses the `EDGAR_IDENTITY` environment variable, which is needed to make requests to the U.S. Securities and Exchange Commission (SEC) EDGAR API because it requires all HTTP requests to have a valid contact identity in the `User-Agent` header.

Inside the file is a Python script that looks through every XBRL US GAAP tag, stored in a global-scope dictionary variable, `FIELD_ALIASES`, which contains all tags for each attribute of the financial metric classes. It has fallback tags in case the first tag does not retrieve any value.

The `get_annual_financials()` function takes in two params: `ticker` and `fiscal_year`, and that is used for retrieving the information on a company in a given fiscal year from `.get_facts()`, and `get_annual_fact()` method.

The latter is used for the helper function, `_extract_metric` below:

```python
def _extract_metric(
    facts, aliases: list[str], fiscal_year: int, default: float | None = None
) -> float | None:
    with warnings.catch_warnings():
        warnings.filterwarnings("ignore", category=UserWarning)
        for tag in aliases:
            fact = facts.get_annual_fact(tag, fiscal_year=fiscal_year)
            if fact is not None and fact.numeric_value is not None:
                return float(fact.numeric_value)

    return default
```

As was done for the DCF functions, a unit test was made for this file too, albeit in a very limited scope, and only to verify an edge case discovered during the Streamlit implementation. 

Because this library queries the actual API, I simply made a hardcoded `FinancialFact` object and verified that operating income can be derived using two approaches: extracted from the XBRL US GAAP tag `OperatingProfitLoss`, or calculated by Gross Profit - Selling, General, and Administrative (SG&A).

### Visualizing with Streamlit

Initially, my plan was to make this a CLI-driven app, using `typer`, `rich`, and any libraries necessary. But where's the fun in that?

That was when I came across Streamlit, an open-source Python framework that allows you to create interactive web applications and dashboards. This was perfect for my use case, and given that I had never used this library before, it's a whole new experience for me. Certainly beats trying to make it out of Tkinter, right? 😂

Not much to say here other than there was one thing I had to keep in mind when I was working on this: **What do I want to show?**

That question necessitated tons of other changes, purely for visuals and readability. This is also where I had to make sure the application was robust enough, with proper error handling, normalizing the currency text for readability, etc. 

This was where, once again, **Gemini** came in handy and helped to create the visuals that I had in mind, through a series of suggested tweaks and edits.

This is also hosted for free (as long as it's public) on Streamlit Cloud. You can check it out [here](https://vkumar-dcf.streamlit.app/).

Without further ado, have a look at the images below at each step, leading up to the Valuation Bridge waterfall chart!

<div style="text-align: center;">
  <img 
    src="https://res.cloudinary.com/u0n8mltb/image/upload/f_auto,q_auto/v1795021722/dcf_form_ex7wwm.png" 
    alt="DCF Calculator Form" 
    style="max-width: 100%; height: auto;"
  >
</div>

<div style="text-align: center;">
  <img 
    src="https://res.cloudinary.com/u0n8mltb/image/upload/f_auto,q_auto/v1795021723/dcf_cost_of_capital_czntve.png" 
    alt="DCF Overview & Cost of Capital" 
    style="max-width: 100%; height: auto;"
  >
</div>

<div style="text-align: center;">
  <img 
    src="https://res.cloudinary.com/u0n8mltb/image/upload/f_auto,q_auto/v1795021723/dcf_projection_sxa4i4.png" 
    alt="DCF 5-Year Unlevered Free Cash Flow Projections" 
    style="max-width: 100%; height: auto;"
  >
</div>

<div style="text-align: center;">
  <img 
    src="https://res.cloudinary.com/u0n8mltb/image/upload/f_auto,q_auto/v1795021722/dcf_summary_rec4o9.png" 
    alt="DCF Valuation Summary" 
    style="max-width: 100%; height: auto;"
  >
</div>

<div style="text-align: center;">
  <img 
    src="https://res.cloudinary.com/u0n8mltb/image/upload/f_auto,q_auto/v1795021722/dcf_waterfall_mqwqbk.png" 
    alt="DCF Valuation Bridge Chart" 
    style="max-width: 100%; height: auto;"
  >
</div>



