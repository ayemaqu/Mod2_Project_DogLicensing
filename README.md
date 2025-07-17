# 🐶 NYC Dog Licensing: Exploring Age & Breed Trends Across the City
##### By: Ayema & Debo

## 📌 Project Overview
- This project explores trends in NYC dog licensing data, with a focus on how a dog’s age and breed affect licensing behaviors across the city.
- The dataset was provided by the Animal Care Centers of NYC (ACC), who manage dog licensing and pet population initiatives in collaboration with the Department of Health.
- The goal was to help ACC understand current licensing patterns and uncover areas where improvements can be made, particularly around the adoption and licensing of older dogs and underrepresented breeds.

## Dataset Background
- The data comes from the NYC Department of Health’s Dog Licensing System, where owners are legally required to register their dogs. The dataset includes:
  - Both new licenses and renewals
  - One row per active license per year
  - Some dogs appear more than once due to renewals
- This structure allows us to analyze annual licensing trends, but does not represent unique dogs.

## Stakeholder
- **Animal Care Centers of NYC (ACC)**
  - ACC helps manage pet adoptions, licensing compliance, and dog population insights across the five boroughs. They rely on licensing data to support outreach, shelter planning, and policy development around responsible pet ownership.

## Business Question
- How does a dog’s age and breed affect licensing trends across NYC?

## EDA + Cleaning Process
#### Duplicate Handling
- Removed fully duplicated rows with df.drop_duplicates()
  - Case: "Abigail" appeared multiple times with identical license info — confirmed true duplicate

#### Year Extraction Validation
- Created IssueYear and ExpireYear from license date columns
- Caught data issues where `ExtractYear` did not match the actual license year range — these rows were removed

#### Missing Values
- Dropped rows missing key fields: `AnimalBirthYear`, `AnimalGender`, or `LicenseExpiredDate`
  - Impact: Only ~130 rows removed

#### Dog Age + Grouping
- Calculated `ApproxDogAge` = `IssueYear` - `AnimalBirthYear`
- Created `DogAgeGroup` bins:
  - `0–1 (Puppy), 2–3 (Young), 4–6 (Adult), 7–10 (Senior), 10+ (Elderly)`
 
#### Column Cleanup
- Removed unused or duplicate fields (e.g. `LicenseIssuedYear`)

#### ZIP Code Cleanup
- ZIP codes were critical for our geographic visualizations (like maps by borough or ZIP), so we needed to make sure the data was clean and accurate. We handled this process in two key steps:
  - **Format & Filter by Length**
    - We first ensured that the `ZipCode` column was in string format and that only entries with 5-digit codes remained.
      - Why: Some entries had ZIPs like `0`, `100`, or were blank. These aren’t valid for NYC mapping.
        - `df['ZipCode'] = df['ZipCode'].astype(str)`
        - `df = df[df['ZipCode'].str.len() == `5]
      - This cleaned up obvious formatting issues but didn’t guarantee that the ZIPs were real NYC ZIP codes.
  - **Cross-Check with Official NYC ZIP List**
    - To validate ZIP codes more thoroughly, we used an external file: `zipcode.csv`, which contained a verified list of valid NYC ZIP codes.
    - We compared every ZIP code in our dataset against this official list to filter out any invalid, outdated, or non-NYC ZIPs.
    - We also cleaned up formatting quirks from Excel exports, such as `.0` suffixes that appeared when numbers were read as floats.\
    - This was important because:
      1. Ensured that all geographic visualizations (especially our ZIP code map) were accurate and didn’t mislead stakeholders with bad data.
      2. Prevented invalid ZIPs from skewing analysis, for example, a license record with `00000` would distort mapping entirely.
      3. Made it possible to link each license to a real NYC neighborhood, helping stakeholders identify underserved or overrepresented areas.
     
#### Breed Standardization
- The `BreedName` column in the dataset had significant inconsistencies. For example, the same breed appeared under many slightly different names such as "Poodle Mix", "Poodle X", "Poodle Crossbreed", or even informal variants like "Poodle Dog" or "Poodle-type". These inconsistencies posed a problem for accurate aggregation and trend analysis, especially when grouping by breed.
- To address this, we implemented a two-step cleaning process:
  - **Regex-Based Cleanup with re**
    - We used regular expressions to standardize mix indicators like "Mix", "X", and "Crossbreed"
    - This ensured that similar breed types were grouped under a single standard label such as "Poodle Mix".
  - **Fuzzy Matching with thefuzz**
    - Even after regex cleanup, breed names varied too widely for manual standardization. So we used the fuzz library from thefuzz package to compare each unique breed name to a curated reference list of common dog breeds and their standardized labels. We mapped any breed that matched with a similarity score ≥ 85 to its closest reference value.
  - This method allowed us to automatically consolidate variants like:
    - "Yorkie Poodle" → "Yorkshire Terrier Mix"
    - "Labrador Retriever" and "Lab Ret" → "Labrador Retriever"
  - **Handling Unknown or Placeholder Breeds**
    - We also identified and replaced invalid or placeholder breed names such as:
    - `"Unknown", "NoName", "Mixbreed", "0"`, etc. These were flagged with a boolean column IsBadName to help with filtering or analysis decisions later in Tableau.

- _This allowed us to have_: 
  1. Reduced noise in the data
  2. Made breed-based grouping accurate
  3. Helped us visualize breed trends more reliably in Tableau

