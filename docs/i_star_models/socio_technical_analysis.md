# Sprint 2: Socio-Technical Modelling of the Agro-Marketplace (i* SD and SR Models)

| | |
| :--- | :--- |
| **Module** | UFCFTR-30-3 Distributed and Enterprise Software Development |
| **Author** | Muhammad Raza |
| **Student ID** | 25020179 |
| **Sprint** | Sprint 2 |
| **Artefacts** | `strategic_dependency_model.drawio`, `strategic_rationale_model.drawio` (with exported images) |

---

## 1. Purpose and Scope

This document gives the socio-technical view of the agro-marketplace for Sprint 2. The Sprint 1 process models describe *what happens* when a producer lists produce and a buyer places an order. The i\* models in this sprint describe *why* it happens that way. They show who depends on whom, what each party is trying to achieve, and which quality concerns shaped the design choices that the operational BPMN and Camunda models later implement.

Two models were produced:

* **Strategic Dependency (SD) model.** Each actor is treated as a black box. The model shows only the dependencies between actors.
* **Strategic Rationale (SR) model.** The two actors whose reasoning drives the system design, the Producer and the Marketplace Platform, are opened up. Their goals, alternative ways of reaching those goals, and the effect of each alternative on quality concerns are made explicit.

The scope covers the listing of produce, organic licence verification, order placement and payment. Delivery logistics and refunds are outside the scope of this sprint.

## 2. Modelling Approach and Notation

The models follow the **iStar 2.0** language guide (Dalpiaz, Franch and Horkoff, 2016), which is the current consolidated version of the i\* framework originally proposed by Yu (1995; 1997). Both diagrams were drawn in draw.io so that the group can edit them as the design changes.

| Construct | Shape | Meaning in this model |
| :--- | :--- | :--- |
| Actor | Circle | An active entity with its own intentions |
| Agent | Circle with a line at the top | A concrete actor, such as a specific system or organisation |
| Role | Circle with a curved line at the bottom | An abstract position that different people or organisations can occupy |
| Goal | Rounded rectangle | A state of affairs the actor wants to reach, with a clear satisfied/not satisfied criterion |
| Quality (softgoal) | Cloud | A quality the actor wants, with no sharp satisfaction criterion |
| Task | Hexagon | A particular way of doing something |
| Resource | Rectangle | A physical or informational entity needed to perform a task |

| Link | Notation | Meaning |
| :--- | :--- | :--- |
| Dependency | Depender → *D* → Dependum → *D* → Dependee | The depender relies on the dependee for the dependum |
| AND-refinement | Line ending in a bar | The parent is achieved only if all children are achieved |
| OR-refinement | Solid arrowhead | Each child is an alternative way of achieving the parent |
| Contribution | Dashed arrow labelled *make*, *help*, *hurt* or *break* | Positive (make, help) or negative (hurt, break) effect on a quality |

Every dependency has the same identifier (D1–D15) in both diagrams and in this document, so a dependency can be traced from the SD model into the SR model.

## 3. Actors

| Actor | iStar 2.0 type | Description | Reason for the type chosen |
| :--- | :--- | :--- | :--- |
| **Producer** | Role | A farmer, grower or cooperative that sells produce through the marketplace | Many different businesses occupy the same position with the same responsibilities |
| **Buyer** | Role | A household or trade customer who buys produce | As above. The model is about the buying position, not a particular person |
| **Marketplace Platform** | Agent | The software system that validates, publishes and sells listings | A single concrete system that the team is building |
| **Organic Certification Body** | Agent | The external organisation that issues and records organic licences | A real organisation with its own registry, outside the platform's control |
| **Payment Gateway** | Agent | The external payment service that authorises card payments | A concrete third-party service |
| **Compliance Officer** | Role | Marketplace staff member who reviews listings that automated checks cannot settle | The responsibility matters, not the individual who holds it |

## 4. Strategic Dependency (SD) Model

### 4.1 Dependency Register

