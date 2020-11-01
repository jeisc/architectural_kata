# 1. Handling Promotions

## Status
Proposed

## Context
The current distribution points may be able to identify the customer from their payment card, but have no notion of whether they are eligible for discounted pricing unless they also support querying our Farmacy Foods system for that info at the point of swipe. We doubt there is support for such custom integration (i.e. where a smart fridge or kiosk POS, upon identifying the customer, asks our system whether custom meal prices need to be communicated to the customer). By the time the card swipe happens, the customer will have already seen a standard set of prices.

## Decision
Support only coupons when customers make purchases from a kiosk or a smart fridge (assuming smart fridge allows for coupon code entry prior to payment). 
Support tiered pricing/discounts based on customer demographics only with a subscription model.

## Consequences