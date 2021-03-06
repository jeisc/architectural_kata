# architectural_kata

[Introduction](#intro)

[Architectural Decision Records](#adrs)

[Actors and Actions](#actors)

[Components and Responsibilities](#components)

[Questions](#questions)

[Links](#links)

<a name="intro"/>

## Introduction
We are proposing a system whose primary responsibilities are to
* track meal inventory 
* allow for management of meal prep orders and distribution
* track customer order history
* track and respond to customer feedback
* track customer demographic details
* manage promotions and dynamic pricing

We consider the following out of scope (or future work)
* pre-ordering meals via website or app
* meal subscription plans
* tiered pricing based on customer demographics
* storing of sensitive health data that has to abide by HIPAA

We assume
* costs and invoices (for kitchens, distributors, fridge and kiosk rentals) are managed externally via Quickbooks
* sales figures from customer transactions are communicated to Quickbooks and any external BI systems for reporting
* meal ids from customer transactions are reported to our solution (to the customer order history component) so they can be aligned with meal-specific feedback
* any pricing data housed within our solution exists only so we can push prices dynamically to the smart fridge and kiosk POS software

<a name="adrs"/>

## Architectural Decision Records (ADRs)
* The initial architecture diagram is [here](https://github.com/hananoyama/architectural_kata/blob/main/Farmacy%20Food%20architecture%20diagram.png). The diagram includes indicators for ADRs. It also follows the convention of solid connectors for synchronous and dashed connectors for asynchronous communication. 

* These are the ADRs:
  * [ADR1 - processing feedback.md](https://github.com/hananoyama/architectural_kata/blob/main/ADR1%20-%20processing%20feedback.md)
  * [ADR2 - meal prep scheduling.md](https://github.com/hananoyama/architectural_kata/blob/main/ADR2%20-%20meal%20prep%20scheduling.md)
  * [ADR3 - handling promotions.md](https://github.com/hananoyama/architectural_kata/blob/main/ADR3%20-%20handling%20promotions.md)
  * [ADR4 - handling distribution.md](https://github.com/hananoyama/architectural_kata/blob/main/ADR4%20-%20handling%20distribution.md)
  * [ADR5 - inventory tracking.md](https://github.com/hananoyama/architectural_kata/blob/main/ADR5%20-%20inventory%20tracking.md)


<a name="actors"/>

## Actors and Actions
We initially tried to identify actors and actions based on the requirements and presentation transcript, captured in [this diagram](https://github.com/hananoyama/architectural_kata/blob/main/Farmacy%20Food%20actors%20and%20actions%20(initial%20draft).png).

* Registered Customer
  * view available meals by location
  * view order history
  * identify diet/health preferences
  * identify demographic characteristics
  * provide feedback on a past meal
  * purchase meal at a kiosk or smart fridge
  * subscribe to a meal plan of delivered meals (future)
  
* Unregistered Customer
  * purchase meal at a kiosk or smart fridge
  * become a registered customer

* Merchant (Farmacy Food)
  * view feedback on a customer's past meal
  * view aggregate feedback on meals
  * view aggregate feedback on locations
  * view aggregate feedback from a customer
  * issue a customer survey
  * set prices on meals
  * set discounts on meals by menu item, by date/time
  * set discounts on meals by demographic (future)
  * provide coupons to customers by demographic
  * view meal inventory by location(s)
  * view aggregate meal transactions by location, by customer, by menu item
  * schedule and request meal prep from the kitchen
  * decide what meals to produce and where to distribute
  
* Kiosk Point-of-Sale (Toast)
  * record transacted meal
  * provide current kiosk inventory

* Smart Fridge (Byte Tech)
  * record transacted meal
  * provide current fridge inventory

* Kitchen
  * prepare meals based on merchant order
  * procure ingredients to prepare meals

* Kitchen Inventory Management (Cheftec)
  * manage meal recipes
  * manage ingredient procurement/inventory
  * track nutrition facts

* Distributor
  * pick up meals from kitchen(s) and deliver to kiosks, fridges and other venues
  
* Donor (future)
  * donate funds
  * identify beneficiaries
  * review donation impact

* Accountant
  * renconcile kitchen invoices with prepped meals
  * manage payment to kitchen staff and distributors
  
<a name="components"/>

## Components and Responsibilities
Based on the actors and actions we identified, we derived a minimal set of components.
* location inventory tracker
  * track meal inventory at a location where meals are sold (kiosk, fridge, etc.)
  * query vendor APIs to maintain inventory
  * receive distributor restocking updates to maintain inventory
* customer order tracker
  * track customer meal order history
* customer demographics
  * collect customer diet and health preferences
  * collect customer income and demographic details (low income, student)
* customer-initiated feedback processor
  * collect customer feedback on ordered meals
  * allow for triage so that merchant, customer service, etc. are notified of critical feedback
* merchant-initiated feedback processor
  * push surveys to customers
  * collect customer survey responses
* meal preparation scheduling
  * allow merchant or admin to send meal orders to the kitchen(s)
  * monitor order fulfillment status and notify inventory tracker of in-flight inventory
* meal distribution
  * inform distributor personnel of prepped meal pickup from kitchen(s)
  * designate where meals need to be distributed (kiosk, fridge, etc. locations)
  * provide updates to inventory tracker upon restocking of distribution points
* pricing
  * allow merchant to set prices on menu items
  * allow setting of discounted prices by menu item or location or other non-customer characteristic
* UI or API components
  * customer UI (web and app)
  * kitchen UI (web or app) or API to interface with kitchen staff or kitchen inventory software
  * merchant/admin UI (web)
  
<a name="questions"/>

## Questions

These were some of the questions we posed to ourselves. The answers were derived from a combination of reading the presentation transcript and assumptions based on what we could learn about the smart fridges and point of sale software.

* How should feedback be handled?
  * How does handling survey feedback differ from handling meal feedback?
    * survey feedback is merchant-initiated and merchant reviews it at their own leisure (merchant pushes to customer)
    * meal feedback is client-initiated (client pushes to merchant) and may be time-sensitive and require review

* How do we handle inventory? Our solution/system is the global repository of meal inventory.
  * How does our system obtain a current snapshot of inventory?
    * Fridges and kiosks, etc. would not push inventory updates (or sales) to our system
    * Need to periodically query fridge/kiosk APIs to get inventory levels and aggregate
    
* How does a kitchen know what meals to prepare?
  * Merchant sets the menu and decides what is required in whcih locations.
  * Kitchen just needs to know how many meals and handle procurement.
  * Distribution is a separate function.

* How does a kiosk differ from a smart fridge?
  * Kiosks and smart fridges are separate modes of distribution. The term "kiosk" is not a synonym for the venue where you might find a smart fridge:  a kiosk is NOT a place where a smart fridge would be placed.

  * A kiosk refers to a place within a coffee shop or some other staffed location where customers can directly grab a Farmacy meal and then pay at a register. From the customers' point of view, they're paying for the Farmacy meal as if it were any other merchandise in the venue. From the venue staff's point of view, there may or may not be a difference, but the transaction has to be captured by Farmacy's point-of-sale system (Toast).

  * A smart fridge could be situated anywhere (e.g. an office). A customer making a purchase at a smart fridge interacts only with the fridge's tablet/electronic interface using a credit/debit card or some other token to auhenticate and pay. The fridge belongs to a third-party (Byte Tech). The completed transaction is captured in the Byte Tech network. The transaction DOES NOT require the involvement of any attendant to complete; that is what makes it a smart fridge. That said, nothing stops a smart fridge from also being placed inside a coffee shop.

* How is inventory tracked by a smart fridge and a kiosk?
  * A smart fridge can track its own inventory levels and act as its own point-of-sale. The product packaging for a meal has an RFID tag that a receiver on the fridge can detect and the act of removing an item from the fridge/closing the door completes the transaction and records the data to the Byte Tech network.

  * The kiosk point-of-sale system (Toast) tracks sales and inventory levels, which are updated when the venue staff completes the sale.

* What if there is insufficient ingredient inventory at a kitchen to prep a meal order?
  * An SLA needs to be set up between the Farmacy Food merchant and any kitchen supplying meals.

* Do distribution points allow for customers to see dynamic prices targeted at them based on their tier (low-income, student)?
  * At a fridge, is there any way to self-identify to see tiered pricing? It seems unlikely, since swiping the payment card is meant to simultaneously identify the customer, allow them to open the fridge door, and authorize charges for any RFID-tagged items that they remove from the fridge.
  * At a kiosk, prices for meals would likely be displayed as in a traditional shop (via stickers or signage?). There is no provision to display dynamic prices.
  * Based on these assumptions, tiered pricing can only apply to subscriptions or web- or app-based ordering, which is future functionality.

* What pricing/cost information lives within our solution?
  * Prices set by the merchant for all the meals. It probably could be customized by location characteristics or be based on the meal's shelf life (i.e. discounting based on shelf life). These prices need to be pushed to the smart fridge or kiosk POS software via their APIs.
  * Underlying costs for ingredients and whatever the kitchen might charge for meal prep falls within the domain of kitchen inventory (Cheftec) and bookkeeping (Quickbooks) software and does not need to be part of our solution. However, accounting may need to be able to check on meal fulfillment by consulting our system via the merchant/admin interface.

<a name="links"/>

## Links
Smart Fridge:
https://bytetechnology.co/
https://thespoon.tech/byte-foods-opens-up-its-smart-vending-platform-with-byte-technology/

Kiosk Point of Sale Software:
https://pos.toasttab.com/

Cheftec Kitchen Inventory Management Software:
https://www.cheftec.com/cheftec-basic