| ID | Depender | Dependum | Type | Dependee | Rationale |
| :--- | :--- | :--- | :--- | :--- | :--- |
| D1 | Marketplace Platform | Batch Listing Details | Resource | Producer | The platform cannot create a listing without harvest date, best-before date, price, quantity and licence reference. |
| D2 | Producer | Listing Published | Goal | Marketplace Platform | The producer cares that the listing goes live, not how the platform achieves it. |
| D3 | Producer | Fair Catalogue Visibility | Quality | Marketplace Platform | Producers expect their listings to be shown on equal terms with competitors. There is no sharp test for "fair", so this is a quality. |
| D4 | Marketplace Platform | Current Stock Levels | Resource | Producer | Stock shown to buyers is only as accurate as the figures producers keep up to date. |
| D5 | Buyer | Order Placed | Goal | Marketplace Platform | The buyer wants a confirmed order and leaves the method to the platform. |
| D6 | Buyer | Product Origin Details | Resource | Marketplace Platform | Farm of origin and batch dates inform the buying decision. |
| D7 | Buyer | Trustworthy Organic Claims | Quality | Marketplace Platform | Buyers pay a premium for organic produce and need confidence that the label is genuine. |
| D8 | Marketplace Platform | Delivery & Payment Details | Resource | Buyer | An order cannot be completed without an address and a payment method. |
| D9 | Marketplace Platform | Payment Authorised | Goal | Payment Gateway | Card authorisation is delegated. The platform needs only the outcome. |
| D10 | Marketplace Platform | Secure Card Handling | Quality | Payment Gateway | Card data never passes through the platform, so the platform relies on the gateway for this quality. |
| D11 | Marketplace Platform | Licence Status Record | Resource | Organic Certification Body | The registry record (licence number, holder, status, expiry) is the evidence for an organic claim. |
| D12 | Marketplace Platform | Registry Availability | Quality | Organic Certification Body | Automated verification works only while the registry service is reachable. |
| D13 | Marketplace Platform | Review Flagged Listing | Task | Compliance Officer | When automated checks fail or are inconclusive, the platform depends on a person to carry out a defined review procedure. It is modelled as a task because the platform prescribes how the review is done. |
| D14 | Compliance Officer | Audit Case File | Resource | Marketplace Platform | The officer can only decide a case with the submitted data, the registry response and the reason the listing was flagged. |
| D15 | Producer | Valid Organic Certificate | Resource | Organic Certification Body | A producer can only claim organic status after being certified. |

**Summary by type:** 3 goals, 1 task, 7 resources and 4 qualities across 6 actors.

### 4.2 Vulnerability Analysis

An i\* dependency is also a vulnerability: if the dependee does not deliver, the depender's own goals are at risk (Yu, 1997). Counting the links in the SD model shows where the system is most exposed.

| Actor | As depender | As dependee | Observation |
| :--- | :---: | :---: | :--- |
| Marketplace Platform | 8 | 6 | The most connected actor. Most of its risk comes from external agents. |
| Producer | 3 | 2 | Depends on both the platform and the certification body. |
| Buyer | 3 | 1 | Mostly a depender. Buyer trust is earned through D6 and D7. |
| Organic Certification Body | 0 | 3 | An external party the platform cannot control. |
| Payment Gateway | 0 | 2 | An external party the platform cannot control. |
| Compliance Officer | 1 | 1 | A small number of links, but D13 is the fallback for a critical path. |

Three dependencies need design attention:

1. **D11/D12 (Certification Body).** If the registry is down, organic listings cannot be verified automatically. Rejecting them would penalise honest producers. Accepting them unverified would break D7. The SR model resolves this with a manual-review alternative (Section 5.2).
2. **D1/D4 (Producer data quality).** Incorrect dates or stock figures pass straight to buyers through D6. The platform reduces this risk by validating data on entry rather than trusting it.
3. **D9 (Payment Gateway).** An order must not be confirmed, and stock must not be permanently reserved, until payment is authorised. This shapes the order of tasks in the ordering process.

## 5. Strategic Rationale (SR) Model

### 5.1 Producer

The Producer's root goal is **Produce Sold Profitably**. The model shows two alternatives (OR-refinement):

* **Sell via Marketplace.** This is AND-refined into *Submit Product Listing* and *Manage Stock Levels*. *Submit Product Listing* is further AND-refined into *Set Price & Quantity*, *Declare Licence Reference* and *Enter Batch & Harvest Dates*.
* **Sell at Local Market.** This is the producer's existing option. It is included because the marketplace has to be more attractive than it.

| Source | Contribution | Quality | Reasoning |
| :--- | :---: | :--- | :--- |
| Sell via Marketplace | make | Wide Customer Reach | Online listing reaches buyers well beyond travelling distance. |
| Sell at Local Market | hurt | Wide Customer Reach | Sales are limited to people who visit the market. |
| Sell at Local Market | help | Low Admin Effort | No forms, licence references or stock updates are needed. |
| Submit Product Listing | hurt | Low Admin Effort | Structured data entry is extra work for the producer. |
| Manage Stock Levels | help | Good Seller Reputation | Accurate stock avoids cancelled orders. |
| Declare Licence Reference | help | Good Seller Reputation | Verified organic status separates the producer from unverified sellers. |

