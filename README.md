# LinkedIn Automation

LinkedIn Automation is an n8n workflow designed to streamline lead generation and outreach on LinkedIn. It automates prospect discovery, evaluates leads for product fit, generates personalized connection messages, and logs results in Google Sheets.

## Workflow Overview

The workflow consists of the following key stages:

1. **Form Trigger**

   * **Node:** `On form submission`
   * **Function:** Captures user input for LinkedIn outreach, including:

     * Company Type
     * Role
     * Company Size
     * Country

2. **Generate Apollo Search URL**

   * **Node:** `Message a model`
   * **Function:** Uses OpenAI (GPT-5) to convert form input into a structured Apollo Search URL.
   * **Input Example:**

     ```
     I want the Founder of all the Creative Agency in India with company size between 1-100
     ```
   * **Output Example:**

     ```json
     {
       "serachURL": "https://app.apollo.io/#/people?personLocations[]=India&sortAscending=false&sortByField=%5Bnone%5D&personTitles[]=founder&page=1&qKeywords=creative%20agency&organizationNumEmployeesRanges []=1%2C100"
     }
     ```

3. **Run Lead Scraper**

   * **Node:** `Run an Actor`
   * **Function:** Uses Apify’s “Leads Scraper” actor to scrape leads from the generated Apollo Search URL.
   * **Settings:**

     * `contact_email_exclude_catch_all: true`
     * `contact_email_status_v2_verfied: true`
     * `include_email: true`
     * `max_result: 5`

4. **Retrieve Scraped Leads**

   * **Node:** `Get dataset items`
   * **Function:** Pulls lead data from Apify datasets.

5. **Split Leads for Processing**

   * **Node:** `Loop Over Items`
   * **Function:** Processes each lead individually for evaluation and message generation.

6. **Lead Fit Evaluation**

   * **Node:** `Profile Matching with Company's Profile`
   * **Function:** Uses OpenAI (GPT-4.1-mini) to determine whether a lead is a good match for D’SIAR TECH’s offerings (e.g., Lawyer Chatbot).
   * **Output Format:**

     ```json
     {
       "fit": "Fit" | "Not a Fit"
     }
     ```

7. **Parse Evaluation Result**

   * **Node:** `Code in JavaScript`
   * **Function:** Converts the JSON string output from OpenAI into a usable JSON object.

8. **Conditional Filtering**

   * **Node:** `If`
   * **Function:** Checks if lead is a fit (`fit === "Fit"`) before proceeding to message generation.

9. **Generate LinkedIn Connection Message**

   * **Node:** `Message a model1`
   * **Function:** Uses OpenAI (GPT-4.1-mini) to create a professional, concise, and personalized LinkedIn connection message for each fit lead.
   * **Example Output:**

     ```
     Hello Pascal, I’m impressed by your expertise as a Rechtsanwalt and would love to connect to explore potential opportunities for collaboration with D’SIAR TECH. Looking forward to connecting!
     ```

10. **Append Results to Google Sheets**

    * **Node:** `Append row in sheet`
    * **Function:** Logs scraped leads and generated messages into Google Sheets for tracking.

11. **LinkedIn Connection Request via Phantombuster**

    * **Node:** `HTTP Request`
    * **Function:** performs connection request task through Phantombuster agents.

12. **Delay Between Actions**

    * **Node:** `Wait`
    * **Function:** Adds a delay to avoid rate limits or API overloads.
   
13. **LinkedIn Calendly Link shaking Automation via Phantombuster**

    * **Node:** `HTTP Request1`
    * **Function:** pass the calendly once our connection has been accepted.

14. **Conditional Checks**

    * **Node:** `If1`
    * **Function:** calendly link shared connections data are collected for futher saving (future addition).
