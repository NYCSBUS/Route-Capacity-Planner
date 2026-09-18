# Route Capacity Planner

A single-file decision tool that shows, week by week and day by day, how many bus routes can be serviced given the vehicles, parking, and driver/attendant staff available — and exactly how many vehicles to buy and how many staff to clear through the pipeline to close any gap.

Everything lives in `index.html`. No build step, no server, no external libraries.

## Running it

**Locally:** double-click `index.html` (or open it from any browser). All data is saved in the browser's local storage, so your inputs persist between visits on the same machine.

**On GitHub Pages:**

1. Push `index.html` (and this README) to a GitHub repository.
2. In the repo go to **Settings → Pages**, set the source to the branch containing `index.html` (root folder), and save.
3. The planner will be available at `https://<your-user>.github.io/<repo>/`.

Because inputs are stored per browser, use **Versions & share** (see below) or **Export JSON / Import JSON** to move a scenario between people or machines.

## Tabs and who enters what

| Tab | Owner | What's there |
|---|---|---|
| **Outputs** | All execs | Summary cards, weekly chart, week → date table, and a read-only summary of every input, assumption and rule behind the numbers |
| **CFFO** (Chief Fleet Officer) | Vehicles & parking | Parking locations and max vehicles at each (with a "Type C allowed" flag), vehicles available for service by type, spare factor % per type, **Pipeline 1** – vehicles currently out of service by status (Out of Service – Long Term 12 wk, Out of Service 6 wk, For Vendor – Out of Service 5 wk, At Vendor 4 wk; return times editable) with the as-of date, plus recurring exits per week per type (unscheduled, back after 6 wk; scheduled inspections, back after 2 wk), and **Pipeline 2** – new vehicles entering service each week |
| **CHRO** (Chief HR Officer) | Drivers & attendants | Active & eligible drivers and attendants, spare factor %, average exits per week, and a weekly pipeline of drivers/attendants cleared |
| **CSDO** | Route schedule | Routes starting on each date, by vehicle type needed |
| **Admin** | Rules | Vehicle → route substitution matrix (tick which vehicle types may run which route types), whether attendants are required, whether parking capacity is enforced, and whether demand accumulates |

Hover any column header (dotted underline) for a plain-language explanation of that number. Non-zero values in the input tables are highlighted in yellow so entered figures stand out.

### Vehicle types

- Type C (Big Bus)
- Type A – no wheelchair
- Type A – wheelchair
- Minivan

### Pipeline semantics

- Weeks start on **Monday**.
- Pipeline entries are **added that week** (effective from Monday) and accumulate.
- Exits are an **average per week**, applied every week from the first week shown; fractional averages accumulate and round down (e.g. 2.5/week → 2, 5, 7, 10 …).
- **Staff exits are permanent** (average per week). **Vehicle exits are temporary.** Pipeline 1 holds (a) the vehicles out of service today by status – each status returns to the fleet on the Monday `return weeks` after the as-of date (returns dated before the first week shown count as already back) – and (b) recurring exits of existing vehicles: a rate per week per type for unscheduled events (back after 6 weeks) and scheduled inspections (back after 2 weeks), both editable. Vehicles out through recurring exits plateau at Σ rate × weeks. "Available for Service" should exclude vehicles already out of service.
- The computed columns next to the pipeline inputs show the resulting fleet / staff for each week.

## Rules applied

Rules marked *(Admin)* can be toggled on the Admin tab; the Outputs tab always lists the rules currently in effect.

1. **Usable count** = floor(count × (1 − spare %)).
2. **Crews** *(Admin)*: by default every route needs one driver and one attendant, so usable crews = min(usable drivers, usable attendants). With the attendant rule off, only drivers limit staffing.
3. **Fleet each week** = available for service + new vehicles to date (Pipeline 2) + out-of-service vehicles returned to date (Pipeline 1) − vehicles currently out through recurring exits.
4. **Parking cap** *(Admin)*: Type C buses must fit in Type C-allowed locations; Type A and Minivans fill remaining space anywhere. If the fleet exceeds parking, the excess is excluded (Type A/Minivan trimmed proportionally) and flagged.
5. **Vehicle → route substitution** *(Admin matrix)*. Defaults:
   - Type A wheelchair buses may cover Type A non-wheelchair routes.
   - Type C buses may cover Type A non-wheelchair routes.
   - Wheelchair routes need Type A wheelchair buses.
   - Minivan routes need minivans; minivans do nothing else.
   - A vehicle always covers its own route type, and each type serves its own routes first; spare vehicles then cover other allowed route types. The assignment is solved as a max-flow so the total number of routes covered is the maximum the matrix allows.
