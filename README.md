# Instaffo case study

To measure the efficiency of candidate acquisition marketing, I propose tracking the following key metrics:

1. Total Candidate Registrations per Campaign/Channel: this shows how effectively each campaign or channel attracts new candidates.
2. Average Cost per Registration per Campaign/Channel: this measures the efficiency of marketing spend, i.e. how much it costs to acquire a single candidate.

3. Conversion Rate from Registration to Application within 3 Days: this evaluates how many registered candidates submit an application within 72 hours (based on your suggestion that approximately 25% of candidates apply within this time frame).

## Database Schema Overview
Here is the database schema for the B2C side of the marketplace:
![DB schema](Instaffo.png)
An interactive version of the schema is available on [DBDiagram.io](https://dbdiagram.io/d/Instaffo-67489e11e9daa85aca0c1d96).

## Key Tables
### Fact Tables:
`marketing_spending`: tracks daily marketing spend by campaign and channel.

`applications`: contains applications submitted by candidates.

`matches`: stores matches with the final salary.

### Dimension Tables:
`channels`, `campaigns`, `candidates`, `resumes`, `job_openings`, and `companies`.

## Example SQL Queries for Key Metrics

### 1. Total Candidate Registrations
This query calculates the total number of candidate registrations per campaign, channel, and day:
```sql
SELECT registration_date,
       aquisition_campaign_id,
       aquisition_channel_id,
       count(DISTINCT candidate_id) AS candidates
FROM candidates
GROUP BY registration_date,
         aquisition_campaign_id,
         aquisition_channel_id
ORDER BY date
```
### 2. Cost per Registration
This query computes the average cost per candidate registration by combining registration data with marketing spend:
```sql
WITH cost_registrations AS
  (SELECT c.registration_date,
          c.aquisition_campaign_id,
          c.aquisition_channel_id,
          count(DISTINCT candidate_id) AS candidates,
          sum(ms.cost_spent) AS sum_cost
   FROM candidates c
   LEFT JOIN marketing_spending ms ON c.aquisition_campaign_id = ms.campaign_id
   AND c.aquisition_channel_id = ms.channel_id
   AND c.registration_date = ms.date
   GROUP BY registration_date,
            aquisition_campaign_id,
            aquisition_channel_id)
SELECT registration_date,
       aquisition_campaign_id,
       aquisition_channel_id,
       sum_cost/candidates
FROM cost_registrations
ORDER BY registration_date
```
### 3. Conversion from Registration to Application
This query calculates the percentage of candidates who submit an application within 3 days from registration:
```sql
WITH candidates_applied AS
  (SELECT c.candidate_id,
          c.registration_date,
          MIN(a.application_date) AS min_application_date
   FROM candidates c
   LEFT JOIN resumes r ON c.candidate_id = r.candidate_id
   LEFT JOIN Applications a ON c.candidate_id = r.candidate_id
   AND ON r.resume_id = a.resume_id
   AND a.application_date BETWEEN c.registration_date AND c.registration_date + INTERVAL 3 DAY
   GROUP BY c.candidate_id,
            c.registration_date)
SELECT COUNT(*) AS total_candidates,
       COUNT(CASE
                 WHEN first_application_date IS NOT NULL THEN 1
             END) AS total_applied,
       ROUND(COUNT(CASE
                       WHEN first_application_date IS NOT NULL THEN 1
                   END) * 100 / COUNT(*), 2) AS conversion_to_application
FROM candidates_applied
```

## Why These Metrics
These metrics provide an operational view of marketing efficiency:

**Registrations**: track how many candidates are attracted by each campaign/channel.

**Cost per Registration**: evaluate the cost-effectiveness of marketing spendings.

**Conversion Rate**: understand the quality of registrations by measuring how quickly they convert to applications.

# Considerations and Caveats

1. Application Timing: not all candidates apply within 3 days. We might also consider calculation for longer periods - up to 3-6 months for a more comprehensive view of conversions over time.
2. Marketplace Efficiency: while it allows us to track B2C marketing efficiency, it doesn't include the b2b side, hence doesn't show us the full overview of marketplace efficiency.
3. Revenue Prediction: if we know the final salaries from accepted job offers, commission rate (e.g., 15%), and the conversion rates from registration to match, we can calculate the maximum allowable cost per registration to remain profitable.

This approach provides actionable insights into marketing performance and ensures alignment between candidate acquisition efforts and long-term marketplace profitability.