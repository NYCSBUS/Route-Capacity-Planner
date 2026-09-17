# Route Capacity Planner

A single-file decision tool that shows, week by week and day by day, how many bus routes can be serviced given the vehicles, parking, and driver/attendant staff available — and exactly how many vehicles to buy and how many staff to clear through the pipeline to close any gap.

Everything lives in `index.html`. No build step, no server, no external libraries.

## Running it

**Locally:** double-click `index.html` (or open it from any browser). All data is saved in the browser's local storage, so your inputs persist between visits on the same machine.

**On GitHub Pages:**

1. Push `index.html` (and this README) to a GitHub repository.
2. In the repo go to **Settings → Pages**, set the source to the branch containing `index.html` (root folder), and save.
3. The planner will be available at `https://<your-user>.github.io/<repo>/`.

Because inputs are stored per browser, use **Export JSON / Import JSON** to move a scenario between people or machines.

## Who enters what

The inputs are grouped by owner:

| Box | Owner | Inputs |
|---|---|---|
| **CFFO** (Chief Fleet Officer) | Vehicles & parking | Parking locations and max vehicles at each (with a "Type C allowed" flag), vehicles available for service by type, spare factor % per type, average vehicles exiting per week, and a weekly pipeline of vehicles entering service |
| **CHRO** (Chief HR Officer) | Drivers & attendants | Active & eligible drivers and attendants, spare factor %, average exits per week, and a weekly pipeline of drivers/attendants cleared |
| Shared | Operations | Route schedule: routes starting on each date, by vehicle type |

Non-zero values in the input tables are highlighted in yellow so entered figures stand out.

### Vehicle types

- Type C (Big Bus)
- Type A – no wheelchair
- Type A – wheelchair
- Minivan

### Pipeline semantics

- Weeks start on **Monday**.
- Pipeline entries are **added that week** (effective from Monday) and accumulate.
- Exits are an **average per week**, applied every week from the first week shown; fractional averages accumulate and round down (e.g. 2.5/week → 2, 5, 7, 10 …).
- The computed columns next to the pipeline inputs show the resulting fleet / staff for each week.

## Rules applied

1. **Usable count** = floor(count × (1 − spare %)). Usable driver/attendant pairs = min(usable drivers, usable attendants). Every route needs one pair.
2. **Fleet each week** = available for service + entries to date − exits to date.
3. **Parking cap:** Type C buses must fit in Type C-allowed locations; Type A and Minivans fill remaining space anywhere. If the fleet exceeds parking, the excess is excluded (Type A/Minivan trimmed proportionally) and flagged.
4. **Vehicle → route substitution:**
   - Type A wheelchair buses may cover Type A non-wheelchair routes.
   - Type C buses may cover Type A non-wheelchair routes.
   - Wheelchair routes need Type A wheelchair buses.
   - Minivan routes need minivans; minivans do nothing else.
   - Each type serves its own routes first; spare A-wheelchair and Type C buses then cover unmet A non-wheelchair routes.
5. **Demand is cumulative:** once a route starts, it needs a vehicle and crew every day thereafter.
6. **Serviceable routes** = min(vehicle-feasible routes, usable staff pairs).
7. **To close the gap** grosses the shortfall up by the spare factor, so it reads as head-count / vehicles to add. Vehicle shortfalls are reported by route type; because Type C or A-wheelchair buses can also cover A non-wheelchair routes, that figure is the minimum to add of *any* eligible type.

## Reading the results

- **Summary cards:** routes needed, serviceable, shortfall, first shortfall date, and staff/vehicles to add by the horizon date.
- **Chart:** for each week, three bars — routes needed (stacked by type), usable vehicles (stacked by type), and staff pairs — with a green line at the serviceable count and the shortfall and its cause underneath. Hover any bar for the exact breakdown.
- **Table:** one row per week; click a row (or "Expand all weeks") to see individual dates. Columns show new routes, routes needed, usable vehicles and staff pairs (each with type/role splits), serviceable, vehicle shortfall (which types are short), staff shortfall, and what CFFO and CHRO each need to add.
- **Show results from / Horizon end** set the visible date range; earlier route starts still count toward demand.

## Sharing

- **Export PDF** builds a clean report (summary, chart, weekly plan, all inputs, rules) and opens the browser's print dialog — choose *Save as PDF*. Expand weeks first if you want dates in the report. `index.html?report` opens the report view directly.
- **Export JSON / Import JSON** saves or loads the full scenario as a file.
- **Reset to defaults** restores the built-in sample data.

## Notes

- The built-in defaults for parking capacity, staff counts, and fleet size are placeholders; replace them with real figures.
- Data is stored in the browser under the key `routeCapacityState.v2`. Clearing site data resets the planner.