#### Feature Engineering
- To enhance our analysis and support more insightful visualizations, we engineered several new columns from the original dataset:
  1. `CleanedBreedName`: A standardized version of breed names using fuzzy matching and manual corrections to address inconsistencies, typos, and duplicate labels.
  2. `ApproxDogAge`: Calculated by subtracting AnimalBirthYear from LicenseIssuedYear, giving us a rough estimate of the dog’s age at the time of licensing.
  2. `DogAgeGroup`: Created by binning ApproxDogAge into meaningful categories (e.g., "`0–1 (Puppy)", "2–3 (Young)`", etc.), allowing us to group and compare age ranges.
  3. `IssueYear` and `ExpireYear`: Extracted from license date fields to simplify time-based filtering and support time series visualizations.
  4. `IsBadName`: A Boolean column identifying missing, placeholder, or junk names (like “NoName” or “WHATSERNAME”), used to flag low-quality entries for filtering or review.

## Visualizations & Insights
🔗 [View Dashboard on Tableau](https://public.tableau.com/app/profile/debo.odutola/viz/Book1_17514704095920/Dashboard12?publish=yes)

#### Histogram: Dog Age Distribution at Licensing
- Most dogs are licensed under age 3
- Very few senior/elderly dogs are licensed
- Suggests that older dogs are either less likely to be adopted or renewed
  - _Action:_ ACC can target awareness campaigns in areas with low senior dog licensing

#### Line Chart: Licensing Trends Over Time by Age Group
- Puppy licenses peak in 2021
- Elderly dog licenses are consistently lowest
- Senior dog licensing is declining year over year
  -_ Action:_ Renewals and long-term ownership may need more city outreach or incentives

#### Tree Map: Breed by Age Group
- Shows which breeds are mostly licensed as filtered by age group, such as in puppies, the most popular breed is American Pit Bull terrier mix
- Identifies breeds like Shih Tzus and Airedale Terriers that appear in older age groups
  - _Action:_ Breed-specific adoption or renewal strategies could be developed

#### Filled Map: Licensing by ZIP Code
- Visualizes where younger vs older dogs are licensed
- Highlights geographic patterns by breed and age group
  - _Action:_ Helps ACC identify neighborhoods for localized campaigns or events


## Based on our analysis, we recommend:
- Encouraging senior dog licensing:
  - Offer discounts, reminders, or partner with vets for older pet outreach
- Target ZIP codes with low senior dog license counts: 
  - These areas could benefit from education or adoption events
- Standardize breed input for better analysis
  - The licensing system could enforce dropdown-based input instead of free text
- Track renewal rates by breed and age
  - This would help ACC understand which populations are staying licensed over time

## Project Reflection
1. What’s the most important insight your dashboard reveals?
- The most impactful insight from our dashboard is that most dogs are licensed when they are young, particularly within the puppy (0–1 years) and young (2–3 years) age groups. In contrast, senior and elderly dogs are significantly underrepresented, especially when it comes to renewals over time. This pattern highlights a potential gap in outreach and awareness for older dogs, both in terms of adoption and license renewal.
- This insight opens the door for targeted education campaigns, renewed focus on senior dog advocacy, and potential policy interventions to encourage long-term licensing behavior and reduce drop-off as dogs age.

2. With limited time, how did you decide what to prioritize?
- At the start of the project, it was easy to feel overwhelmed. There were so many directions we could go. We had access to a rich dataset with dozens of columns, years of records, and endless possibilities. But from the very beginning, we reminded ourselves of one thing:
  - 👉 “Stick to the business question.”
- _Our stakeholder asked:_
  - “How does a dog’s age and breed affect licensing trends across NYC?”
  - So we treated this question like a compass. Every time we felt tempted to explore something unrelated or scope out, we brought ourselves back to that core. We broke the big question into smaller ones that helped guide our analysis:
    1. Are older or younger dogs more likely to be licensed?
    2. How has the licensing of older vs. younger dogs changed over the years?
    3. Which breeds are most commonly licensed at older ages?
    4. Are there regional differences in breed or age group licensing (e.g., by ZIP code)?
    5. Are certain breeds underrepresented or possibly declining?

- These became our building blocks. From there, we asked ourselves:
  - Which of these supporting questions actually give the stakeholder insight into behavior and trends?
  - What visuals will speak clearly in five minutes, without overwhelming the audience?
- That mindset helped us filter out noise, like redundant columns or overcomplicated features, and stay focused on what truly matters. We didn’t just prioritize visuals; we prioritized value. Each visualization was selected not just to show data, but to tell part of a story, one that could help Animal Care Centers of NYC make informed decisions about outreach, renewal campaigns, and breed-specific policies.

In the end, it wasn’t about analyzing everything — it was about answering the right question, with purpose.


## Folder Structure 
```text
.
├── data/
│   ├── raw/
         ├── NYC_Dog_Licensing_Dataset_.csv
│   └── cleaned/
         ├── final_dataset.csv
         ├── zipcode.csv
├── excel/
│   ├── final_dataset.csv
├── notebooks/
│   ├── cleaning_and_eda.ipynb
|
└── READE.md

```

## 👥 Connect With Us
We’re passionate about data storytelling and using analysis to drive actionable insights. Feel free to connect with us on **LinkedIn**!
- [Debo Odutola](https://www.linkedin.com/in/deboodutola/)
- [Ayema Qureshi](https://www.linkedin.com/in/ayema-qureshi-901287187/)



