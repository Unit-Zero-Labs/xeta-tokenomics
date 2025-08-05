# XETA Seller Rewards LP Simulation

## Overview

This simulation compares two economic incentive models for the XETA protocol:
- **Client Model**: Traditional tier-based rewards system
- **Unit Zero Model**: Continuous rewards with emission sharing

The simulation creates a virtual economy with different participants (Sellers and Node Operators) and tracks their behavior over 12 months under each model.

### Key Questions We're Answering
- Which model produces a more stable token price?
- Which generates more predictable protocol revenue?
- How does selling pressure differ between models?
- What are the optimal parameters for the Unit Zero model?

### Script Architecture
The simulation consists of five main components:

1. **Configuration & Data Loading** - Pulls parameters from Google Sheets
2. **Core Classes** - Defines simulation actors and environment
3. **Simulation Engine** - Runs month-by-month calculations
4. **Analysis & Optimization** - Monte Carlo testing and parameter optimization
5. **Execution & Visualization** - Results processing and plotting
## 1. Data Loading and Parsing

The `load_and_parse_sheet` function handles all parameter loading from Google Sheets, keeping the simulation configurable without code changes.

### Process Flow
- **Authentication**: Connects to Google Sheets API
- **Data Import**: Pulls all data from the 'Basic Emissions Model' worksheet
- **Value Extraction**: The `find_value` helper locates specific parameters (like "Total XETA Supply" or "Box Price") and extracts their values
- **Table Parsing**: The `parse_program_table` function reads multi-row tables for both seller programs, processing each tier's rules and rewards
- **Configuration Assembly**: All parsed data gets compiled into a comprehensive CONFIG dictionary

### Configuration Contents
The CONFIG object includes:
- Base tokenomics (supply, launch price, etc.)
- 12-month node growth projections  
- Complete program rules for both Client and Unit Zero models
- Unit Zero's 25% seller emission share allocation
## 2. Core Simulation Classes

These classes represent the participants and infrastructure of the economic simulation.

### Agent (Base Class)
The foundation class for all simulation participants. Every agent has:
- Unique ID
- Type (seller or node_operator)  
- Token wallet

### Seller(Agent)
Models the sales network behavior with several key features:

**Initialization**: Each seller gets assigned:
- Tier information
- Box selling patterns (steady vs. volatile)
- Token selling behavior (hodler, consistent, or full_sell)

**Monthly Operations**:
- `sell_boxes_this_month`: Calculates monthly box sales based on assigned behavior patterns
- `tier_reward_claimed`: Tracks one-time rewards in the Client program to prevent double-payments
- **Token Processing**: Determines how many reward tokens to sell based on individual behavior and global sell rates

### NodeOperator(Agent)
Represents node operators with sophisticated selling logic:

**Dynamic Selling Decisions** based on:
- Base behavior profile (hodler, balanced, full_sell)
- Global monthly sell percentage
- **Price sensitivity**: Operators are more likely to sell when prices are high (taking profits) and hold when prices are low, creating realistic market feedback

### LiquidityPool
Simulates an AMM-style trading pool (similar to Uniswap):

**Core Mechanics**:
- **Reserves**: Maintains XETA and USD balances
- **Pricing**: Uses constant product formula (price = usd_reserve / token_reserve)
- **Trading**: Handles buy/sell operations with realistic slippage effects
- **Fee Collection**: Takes trading fees and reinvests them back into the pool, gradually increasing liquidity depth over time
## 3. Simulation Engine

The `Simulation` class orchestrates the entire economic model, running month-by-month calculations for a full year.

### Monthly Simulation Cycle

The `run()` method executes these steps each month:

#### Network Growth
- Adds new Node Operators based on the growth schedule from the spreadsheet

#### Revenue Calculation  
- Tallies total box sales across all sellers
- Calculates protocol revenue from box sales and recurring fees

#### Emission Distribution
**Client Program**: 100% of monthly emissions go to Node Operators  
**Unit Zero Program**: Split between Sellers (25%) and Node Operators (75%)

#### Seller Rewards
**Client Program**: 
- Checks if sellers hit tier thresholds
- Pays one-time lump-sum rewards for new achievements

**Unit Zero Program**:
- Calculates continuous rewards based on monthly box sales × tier multipliers
- Distributes monthly emission share to active sellers

#### Selling Pressure Calculation
- Sellers decide how many reward tokens to sell based on their behavior profiles
- Results tracked as `monthly_seller_sell_pressure`

#### Node Operator Rewards & Selling
- Distributes emission allocation equally among all active operators  
- Each operator uses dynamic selling logic (factoring in current price vs. launch price)
- Results tracked as `monthly_operator_sell_pressure`

#### Market Execution
1. **Sell Pressure**: Combined selling from sellers and operators hits the liquidity pool, driving price down
2. **Protocol Buybacks**: Protocol uses portion of revenue to buy tokens back, supporting price  
3. **Market Shocks**: Random external trading events simulate real market volatility

#### Data Logging
- Records monthly state (price, revenue, volumes, etc.)
- Builds complete 12-month DataFrame for analysis
## 4. Monte Carlo Analysis & Optimization

Since individual simulations contain random elements, we need statistical analysis across many runs to get meaningful results.

### Monte Carlo Testing
**`run_monte_carlo`**: Runs the simulation hundreds or thousands of times (configurable via `NUM_SIMULATIONS`) to generate a distribution of possible outcomes rather than relying on a single run.

### Results Analysis  
**`analyze_results`**: Processes Monte Carlo output to calculate key metrics:
- Average and median final prices
- 5th percentile "worst-case" price  
- Total revenue distribution
- Price volatility measures

### Automated Optimization
**`optimize_with_optuna`**: Uses AI-powered optimization to find the best Unit Zero program parameters.

**Optimization Goal**: Maximize the worst-case (5th percentile) token price while minimizing volatility

**Process**: 
- Optuna proposes multiplier combinations
- Runs quick simulations to evaluate performance  
- Learns from results to improve next proposals
- Iterates for `OPTIMIZATION_TRIALS` to converge on optimal settings
## 5. Execution and Visualization

The main execution pipeline brings together all components to generate comprehensive results.

### Execution Flow

1. **Load Configuration**: Pulls all parameters from Google Sheets
2. **Run Three Scenarios**:
   - **Client Program**: Traditional tier-based model
   - **Unit Zero Base**: Using default spreadsheet multipliers  
   - **Unit Zero Optimized**: First runs Optuna optimization, then full Monte Carlo with optimal parameters
3. **Aggregate Results**: Combines all scenario data into a unified DataFrame

### Visualization Dashboard

The script generates several comparative visualizations:

**Price Trajectories**: Shows median price paths with 5th-95th percentile confidence bands to illustrate both expected outcomes and risk ranges

**Revenue Distribution**: Boxplots comparing total revenue distributions across all three scenarios

**Selling Pressure Analysis**: Time-series charts breaking down token sales by source (sellers vs. operators) for each model

**Optimization Results**: Bar charts displaying the optimal multipliers discovered by Optuna

### Performance Summary

A final text table presents key performance indicators for easy comparison:
- Final token prices (median, worst-case)  
- Total protocol revenue
- Price volatility metrics
- Model-specific KPIs

This quantitative summary enables quick decision-making about which incentive model performs best under different criteria.