The negative link from *Submit Product Listing* to *Low Admin Effort* matters for design. It is the main reason a producer might stay with the local market. The listing form in the operational model should therefore ask only for data the platform genuinely needs (D1) and should be pre-filled where possible.

### 5.2 Marketplace Platform

The platform has two main goals, **Listing Published** and **Order Processed**.

**Listing Published** has two alternatives:

* **Publish Without Checks.** Fast for producers, but nothing stops wrong dates or false organic claims reaching buyers.
* **Publish After Validation.** This is AND-refined into *Validate Listing Data*, *Licence Verified* and *Index Listing in Catalogue*.

The goal **Licence Verified** is OR-refined into *Query Certification Registry* and *Refer to Compliance Review*. In iStar 2.0 an OR-refinement is inclusive, so the design uses both. The registry query is the normal path. The manual review is used when the registry is unavailable (D12) or when the registry record does not match the producer's declaration.

**Order Processed** is AND-refined into *Reserve Stock Atomically* and *Take Payment*.

*Run Nightly Seasonal Check* is a separate scheduled task. It withdraws listings whose best-before date or season has passed.

| Source | Contribution | Quality | Reasoning |
| :--- | :---: | :--- | :--- |
| Publish Without Checks | help | Fast Producer Onboarding | The listing goes live straight away. |
| Publish Without Checks | break | Data Accuracy | Nothing prevents invalid or misleading listings. |
| Publish After Validation | hurt | Fast Producer Onboarding | Validation adds time, especially when a manual review is needed. |
| Validate Listing Data | make | Data Accuracy | Rules such as price > 0, harvest date ≤ today and best-before date > today are enforced before publication. |
| Run Nightly Seasonal Check | help | Data Accuracy | Expired or out-of-season produce is removed without relying on producers. |
| Query Certification Registry | hurt | Fault Tolerance | Listing approval now depends on an external service. |
| Refer to Compliance Review | help | Fault Tolerance | Listings can still be decided when the registry is unavailable. |
| Reserve Stock Atomically | make | No Overselling | A conditional update decrements stock only if enough remains, so concurrent orders cannot claim the same unit. |
| Licence Verified | make | Buyer Trust | Every organic badge is backed by a registry record or an officer's decision. |
| Data Accuracy | help | Buyer Trust | Correct dates and origin details support confidence in the catalogue. |
| No Overselling | help | Buyer Trust | Buyers are not let down by orders that cannot be fulfilled. |

### 5.3 Dependencies in the SR Model

The SR model connects each dependency to the internal element that needs it or provides it.

| ID | Internal element (depender side) | Internal element (dependee side) |
| :--- | :--- | :--- |
| D1 | Platform: Validate Listing Data | Producer: Enter Batch & Harvest Dates |
| D2 | Producer: Sell via Marketplace | Platform: Listing Published |
| D3 | Producer: Produce Sold Profitably | Platform: Index Listing in Catalogue |
| D4 | Platform: Index Listing in Catalogue | Producer: Manage Stock Levels |
| D5 | Buyer (actor) | Platform: Order Processed |
| D7 | Buyer (actor) | Platform: Buyer Trust |
| D9, D10 | Platform: Take Payment | Payment Gateway (actor) |
| D11 | Platform: Query Certification Registry | Organic Certification Body (actor) |
| D13 | Platform: Refer to Compliance Review | Compliance Officer (actor) |
| D15 | Producer: Declare Licence Reference | Organic Certification Body (actor) |

D6, D8, D12 and D14 are attached to the Marketplace Platform boundary as a whole. They concern the platform in general rather than one internal element.

### 5.4 Evaluation of Alternatives

A qualitative forward evaluation (Horkoff and Yu, 2016) was used to compare the two ways of publishing a listing. Each alternative was marked as selected and its effect propagated through the contribution links.

| Quality | Alternative A: Publish Without Checks | Alternative B: Publish After Validation (chosen) |
| :--- | :--- | :--- |
| Fast Producer Onboarding | Partially satisfied (help) | Partially denied (hurt) |
| Data Accuracy | Denied (break) | Satisfied (make from validation, help from nightly check) |
| Fault Tolerance | Not affected | Partially satisfied. The hurt from the registry query is offset by the help from manual review. |
| Buyer Trust | Partially denied. No licence evidence, and data accuracy is denied. | Satisfied (make from Licence Verified, help from Data Accuracy and No Overselling) |
| Effect on D7 (Trustworthy Organic Claims) | Not met | Met |

