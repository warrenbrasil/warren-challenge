# cocos-challenge-qa-automation

**Summary:**
You have an existing React Native (Expo) investment app—`app-qa`—that consumes a REST API (instruments, portfolio, search, and order submission). Your task is not to add functionality, but to **assess its quality and automate its validation**.

What we are most interested in evaluating is not how many tests you write, but your **judgment**: what you decide to test, **what you decide NOT to test, and why**. A good QA engineer prioritizes based on risk and knows how to justify the scope of their work.

Estimated time: **one week**.

## The app under test

`app-qa` is an existing React Native (Expo) trading app. Clone the repository from [github.com/cocoscap/app-qa](https://github.com/cocoscap/app-qa) and follow its `README` to run it (Bun, an iOS simulator or Android emulator, and a `.env` copied from `.env.example`).

At a high level, the app includes:

- **Instruments**: a list with ticker, name, last price, and daily return.
- **Search**: an instrument search feature by ticker.
- **Portfolio**: available cash and positions, including market value, profit, and return.
- **Orders**: order submission (`BUY`/`SELL`, `MARKET`/`LIMIT`) and order history with status, plus an action to reset the account.

The order form accepts either the **exact number** of shares or an **amount in pesos**, which the app converts into the maximum number of whole shares using the latest price (fractional shares are not allowed).

You decide which level to automate—the UI on a device/simulator, the API consumed by the app, or a combination—and which tools to use. Your choice should be **justified** based on the problem.

## The API under test

Base URL: `https://dummy-api-topaz.vercel.app`

The app consumes this REST API; you may call it directly to automate at that level.

### Required headers and state isolation

Every request requires two headers:

- `X-Enable-Bugs` — required. Controls the API's "defect level" and only accepts the values `off`, `easy`, `medium`, or `hard` (case-insensitive); any other value—or an omitted header—causes the API to respond with `400`.
  - With `off`, the API behaves **correctly** (the "golden path"): this is the baseline against which you write your assertions.
  - `easy`, `medium`, and `hard` **inject intentional defects** with increasing levels of difficulty. They allow you to **validate your own suite**: your tests should **pass with `off`** and **begin to fail** as you increase the level. Detecting every individual bug is not an explicit deliverable, but a good suite should be capable of doing so.
- `X-Candidate-Id: <your-id>` — identifies your session. The `/portfolio`, `/orders`, and `/reset` endpoints respond with `400` without it. The API **isolates** your state (orders, cash, and holdings) by this id, so choose your own value (for example, your name) and you will work with your own account without interfering with other candidates.

Instruments and their prices are **shared and read-only**; your portfolio and orders are scoped per candidate.

Each candidate starts with **1,000,000 ARS** and no positions. The portfolio (cash + holdings) is **derived from your orders with a `FILLED` status**; there is no separately stored balance. Your state **persists** across runs; `POST /reset` clears it (which is useful for preparing or cleaning up scenarios). This is intended to make test case setup easier. The deliverable must explain how the automation could be set up in an environment where a reset is not possible.

### Endpoints

- `GET /instruments` — list of instruments. Each one includes `ticker`, `name`, `last_price`, and `close_price`. The daily return is calculated from the last price and closing price.
- `GET /search?query=<text>` — search for instruments by ticker.
- `GET /portfolio` — `{ cash, holdings }`, derived from your `FILLED` orders and **net of amounts reserved by your `PENDING` orders**. Each holding includes: `ticker`, `quantity`, `last_price`, `close_price`, and `avg_cost_price` (weighted average purchase price). A position's market value is `quantity * last_price`; use `avg_cost_price` to calculate profit ($) and return (%).
- `GET /orders` — your order history.
- `POST /orders` — submits an order. Body:

  ```json
  // Market order, by number of shares
  { "instrument_id": 1, "side": "BUY", "type": "MARKET", "quantity": 1234 }

  // Limit order (requires price)
  { "instrument_id": 1, "side": "SELL", "type": "LIMIT", "quantity": 123, "price": 84.5 }
  ```

  The response includes an `id` and a `status`.
- `POST /reset` — clears your state so you can start over.

### Documented business rules

- Prices are in pesos (ARS), and fractional shares are not allowed: `quantity` must be a **positive integer**.
- `side` can be `BUY` or `SELL`; `type` can be `MARKET` or `LIMIT`.
- An order's `status` can be `FILLED`, `PENDING`, or `REJECTED`.
- `MARKET` orders are executed immediately (`FILLED`) at the instrument's `last_price`.
- `LIMIT` orders are **always created as `PENDING`** and are resolved at some point (conceptually, when the market accepts them; in practice, under certain conditions and with a random factor): each `PENDING` order may remain `PENDING`, transition to `FILLED`, or transition to `REJECTED`.
- When created, both `MARKET` and `LIMIT` orders **reserve funds**: a buy reserves cash and a sell reserves shares. A `PENDING` order maintains that reservation (reducing what is available for new orders), a `FILLED` order settles it, and a `REJECTED` order releases it. Therefore, the `cash` and `holdings` returned by `/portfolio` are **net of reserved amounts**.

## What we expect you to deliver

1. **Test plan** (it may be included in the README). At a minimum:
   - Scope: what you will cover and to what depth.
   - **Out of scope**: what you decided NOT to test and **why** (time, risk, value, environment limitations, etc.).
   - Risk-based prioritization: what is most critical in this system and why.
   - Assumptions made in response to any ambiguity in the assignment or the actual behavior of the app/API.
2. **Automated test suite**. Use the language and framework of your choice. It must include documentation explaining how to run it.
3. **Bug / findings report**. Include any behavior you consider incorrect, inconsistent, or unexpected relative to the documentation. For each finding, provide reproduction steps, expected vs. actual results, severity, and evidence.
4. **README** explaining how to run the suite and the decisions you made.

## Technical considerations

- The suite must be **reproducible** by someone else: provide clear instructions and a single way to run it.
- Consider the **reliability** of your tests: they should not be flaky, and their assertions should verify actual behavior rather than merely confirming that the request "did not crash." Keep in mind that the resolution of `LIMIT` orders is **non-deterministic**.
- Consider **test isolation**: your state persists across runs, and multiple runs may interfere with one another. Use your `X-Candidate-Id` and `POST /reset` to your advantage.
- The app may have **quality issues**, including issues that **make automation more difficult**. Identifying and documenting them is part of the challenge.
- If you need to **modify the app** to automate it, do so; document **what you changed and why**.
- Provide a readable results report (HTML, JUnit, etc.).

## Optional / Nice to have

- Response contract/schema validation.
- A Postman/Insomnia/REST Client collection to support exploratory testing.

## Submission

Push your solution to a Git repository (public or with access granted) with the full commit history. Approach it as if it were intended for a real-world environment (production-ready).
