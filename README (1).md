# Restaurant Market Analysis: Where Should a New Restaurant Open in Bangalore? (SQL)

**Business question:** Where should a new restaurant open in Bangalore, and which cuisine and price range should it offer?

**Answer in one line:** Open an upper-mid-range (Rs 800 to 1,500 for two) European / Continental / Mediterranean restaurant in central Bangalore, around Church Street, Lavelle Road or MG Road, where customer demand per restaurant is high and competition is low.

![Demand vs competition](images/01_demand_vs_competition.png)

## Dataset
Kaggle **Zomato Bangalore Restaurants** (Himanshu Poddar): 51,717 listings with 17 columns. I used 11 of them (the review text, URL, address, phone, dishes and menu columns were dropped). The data is a 2019 snapshot.

## Tools and skills
MySQL 8 | data cleaning in SQL | joins | CTEs | window functions (`ROW_NUMBER`, `RANK`, `NTILE`, `AVG() OVER`, `SUM() OVER`) | `CASE` | string functions | `HAVING` | analytical thinking

## Project files
| File | Purpose |
|---|---|
| `sql/00_load_raw_data.sql` | The 51,717 raw rows as INSERT statements (use this if `LOAD DATA` gives file-path errors) |
| `sql/01_create_and_load.sql` | Creates the database and the all-text raw table; contains the `LOAD DATA` option |
| `sql/02_clean_data.sql` | Profiles data problems, fixes types, removes duplicates, splits cuisines |
| `sql/03_analysis_queries.sql` | 15 business queries (basic to advanced) |
| `data/zomato_trimmed.csv` | The 11-column CSV |
| `results/query_output.txt` | Full output of every query |
| `images/` | Charts used in this README |

**Run order:** `01` then `00` (or the CSV load), then `02`, then `03`.

## Data cleaning (`02_clean_data.sql`)
| Problem in raw data | Fix |
|---|---|
| Ratings stored as text: `4.1/5`, `4.1 /5` | Parsed to `DECIMAL(2,1)` |
| 2,208 ratings of `NEW`, 69 of `-`, 7,775 blank | Set to NULL and excluded from rating averages |
| Costs stored as text with commas (6,917 rows, e.g. `1,200`) | Converted to integers |
| 21 rows with no location | Dropped |
| 110 exact duplicate rows | Removed |
| Same restaurant listed once per Zomato category (51,717 rows but only 12,137 unique name + location pairs) | Kept one row per restaurant with `ROW_NUMBER()` (highest votes) |
| Multi-value `cuisines` column (up to 8 per restaurant) | Split into a `restaurant_cuisines` table using `SUBSTRING_INDEX` and a numbers table |

**Result:** 12,126 unique restaurants across 93 locations, 28,031 restaurant-cuisine rows, and a `price_range` column (Budget up to 400, Mid 401 to 800, Upper-Mid 801 to 1,500, Premium above 1,500).

## Key findings
**Market overview**
- 12,126 restaurants, average rating **3.63** (9,231 rated), average cost for two **Rs 490**.
- 52.4% accept online orders, only 7.8% offer table booking.
- Rating is tough to earn: 43.9% of rated restaurants sit at 3.5 to 3.9, and only **2.2%** reach 4.5 or above.

**Competition**
- Whitefield (821), BTM (699), Electronic City (694), HSR (683) and Marathahalli (657) have the most restaurants.
- Whitefield has more restaurants than any other area but only 176 votes per restaurant on average, so it is crowded and customer attention is thin.
- Indiranagar has fewer restaurants (525) but the highest total votes (258,571).

**Price range**
| Price range | Restaurants | Avg rating | Avg votes |
|---|---|---|---|
| Budget (up to 400) | 7,137 | 3.56 | 58 |
| Mid (401 to 800) | 3,717 | 3.62 | 223 |
| Upper-Mid (801 to 1,500) | 927 | 3.94 | 782 |
| Premium (1,500+) | 288 | 4.16 | 1,054 |

![Price range](images/02_price_range.png)

**Cuisine**
- Most common: North Indian (4,925), Chinese (3,524), South Indian (2,328). They are crowded and average only 3.55 to 3.58.
- Best performers (100+ restaurants): **European 4.20**, Asian 4.07, BBQ 4.01, American 3.97.
- Cuisine gap (rating above market average, 50 to 600 restaurants, ranked by demand): Mediterranean, European, Steak, BBQ and American.

![Cuisine gap](images/03_cuisine_gap.png)

**Opportunity locations** (top quartile for votes per restaurant, bottom half for number of restaurants, rating at or above market average)
| Location | Restaurants | Avg votes | Avg rating | Avg cost for two |
|---|---|---|---|---|
| Church Street | 56 | 876 | 3.90 | 763 |
| Lavelle Road | 56 | 778 | 4.08 | 1,323 |
| MG Road | 94 | 387 | 3.83 | 1,081 |
| Residency Road | 81 | 361 | 3.84 | 943 |
| Cunningham Road | 49 | 292 | 3.74 | 720 |

Inside these locations, upper-mid restaurants (801 to 1,500) average a 3.99 rating and 956 votes, versus 3.64 and 107 votes for budget restaurants.

## Recommendation
1. **Location:** Church Street or Lavelle Road first, with MG Road as the larger-market alternative.
2. **Concept:** casual dining with a European / Continental / Mediterranean menu.
3. **Price:** about Rs 800 to 1,500 for two. Premium restaurants rate highest, but there are few of them, so this band balances quality and demand.
4. **Operations:** offer table booking and online ordering. Restaurants with table booking average ratings above 4.0 and about 1,000 votes.
5. **Avoid:** budget North Indian / Chinese in crowded areas such as Whitefield, BTM and Electronic City, where competition is highest and ratings and votes are lowest.

## Limitations
- The data is a **2019 snapshot**; the market has changed since.
- **Votes are a proxy for demand**, not sales. There is no data on rent, footfall or profit, and central locations usually have higher rents.
- Findings show association, not cause. For example, table booking and high ratings both go with higher-priced restaurants.
- Restaurants with the same name in the same locality are treated as one restaurant.
- A few restaurant names in the source file contain garbled characters (a text-encoding issue in the original data).