Alternative B is chosen. It gives up some onboarding speed, which is a Producer-side quality. In return it satisfies the qualities that the Buyer depends on (D7) and that protect the marketplace legally. The onboarding cost is limited by making the registry query automatic, so most producers never wait for a manual review.

## 6. Socio-Technical Trade-offs

1. **Onboarding speed against data accuracy.** Producers want listings live quickly, while buyers need listings they can trust. Validation is kept automatic and immediate, and only uncertain cases are delayed. This narrows the gap between the two.
2. **Automation against dependence on an outside organisation.** Automatic licence checks are fast but make the platform depend on a registry it does not control (D11, D12). The manual-review path keeps the process running during outages. The cost is officer workload, which is why manual review is the fallback rather than the default.
3. **Producer effort against buyer confidence.** The data required from producers (D1) is the main source of producer friction (*Submit Product Listing* hurts *Low Admin Effort*). It is also the data buyers rely on (D6). The form therefore asks only for fields that are validated or shown to buyers, so no data is collected that serves neither side.

## 7. Traceability to the Operational Process Models

The table below shows how the SR elements are expected to be realised in the Sprint 2 operational BPMN and Camunda models.

| i\* element | Type | Operational BPMN construct | Camunda realisation |
| :--- | :--- | :--- | :--- |
| Submit Product Listing | Task (Producer) | User task in the Producer lane | User task with a form covering the D1 fields |
| Validate Listing Data | Task | Service task followed by an exclusive gateway (valid / invalid) | Service task (Java delegate or external task) setting a validation result variable |
| Query Certification Registry | Task | Service task calling the registry | External task or connector with retries |
| Refer to Compliance Review | Task | Error boundary event on the registry call, leading to a user task | User task assigned to the compliance officer candidate group (D13) |
| Licence Verified | Goal | Converging gateway after the automatic and manual paths | Process variable recording the licence status |
| Index Listing in Catalogue | Task | Service task at the end of the listing process | Service task that sets the listing status to active |
| Run Nightly Seasonal Check | Task | Separate process started by a timer start event | Timer start event with a daily cycle |
| Reserve Stock Atomically | Task | Service task before payment in the order process | Service task performing a conditional stock update |
| Take Payment | Task | Service task with an error boundary event for declined payments | Service task calling the payment gateway (D9) |
| No Overselling | Quality | Exclusive gateway routing "insufficient stock" to an out-of-stock end event | Gateway condition on the stock reservation result |

## 8. Assumptions and Limitations

* The Organic Certification Body is assumed to offer an online lookup of licence status. If only a downloadable register is available, *Query Certification Registry* would instead read a locally synchronised copy, and D12 would become less critical.
* Only the Producer and the Marketplace Platform are expanded in the SR model. The Buyer, Payment Gateway, Certification Body and Compliance Officer are shown as collapsed actors, because their internal reasoning does not change the platform design in this sprint.
* Contribution strengths (make/help/hurt/break) are the team's qualitative judgement. They have not been measured.

## 9. Model Artefacts

| File | Content |
| :--- | :--- |
| `strategic_dependency_model.drawio` | Editable source of the SD model (6 actors, 15 dependencies) |
| `strategic_rationale_model.drawio` | Editable source of the SR model (Producer and Marketplace Platform expanded) |
| `strategic_dependency_model.png` / `.svg` | Image exported from draw.io for viewing on GitHub |
| `strategic_rationale_model.png` / `.svg` | Image exported from draw.io for viewing on GitHub |

## 10. References

Dalpiaz, F., Franch, X. and Horkoff, J. (2016) *iStar 2.0 Language Guide*. arXiv:1605.07767. Available at: https://arxiv.org/abs/1605.07767

Horkoff, J. and Yu, E. (2016) 'Interactive goal model analysis for early requirements engineering', *Requirements Engineering*, 21(1), pp. 29–61.

Yu, E. (1995) *Modelling Strategic Relationships for Process Reengineering*. PhD thesis. University of Toronto.

Yu, E. (1997) 'Towards modelling and reasoning support for early-phase requirements engineering', in *Proceedings of the Third IEEE International Symposium on Requirements Engineering*. Annapolis, MD: IEEE, pp. 226–235.

Yu, E., Giorgini, P., Maiden, N. and Mylopoulos, J. (eds.) (2011) *Social Modeling for Requirements Engineering*. Cambridge, MA: MIT Press.
