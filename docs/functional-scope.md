# Functional Scope and Candidate Functionalities

Based on the Case Study and Use Case 1, the following two major functionalities have been selected for detailed development in Sprint 2 and 3:

## Functionality 1: Product Listing & Data Validation
* **Description:** A producer enters product details via a Camunda Form. The system validates mandatory fields (name, price, category). If organic certification is claimed, a service task calls an external verification API. If valid, the product is published to the marketplace.
* **Why selected:** Demonstrates user interaction, business logic, gateway routing, and external service integration.

## Functionality 2: Inventory & Availability Management
* **Description:** The system tracks product stock levels. When a customer places an order, stock is deducted. If stock reaches zero, the product automatically becomes unavailable. If restocked and within the seasonal period, it becomes available again.
* **Why selected:** Demonstrates automated business rules, process variables, event handling, and alternative paths (available vs. out-of-stock).
