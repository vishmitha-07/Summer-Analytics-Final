# Summer-Analytics-Final


Optimized Parking Pricing Report

 1. Overview
This project implements an advanced dynamic pricing engine for smart parking systems. It calculates and adjusts parking lot prices based on real-time demand, surrounding competition, and traffic conditions, with the ultimate goal of balancing occupancy and maximizing revenue.

2. Demand Function Design

The demand function in the provided program is implemented within the DemandBasedPricingModel class, specifically in the calculate_demand method. Here's the demand function extracted from the code:
def calculate_demand(self, occupancy, capacity, queue_length, 
                    traffic_weight, is_special_day, vehicle_weight):
    occupancy_rate = occupancy / capacity
    demand = (self.alpha * occupancy_rate + 
             self.beta * queue_length - 
             self.gamma * traffic_weight + 
             self.delta * is_special_day + 
             self.epsilon * vehicle_weight)
    
    normalized_demand = self._approx_tanh(demand)
    return normalized_demand
This demand function takes several inputs:

a. occupancy: Current number of occupied parking spots
b. capacity: Total parking capacity
c. queue_length: Number of vehicles waiting for parking
d. traffic_weight: Traffic condition factor (low=0.8, average=1.0, high=1.3)
e. is_special_day: Binary indicator for special/holiday days
f. vehicle_weight: Vehicle type factor (car=1.0, bike=0.7, truck=1.5, cycle=0.5)

The function calculates demand as a weighted sum of these factors:

a. Positive contribution from occupancy rate (α = 1.2)
b. Positive contribution from queue length (β = 0.3)
c. Negative contribution from traffic weight (γ = 0.5)
d. Positive contribution from special days (δ = 0.8)
e.Positive contribution from vehicle type (ε = 0.4)

The raw demand value is then normalized using a simplified tanh approximation to keep it within a reasonable range.

This demand value is later used in the calculate_price method to determine the parking price:

price = self.base_price * (1 + 0.5 * demand)

The demand function essentially models how different factors affect the parking demand, which then influences the dynamic pricing strategy.



3. Assumptions:
   A. Demand Model Assumptions
•Linear Additive Demand:
The demand function assumes that all factors (occupancy, queue length, traffic, etc.) contribute linearly and additively to demand. In reality, some factors may interact non-linearly (e.g., high traffic + high occupancy could have a compounding effect).

•Fixed Weights (α, β, γ, δ, ε):
The coefficients (alpha=1.2, beta=0.3, etc.) are hardcoded and assumed to be static. In practice, these should be learned from data or adjusted dynamically.

•Simplified tanh Normalization:
The _approx_tanh function is a rough approximation of the real tanh function, which may not properly constrain demand in extreme cases.

   B. Pricing Model Assumptions
•Base Price is Fixed ($10):
The base_price is arbitrary and does not adapt to market conditions or inflation.

•Price Bounds are Arbitrary:
The linear model caps prices at 2 × base_price and floors them at 0.5 × base_price, but these limits are not empirically justified.

•Competitor Influence is Static:
The competition_factor=0.3 assumes competitors always affect pricing by 30%, but in reality, this could vary by location/time.

  C. Data & Environment Assumptions
•Competitor Prices are Random:
The program generates fake competitor prices (np.random.uniform(8, 15)), assuming they are uniformly distributed, which is unrealistic.

•Traffic and Vehicle Type Impacts are Fixed:
Traffic weights (low=0.8, high=1.3) are static.
Vehicle weights (car=1.0, truck=1.5) are arbitrary and don’t reflect real-world demand elasticity.

•Special Days are Binary (0 or 1):
The IsSpecialDay column is treated as a simple flag, ignoring varying degrees of demand (e.g., holidays vs. weekends).

•Competitor Proximity is Simplified:
The competitive model assumes a fixed competitor at (26.1445, 91.7361) and uses squared distance instead of real-world travel distance.

  D. Operational Assumptions