6. **Demand is cumulative** *(Admin)*: once a route starts, it needs a vehicle and crew every day thereafter.
7. **Serviceable routes** = min(vehicle-feasible routes, usable crews).
8. **To close the gap** grosses the shortfall up by the spare factor, so it reads as head-count / vehicles to add. Vehicle shortfalls are reported by route type; where substitutions are allowed, that figure is the minimum to add of *any* eligible type.

## Reading the results

- **Summary cards:** routes needed, serviceable, shortfall, first shortfall date, and staff/vehicles to add by the horizon date.
- **Chart (Bars):** for each week, three bars — routes needed (stacked by type), usable vehicles (stacked by type), and staff pairs — with a green line at the serviceable count and the shortfall and its cause underneath. Hover any bar for the exact breakdown.
- **Chart (Lines):** the same series as lines with markers on a y-axis zoomed to the data (not anchored at 0) so the gaps between need, vehicles and staff are visible. Click a marker to explode it into its categories (vehicle types, or drivers/attendants); click the chart background to collapse. Toggle Bars/Lines at the right of the legend.
- **Vehicles returning / arriving / out column:** per week or date, the net fleet movement with *Old (returning)* (Pipeline 1 status returns plus recurring-exit returns), *New* (Pipeline 2) and *Out (recurring)* (existing vehicles newly out of service), each by vehicle type.
- **Subcontracted routes:** tick *Subcontracted* on a row in the CSDO schedule (e.g. the 11 MW Type A non-WC routes starting 9/10) to keep it visible in the schedule but exclude it from every calculation. Excluded routes are greyed in the schedule and noted in the Outputs cards and table.
- **Table:** one row per week; click a row (or "Expand all weeks") to see individual dates. Columns show new routes, routes needed, usable vehicles and staff pairs (each with type/role splits), serviceable, vehicle shortfall (which types are short), staff shortfall, and what CFFO and CHRO each need to add.
- **Show results from / Horizon end** set the visible date range; earlier route starts still count toward demand.

## Sharing and versions

Click **Versions & share** in the header.

### Named versions on GitHub (short, listed links)

Saves the current inputs as `versions/<name>.json` in this repository and keeps a list in `versions/index.json`. Each version gets a short link of the form

```
https://<your-user>.github.io/<repo>/?version=<name>
```

Opening the link loads that version for anyone — no login needed once GitHub Pages has published the file (usually 1–2 minutes after saving). A banner shows which version is being viewed; edits stay in the viewer's browser until they save a new version.

One-time setup (the person saving versions needs this; viewers don't):

1. Open **Versions & share → GitHub settings**. On GitHub Pages the owner and repository are pre-filled; set the branch if it isn't `main`.
2. Create a **fine-grained personal access token**: GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens → *Generate new token*. Limit it to this repository and grant **Contents: Read and write**.
3. Paste the token and click **Save settings**. It is stored only in that browser and is never written into versions, exports, or links.

Saving an existing name asks before overwriting. The **Saved versions** list shows every version with *Copy link* and *Load*.

### Quick share link (no setup)

**Copy quick share link** packs the current inputs into the URL itself (`index.html#s=…`, about 3 KB). Anyone opening it sees exactly those inputs. It isn't listed anywhere, so it suits one-off sharing; use named versions for anything you want to refer back to.

### Other

- **Export PDF** prints the Outputs tab (summary, chart, weekly plan, inputs & assumptions, rules in effect) via the browser's print dialog — choose *Save as PDF*. Expand weeks first if you want dates in the report. `index.html?report` opens the report view directly.
- **Export JSON / Import JSON** saves or loads the full scenario as a file.
- **Reset to defaults** restores the built-in sample data.

## Notes

- The built-in defaults for parking capacity, staff counts, and fleet size are placeholders; replace them with real figures.
- Data is stored in the browser under the key `routeCapacityState.v5`. Clearing site data resets the planner.
