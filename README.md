# NJ Transit Operational & Delay Report 2019

An interactive Looker Studio dashboard that shows how New Jersey Transit trains ran in 2019: how busy each route was, and where and when delays happened.

**[Open the live dashboard](https://datastudio.google.com/reporting/5711bec6-a910-47fc-bdb7-2dae91a9799f/page/VnHxF)**

![Operational page](images/dashboard-operational.png)

![Delay report page](images/dashboard-delay-report.png)

> The dashboard labels are in Indonesian. Quick guide: *Volume Perjalanan* = trip volume, *Jumlah Rute* = number of routes, *Jumlah Kereta Aktif* = number of train numbers, *Persentase Ketepatan Waktu* = on-time percentage, *Rata-Rata Keterlambatan* = average delay, *Keterlambatan* = delay.

## Business Problem

NJ Transit runs a large rail network with millions of train records every year. When delays are not tracked well, it is hard to see which routes, stations, and hours hurt passengers the most.

The goal of this project is a monitoring dashboard with two views:

- **Operational:** trip volume, usage by hour, and number of stops per route.
- **Delay Report:** on-time rate, the stations and trains most affected by delays, and how delays change through the day.

## Dataset

- **Source:** [NJ Transit + Amtrak (NEC) Performance](https://www.kaggle.com/datasets/pranavbadami/nj-transit-amtrak-nec-performance) on Kaggle.
- **Coverage:** North East Corridor rail operations from 2018 to 2020.
- **Used here:** NJ Transit data for January to December 2019, about 2.5 million rows.
- The raw files are not in this repo because of their size. You can download them from Kaggle.

## Tools

- Google BigQuery, for storing and querying the data
- Google Looker Studio, for the dashboard
- Calculated fields (`CASE WHEN`) for the on-time logic

## Process

**1. Data preparation**
The monthly CSV files (2019_01 to 2019_12) were uploaded to BigQuery, then connected to Looker Studio.

**2. Calculated fields**
A trip counts as on time when it is 6 minutes late or less, a common tolerance in rail transport.

```sql
-- on_time_performance
CASE WHEN Final_Delay_Minutes <= 6 THEN 1 ELSE 0 END

-- delay_category
CASE
  WHEN Final_Delay_Minutes <= 6  THEN 'Tepat Waktu'             -- On time
  WHEN Final_Delay_Minutes <= 15 THEN 'Keterlambatan Ringan'    -- Minor delay
  WHEN Final_Delay_Minutes <= 30 THEN 'Keterlambatan Menengah'  -- Moderate delay
  ELSE 'Keterlambatan Parah'                                    -- Severe delay
END
```

**3. Dashboard design**
Two pages (Operational and Delay Report), with filters for date range, route, and month.

**4. Quality check**
During QC, the "On-Time Percentage" card and the delay donut chart were about 40 percentage points apart, even though both should use the same category logic. The cause was the wrong metric selected in the donut chart settings. After the fix, both visuals show the same numbers.

## Key Findings

### Operations

- 2.57 million trip records in 2019, across 11 routes and 1,176 train numbers.
- Usage has two daily peaks, around 07:00 to 08:00 and 17:00 to 18:00. This is a typical commuter pattern. The quietest hour is around 03:00.
- The busiest routes are Northeast Corridor (433,628), Morristown Line (417,361), and No Jersey Coast (399,149). Together they make up almost half of all volume.
- Montclair-Boonton and No Jersey Coast have the most stops (28 each).

### Delays

- **81.9%** of trips were on time, with an average delay of **4.3 minutes**.
- 13.9% of trips had a minor delay (6 to 15 minutes). About 4.2% had a moderate or severe delay (over 15 minutes).
- The most affected stations are Secaucus Upper Lvl, Newark Penn Station, and New York Penn Station.
- Atlantic City Line stands out by route: its average delay is **16.3 minutes**, more than three times the next routes (No Jersey Coast 4.62, Morristown Line 4.39).
- Train 9225 and train 9227 (Montclair-Boonton) have the highest average delays: 150.04 and 114.58 minutes. Train 4689 (Atlantic City Line) follows with 91.28 minutes.
- By hour, delays are highest from 00:00 to 02:00 and again in the evening, around 19:00. The morning rush does not show a clear spike.

## Recommendations

1. **Look into trains 9225 and 9227.** Their delays are far above everything else. First check how many trips each one made. A few extreme days would explain it. If it happens often, the service itself needs a review.
2. **Review the Atlantic City Line.** Its average delay is the highest of all routes by a wide margin, so it is a route-level issue and not just a few bad trains.
3. **Focus on the hub stations.** Secaucus Upper Lvl, Newark Penn Station, and New York Penn Station are where delays show up most. Check platform and track capacity there.
4. **Add buffer time in the evening rush.** Delays rise from 18:00 and peak around 19:00. Extra schedule time or extra capacity in this window would help.
5. **Check the late-night delays.** Few trains run between 00:00 and 02:00, so confirm whether this is a real pattern or a small-sample effect before acting on it.

## Limitations

- Averages can be pulled up by a few extreme delays. Showing trip counts and the median next to each average would make the rankings more reliable.
- The dashboard shows where and when delays happen, not why. There is no weather, track, or incident data in this dataset.
- It covers one year only, so it cannot show long-term trends.

## What I Would Add Next

- Median delay and trip counts beside every average
- Delay trend by month and by day of week
- Axis labels and data labels on the hourly and station charts

## Repo Structure

```
nj-transit-delay-dashboard/
├── README.md
└── images/
    ├── dashboard-operational.png
    └── dashboard-delay-report.png
```

## Author

Alif Al Farizy, aspiring data analyst with a GIS background.
Email: aalifalfarizyy@gmail.com