•No Time-Based Effects:
The model ignores time-of-day or day-of-week trends (e.g., higher demand during rush hour).

•No Elasticity of Demand:
The model does not account for how price changes might reduce demand (e.g., if prices are too high, drivers may leave).

•Perfect Data Availability:
The code assumes all columns (Occupancy, QueueLength, etc.) are always available with no missing values.

  E. Simulation Assumptions
•Single Competitor:
The competitive pricing model only considers one competitor, whereas real markets have multiple rivals.

•No Latency in Updates:
The model reacts instantly to changes in occupancy/queue length, but real systems may have delays.


4. How price changes with demand and competition:

  price changes are determined by two key models:

    •Demand-based pricing (DemandBasedPricingModel)
    •Competition-aware pricing (CompetitivePricingModel)

  Here’s how prices adjust dynamically based on demand and competition:
A. Demand-Based Pricing
The DemandBasedPricingModel calculates price as:
Price = BasePrice × (1 + 0.5 × NormalizedDemand)

Demand Factors & Their Effects
Factor	               |  Coefficient	 |    Impact on Price
Occupancy rate	       |    α = 1.2    |  ↑ Occupancy → ↑ Price
Queue length           |  	β = 0.3    |  ↑ Queue → ↑ Price
Traffic conditions     |	  γ = 0.5    |  ↑ Traffic → ↓ Price (counterintuitive*)
Special days	         |    δ = 0.8	   |  Holiday → ↑ Price
Vehicle type           |   	ε = 0.4	   |  Trucks (weight=1.5) → ↑ Price


Example:

•If demand surges due to high occupancy (α) and a long queue (β), the price rises.
•The tanh normalization keeps demand between -1 and 1, so prices stay within 50% of the base price (e.g., $10 → $5 to $15).

* Note: The negative γ (traffic weight) is unusual—real models might treat traffic as increasing demand (and thus price). This might be a bug or oversimplification.

B. Competition-Aware Pricing
  The CompetitivePricingModel adjusts prices further based on:

    •Competitor’s price
    •Proximity to competitor (if within 1 km)

   Price Adjustment Rules
     a.If near competitor (distance < 1 km):

         ○ Calculate adjusted price:
            price_diff = CompetitorPrice - OurBasePrice  
            adjustment = 0.3 × price_diff  # 30% of the difference  
            adjusted_price = OurBasePrice + adjustment

          ○ Apply price ceiling: Never exceed the competitor’s price if occupancy is high.:
            if (occupancy >= capacity) & (competitor_price < adjusted_price):
                 price = min(adjusted_price, competitor_price)
            else:
                 price = adjusted_price
      b. If not near competitor:

            ○ Use the pure demand-based price.

        Example:
           •Competitor charges $12, our base price is $10 → Adjusted price = $10 + 0.3×($12-$10) = $10.60.
           •If we’re full (occupancy >= capacity) and the competitor is cheaper ($12 < $10.60), we match their price ($12).

          Key Dynamics
 Scenario                       |	Price Change Mechanism
High demand (↑ occupancy/queue) |	Price increases via demand_price = base_price × (1 + 0.5 × demand).
Competitor undercuts us	        |We shift toward their price (but only 30% of the difference).
Traffic congestion              |	Price decreases (questionable—real models might do the opposite).
Special event (holiday)	        |Price increases due to δ = 0.8.


Suggested Improvements:
•Fix traffic weight: Change γ to a positive value (congestion → higher demand → higher prices).
•Add price elasticity: Reduce demand if prices exceed a threshold.
•Dynamic competitor data: Fetch real competitor prices/locations via API.

5. Visual Representation of Dynamic parking pricing models:

![Screenshot 2025-07-09 231012](https://github.com/user-attachments/assets/f078b129-38c3-47d3-8775-17666f0a7320)
![Screenshot 2025-07-09 231040](https://github.com/user-attachments/assets/6fead3d9-9dce-4dfd-9eba-93ded2b3e21e)


                        End of Report
