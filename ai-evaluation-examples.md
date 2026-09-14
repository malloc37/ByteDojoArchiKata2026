# AI evaluation examples

How one evaluation case is judged, for three use cases. **Every record below is synthetic.**
The examples show how a case passes or fails. They are not evidence of real-world
performance. The evaluation rules are in
[ADR-011](adrs/adr-011-ai-evaluation-human-review.md), and the meaning of confidence is in
[ADR-017](adrs/adr-017-ai-model-approach-and-confidence.md).

---

## UC-1 Animal health: three cases

**Case A. A real concern (synthetic).**

| Step | Content |
|---|---|
| Input | Otter O-07. Over four weeks it ate 1.2 kg a day on average. On the last three days it ate 0.6, 0.5 and 0.6 kg. Its activity sensor shows 40 per cent less movement. A keeper note from day two reads "slow to come to feed". |
| Output | `Animal Anomaly Detected`. Four of five runs flag the animal, which the calibration table maps to confidence 0.82. The source events are the three feedings, the activity readings and the note. |
| Expected | A flag. The synthetic label says a keeper confirmed a concern. |
| Result | **Pass.** Confidence 0.82 is above the *proposed* threshold of 0.6, so an inspection task is created, and the label confirms the concern. |

**Case B. A change that is not a concern (synthetic).**

| Step | Content |
|---|---|
| Input | All five otters in enclosure E-12 eat 20 per cent less for three days after their feeding time moves from 09:00 to 11:00. Activity is normal. |
| Output | One of five runs flags otter O-09, which maps to confidence 0.15. |
| Expected | No flag. The synthetic label says keepers found nothing. |
| Result | **Pass.** Confidence is below the threshold, so no task is created. |

**Case C. Missing data (synthetic).**

| Step | Content |
|---|---|
| Input | The activity sensor in enclosure E-04 has not reported for 30 hours. |
| Output | Abstention, with the reason "input stale". |
| Expected | Abstention. |
| Result | **Pass.** A stale input produces an abstention, not a normal result. If the model had returned "no anomaly", the case would fail. |

---

## UC-2 Crowd forecast: one case

| Step | Content |
|---|---|
| Input | Saturday, area Rides North. Counts up to 12:00, plus the past six Saturdays. Forecast asked for 14:00. |
| Output | 1,400 visitors, with a range of 1,200 to 1,650 across five runs. |
| Expected | The actual 14:00 count, 1,520 (synthetic). |
| Result | **Pass.** The actual count is inside the range. The forecast is 7.9 per cent off. The same-weekday baseline predicted 1,250, which is 17.8 per cent off, so the LLM beats its baseline on this case. |

The release decision uses forecast error across all test weeks, never a single case.

---

## UC-3 Visitor itinerary: one case

| Step | Content |
|---|---|
| Input | Preference text: "two kids under 8, no heights, we love animals". The catalogue shows the carousel closed. |
| Output | An itinerary with five stops. One of them is the carousel. |
| Expected | No closed attraction reaches the visitor, and no stop has a height requirement. |
| Result | **The guard passes** and removes the carousel before display. **The model output fails** its quality check, and that failure counts against the model's score. |
