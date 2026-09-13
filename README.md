# IPL-Dashboard

# IPL Analytics Dashboard (2008–2025)

**An end-to-end Power BI case study** : from raw ball-by-ball data to an interactive tournament dashboard covering 18 seasons (2008-2025) of the Indian Premier League.

<img width="1000" height="560.3" alt="ipl_analytics_dashbaord" src="https://github.com/user-attachments/assets/ae057e32-e1fa-4d73-aa2f-9985be6e0025" />
---

## 1. Business Context

Fragmented IPL data limits the ability to derive timely, consistent insights on team and player performance. The objective was to **build a unified, season-aware analytics solution** to enable **performance benchmarking, identify standout players, and support data-driven evaluation of teams and seasons**.

## 2. Data Architecture & Flow


<img width="800" height="282.2" alt="image" src="https://github.com/user-attachments/assets/83c59bda-4879-4b53-84a1-3c7782db1ebe" />



**Grain of each source:**

| Source | Grain | Rows | Role in model |
|---|---|---|---|
| `ball_by_ball_data.csv` | One row per delivery bowled | ~278,000 | Fact table |
| `ipl_matches_data.csv` | One row per match | ~1,169 | Dimension (match context) |
| `players-data-updated.csv` | One row per player | ~772 | Dimension (player attributes) |
| `teams_data.csv` | One row per franchise | 16 | Dimension (team branding/lookup) |

## 3. Data Modeling

Built as a **star schema** in Power BI:

<img width="800" height="358.2" alt="image" src="https://github.com/user-attachments/assets/88b3e596-6792-4e87-9e13-2fe36078653c" />


## 4. Data Cleaning & Transformation (Power Query)

- Standardized inconsistent team name variants across seasons into a single canonical name per franchise
- Handled `NULL` values across dismissal fields (`player_out`, `wicket_kind`, `fielders_involved`). Most deliveries have no wicket, so nulls needed explicit handling rather than filtering rows out
- Converted extras logic (wide, no-ball, leg-bye, bye, penalty) into a single derived **"legal delivery"** flag, since over/ball counts depend on excluding wides and no-balls
- Cast `match_date` to a proper date type and derived `season` as a clean numeric field for slicing
- De-duplicated player metadata where the same player appears under slightly different name spellings across seasons

## 5. KPI Development (DAX)

Every card on the dashboard is a season-dynamic DAX measure, not a static value. Representative logic:

```DAX
Total Sixes =
CALCULATE(
    COUNTROWS('Deliveries'),
    'Deliveries'[batter_runs] = 6
)

Orange Cap Holder =
CALCULATE(
    TOPN(1, VALUES('Deliveries'[batter]), [Total Runs], DESC)
)

Total Points =
VAR Wins = CALCULATE(COUNTROWS('Matches'), 'Matches'[match_winner] = SELECTEDVALUE('Teams'[team_name]))
VAR Ties = CALCULATE(COUNTROWS('Matches'), 'Matches'[result] = "tie")
RETURN (Wins * 2) + (Ties * 1)
```

This pattern of rank-based `TOPN`/`RANKX` measures for cap holders and max-hitters, `CALCULATE` with filter context for count-based KPIs is what lets a single season slicer drive every card, table, and player photo on the page simultaneously.

## 6. Dashboard Design & UX

Design decisions were made around **one-glance readability**:

- **Season slicer isolated top-right** is the single control that drives the entire page, kept visually separate from KPI cards so it reads as "the dial," not another metric
- **Player cards with photos** (Orange Cap, Purple Cap, Max Six-hitter, Max Four-hitter) rather than plain text — recognition is faster with a face and jersey than a name in a table
- **Points table sorted and color-banded** by total points so season standings are scannable without reading every row
- **KPI strip at the top** (matches, teams, sixes, fours, stadiums, centuries, half-centuries, wickets, five-wicket hauls) gives tournament-wide context before drilling into season specifics

## 7. Key Insights Surfaced (2025 season, as example)

- Royal Challengers Bangalore won the season; Punjab Kings were runners-up
- B Sai Sudharsan led both the Orange Cap race (759 runs) and max four-hitting (88 fours) which is a double signal of consistent, boundary-heavy scoring
- M Prasidh Krishna's 25 wickets led the Purple Cap race, both from Gujarat Titans which suggests a team built around a small core of standout individual performances
- The points table shows a tight top of the table (Punjab Kings and RCB both on 19 points) versus a clear bottom tier (Chennai Super Kings, 8 points) that tells it was a season with a compressed playoff race and a decisive relegation-zone gap


## 8. Tech Stack

- **Power BI** — data modeling, DAX, report design
- **Power Query (M)** — ETL: cleaning, type-casting, name-mapping joins
- **DAX** — dynamic KPI and ranking measures


## Author

**Sakti Mohapatra**
mohapatrasakti04@gmail.com
