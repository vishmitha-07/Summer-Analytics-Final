# Summer-Analytics-Final


Optimized Parking Pricing Report

 1. Overview
This project implements an advanced dynamic pricing engine for smart parking systems. It calculates and adjusts parking lot prices based on real-time demand, surrounding competition, and traffic conditions, with the ultimate goal of balancing occupancy and maximizing revenue.

2. Demand Function Design
The demand function used is a sigmoid-based model that scales sharply around 60% occupancy. It includes the following factors:
Factor	                  |      Contribution Logic
Occupancy                 |	Sigmoid growth: sharply increases when occupancy > 60%
Queue Length              |	Scales linearly up to a capped value (max +0.3 contribution)
Traffic Level             |	Linear multiplier (0.2 * traffic)
Special Days              |	+0.3 boost in demand if it’s a holiday or event
Vehicle Type	            |Adjusted weight: truck (+0.8), bike (-0.4), car (baseline)
Final demand score is clipped to [0, 1] and directly influences the price scaling.

3. Pricing Mechanism
Base Price: $10
Min Price: $5 (50% of base)
Max Price: $20 (200% of base)
Formula:
 raw_price = BASE_PRICE * (0.5 + demand)
adjusted_price = raw_price * competitive_multiplier
smoothed_price = previous_price * (1 - α) + adjusted_price * α
final_price = clip_and_round(smoothed_price)
 Where α is the smoothing factor (0.3).

4. Competitive Adjustment
Nearby competitors within 0.5 km are evaluated. If occupancy is high and nearby prices are lower, price is raised up to 10%. If a lot is underutilized, it may lower price up to 2%.
Rules:
If occupancy > 90% and nearby avg occ < 70% → Increase price (1.1x)
If current price > avg × 1.1 → Slight reduction (0.98x)
If price < avg × 0.9 → Slight increase (1.02x)

5. Rerouting Logic
If a lot’s occupancy exceeds 90%, rerouting suggestions are made by checking:
Nearby lots within 0.5 km
Occupancy less than 80%
Sorted by shortest distance and least occupancy

6. Assumptions
Location coordinates are sampled near New York City.
Traffic, occupancy, and special day info is generated if not available.
Competitor prices default to base price unless historical data exists.

7. Visualization
Each lot’s:
Price evolution
Occupancy trend
are shown using Bokeh with interactive hover tools.

8. Technologies Used
Python 3
pandas, numpy
haversine
bokeh for plots
pathway (for future streaming extension)

9. Results
Real-time prices are adjusted dynamically.
Low occupancy lots retain low prices.
Overcrowded lots have rerouting suggestions.
See the Bokeh plots in the notebook for evidence.

10. Future Improvements
Real-time streaming integration using pathway
Machine learning to forecast demand
Weather and event-based external factors
Mobile app rerouting recommendation system






End of Report
