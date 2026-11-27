# Can the collateral pay a Saturday cash call?

27 November 2026. For a small treasury or investment team deciding how much tokenised collateral to count toward an out-of-hours cash obligation.

## What the source establishes

The Guardian Tokenised Collateral Workgroup's 19 November 2026 report separates moving a token, exercising rights over collateral, and raising usable liquidity. Its default and enforcement section asks whether buyers, pricing, settlement assets and fallback financing are available when cash is needed. It is an industry implementation framework published by the Monetary Authority of Singapore. It does not determine the legal status of a particular arrangement. [Guardian Tokenised Collateral Framework, Step 7](https://www.mas.gov.sg/publications/monographs-or-information-paper/2026/guardian-tokenised-collateral-framework)

## Turn that distinction into a cash test

Say the desk owes $750,000 by 18:00 on a Saturday. A token can move at 17:45. One buyer can deliver $800,000 and a standby financing desk can advance $875,000. Both routes use the same position in full, so the spreadsheet cannot add them. In the first scenario the buyer's cash arrives at 18:05. The financing desk can pay at 17:50, but the agreement has not been confirmed to cover this instrument in the scenario. Count zero for the cutoff. If the financing route is confirmed and all its other assumptions hold, count $875,000 once.

Before counting a route, record the instrument and account, the source and age of the quote, who verified that the documented right applies in this scenario, and when net cash reaches the account that must pay. Include approval, custody and payment cutoffs in that last timestamp. A dashboard that records the fastest moving part can be impressively punctual about the wrong thing.

## A runnable desk check

The example treats each route as an all-or-nothing use of one position. Inputs are synthetic. The two booleans stand for evidence somebody has already checked. The program does not determine whether a right exists. Save the block as `cutoff.py` and run `python3 cutoff.py`.

```python
from dataclasses import dataclass, replace


@dataclass(frozen=True)
class Route:
    name: str
    net_cash: int
    usable_at: int  # Minutes after midnight, same day and time zone
    right_confirmed: bool
    cash_route_confirmed: bool


def cash_by_cutoff(routes, cutoff):
    eligible = (
        route.net_cash
        for route in routes
        if route.right_confirmed
        and route.cash_route_confirmed
        and route.usable_at <= cutoff
    )
    return max(eligible, default=0)


cutoff = 18 * 60
cash_call = 750_000
sale = Route("Sale", 800_000, 18 * 60 + 5, True, True)
facility = Route("Standby facility", 875_000, 17 * 60 + 50, False, True)

assert cash_by_cutoff([sale, facility], cutoff) == 0

confirmed_facility = replace(facility, right_confirmed=True)
assert cash_by_cutoff([sale, confirmed_facility], cutoff) == 875_000
assert cash_by_cutoff([sale, confirmed_facility], cutoff) >= cash_call

sale_at_cutoff = replace(sale, usable_at=cutoff)
assert cash_by_cutoff([sale_at_cutoff, confirmed_facility], cutoff) == 875_000

unavailable_rail = replace(confirmed_facility, cash_route_confirmed=False)
assert cash_by_cutoff([sale, unavailable_rail], cutoff) == 0
print("Four scenarios passed.")
```

The notebook's scheduled agent ran this exact block locally in Python 3. It covered late cash, an unconfirmed right, competing routes on the same position and an unavailable payment rail. No actual collateral, custody system, market quote or payment was tested.

A live tabletop would start with a specific obligation, ask the custodian and financing desk to walk through their actual weekend handoffs, and have the legal owner verify the instrument and scenario covered by the agreement. Repeat it with one unavailable provider. Record the remaining cash shortfall and who is responsible for meeting it.
