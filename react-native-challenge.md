# cocos-challenge-react_native

**Summary:**
Develop an app that displays the information returned by the following endpoints:
- **/instruments**: The screen must display the list of instruments returned by this endpoint. Show the ticker, name, last price, and return (calculated using the last price and the closing price returned by the same endpoint).
- **/portfolio**: The screen must display the list of assets returned by this endpoint. For each asset, show the ticker, position quantity, market value, profit, and total return (use `avg_cost_price` as the purchase price).
- **/search**: Develop an asset search feature by ticker.
- **/orders**: When an instrument is clicked, display a modal with a form for submitting an order (using the POST method). Users must be able to specify whether it is a buy or sell order (`BUY` or `SELL`), whether the order type is `MARKET` or `LIMIT`, the number of shares to submit, and, only for LIMIT orders, the price to submit. The POST response will include an `id` and a `status`, which can be `PENDING`, `REJECTED`, or `FILLED`. Display the returned id and status. (An example of the parameters to include in the POST body is provided below.)

# Functional considerations
- For the design, you may draw inspiration from applications such as **coinbase.com, binance.com (lite), and robinhood.com.**
- Asset prices are in pesos.
- When users submit an order, they must provide the number of shares they want to buy or sell. Allow users to enter either the exact number of shares or a total investment amount in pesos (in the latter case, calculate the maximum number of shares they can submit using the latest price; fractional shares are not allowed).
- Orders can have different statuses:
    - `**PENDING**` - when a `LIMIT` order is submitted to the market, it is assigned this status.
    - `**FILLED**` - when an order is executed. Market orders are executed immediately upon submission.
    - `**REJECTED**` - when an order is rejected by the market because it does not meet the requirements, for example, when an order is submitted for an amount greater than the available balance.
- When users submit a `LIMIT` order, the returned order status is `PENDING` or `REJECTED`.
- When users submit a `MARKET` order, the returned order status is `REJECTED` or `FILLED`.
- To calculate the market value of a portfolio position, use `quantity * last_price`. Note that `avg_cost_price` is the average purchase price; use it to calculate total profit (absolute amount in $) and return (%).

# Technical considerations
- Develop the application in **React Native** with **TypeScript**. You may use a bare project or any framework of your choice. Expo is preferred, but not required.
- Include a README.md file with clear instructions for running the project and a dedicated section explaining the technical decisions you made during development.
- Any additional functionality you implement will be considered a plus. Approach the application as if it were intended to be production-ready.
- Aspects to consider:
  - Project structure: design an architecture that facilitates maintainability and scalability.
  - Error handling: implement a robust approach to managing application failures.
  - State management: use an appropriate strategy for managing global and/or local state.
- **Add unit tests where appropriate.**

# API data
- GET https://dummy-api-topaz.vercel.app/portfolio
- GET https://dummy-api-topaz.vercel.app/instruments
- GET https://dummy-api-topaz.vercel.app/search?query=DYC
- POST https://dummy-api-topaz.vercel.app/orders
  ```
  Example body 1
  {
      instrument_id: 1,
      side: 'BUY',
      type: 'MARKET',
      quantity: 1234
  }
  Example body 2
  {
      instrument_id: 1,
      side: 'SELL'
      type: 'LIMIT',
      quantity: 123,
      price: 84.5
  }
  ```
    
