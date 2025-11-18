###Module 2 — Plan Builder (Options → Days)###
Revised with Distance Mode Awareness, Budget Adherence, and Clear Input Definitions

Inputs
Required Inputs
>Trip dates (start and end)
>Lodging location (coordinates or address)
>Transit mode (walk, public transit, drive)
>Daily budget target and hard cap
>User preferences (themes, dietary restrictions, accessibility needs, must-see activities)

Optional Inputs
>Reservations (restaurants, events)
>Weather forecast (to adapt indoor/outdoor activities)
>Energy level preference (light vs. intensive days)
>Companion constraints (family-friendly, pet-friendly, age restrictions)
>Rule: If required inputs are missing, the module must reject the task gracefully rather than hallucinate or infer unsupported details.

Activity Schema
>Type: Attraction, Restaurant, Park, Event, Shop
>Theme: Culture, Nature, Food, Nightlife, Shopping, Family, Wellness
>Estimated duration: On-site hours (excluding travel)
>Cost range: Free, , $, $$$
>Distance: Time-based travel estimate, computed per transit mode (walk, public transit, drive)
>Opening hours: Day-specific availability windows
>Constraints: Accessibility, dietary fit, reservation requirement
>Confidence: Data completeness score

Selection Rules per Slot
>Morning: Near lodging, open by morning window, lighter activity if preferred
>Midday: Close to morning activity, feasible travel radius based on transit mode
>Afternoon: Different theme, medium duration, feasible travel from midday activity
>Evening: Restaurant or event, dietary/nightlife preferences, reservation-aware

Global Rules
>Distance Mode Awareness:
>Respect transit mode (walk, public transit, drive)
>Compute feasible radii and travel times accordingly
>Adjust total day duration to include transit time
>Estimate transit costs (e.g., fares, parking) and add to budget tracking

Budget Adherence:
>Track cumulative daily cost (activities + transit + meals)
>Enforce daily budget cap; flag overages with alternatives
>Reject infeasible plans that exceed hard budget limits

Time Feasibility:
>Sum(duration + travel + buffer) per slot ≤ slot window
>Adjust or trim activities if overflow occurs

Theme Diversity:
>Avoid repeating themes back-to-back unless explicitly requested or inventory is limited
>Weather Adaptation:
>Tag indoor/outdoor activities; swap if forecast conflicts

Pseudocode
python
for day in trip_days:
    context = build_day_context(
        day, lodging, prefs, budget, weather, reservations, transit_mode
    )

    morning = pick_activity(
        candidates,
        slot="morning",
        near=lodging,
        open_in=slot_window["morning"],
        constraints=prefs.constraints,
        mode=transit_mode,
        budget=budget
    )

    midday = pick_activity(
        candidates,
        slot="midday",
        near=morning.location,
        open_in=slot_window["midday"],
        constraints=prefs.constraints,
        mode=transit_mode,
        budget=budget
    )

    afternoon = pick_activity(
        candidates,
        slot="afternoon",
        theme!=morning.theme,
        open_in=slot_window["afternoon"],
        constraints=prefs.constraints,
        mode=transit_mode,
        budget=budget
    )

    evening = pick_activity(
        candidates,
        slot="evening",
        type in ["restaurant","event"],
        dietary=prefs.dietary,
        reservation_aware=True,
        open_in=slot_window["evening"],
        mode=transit_mode,
        budget=budget
    )

    plan_day = resolve_conflicts_and_adjust(
        [morning, midday, afternoon, evening],
        time_buffers,
        budget_caps,
        weather_rules,
        transit_mode
    )

    if plan_day.exceeds_budget:
        plan_day = suggest_alternatives(plan_day, budget)

    output.append(plan_day)
