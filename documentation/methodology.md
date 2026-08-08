# Data Modeling Methodology

## 1. Project Objective

The objective of this project was to transform a collection of fragmented operational files into a centralized Power BI semantic model capable of supporting multiple business domains.

The model was designed around four primary goals:

1. Establish a consistent analytical structure across business processes.
2. Reduce duplicated data and business logic.
3. Enable cross-domain analysis using shared dimensions.
4. Introduce a scalable approach to data governance and regional access.

The resulting architecture combines **Sales, Inventory, Marketing, Fulfillment, Targets, Customers, Products, and Geography** within a multi-domain Star/Galaxy Schema.

---

# 2. Source Data Assessment

The source material consisted of **22 flat files** representing different business processes and entities.

Before building the model, the data was assessed to understand:

- Business entities
- Transactional processes
- Relationships between datasets
- Repeated attributes
- Potential primary and foreign keys
- Data types
- Duplicate records
- Missing values
- High-cardinality fields
- Fields requiring transformation

The objective was to understand the business structure before creating relationships inside Power BI.

---

# 3. Business Domain Identification

The source files were grouped into logical business domains.

### Sales

- Sales transactions
- Revenue targets
- Customer information
- Product information

### Inventory

- Inventory balances
- Product and supplier information
- Geographic attributes

### Marketing

- Campaign information
- Campaign spending
- Promotion coverage

### Fulfillment

- Order lifecycle milestones
- Invoice information
- Shipment and delivery events
- Payment events

### Governance

- Regional security mapping
- User-to-region access rules

This domain-based approach helped determine which datasets should become facts, dimensions, bridges, or supporting tables.

---

# 4. Fact and Dimension Identification

The next step was separating transactional processes from descriptive entities.

## Fact Tables

The main transactional tables were modeled as facts:

- `fact_sales`
- `fact_inventory`
- `fact_order_process`
- `fact_promotion_coverage`
- `fact_campaign_spend`
- `fact_sales_targets`

Each fact represents a specific business process or measurable event.

## Dimension Tables

Shared descriptive entities were modeled as dimensions:

- `dim_customer`
- `dim_product`
- `dim_geo`
- `dim_campaign`
- `dim_date`
- `dim_order_flags`

This separation creates reusable dimensions that can support multiple analytical processes.

---

# 5. Conformed Dimensions

Shared dimensions were designed as conformed dimensions wherever the same business entity was required across multiple processes.

For example:

```text
dim_product
      │
      ├── fact_sales
      ├── fact_inventory
      └── fact_promotion_coverage
```

This allows product-level analysis to remain consistent across sales, inventory, and marketing.

The same principle was applied to dimensions such as:

- Customer
- Geography
- Date
- Campaign

The goal was to establish a **single analytical definition for common business entities**.

---

# 6. Star / Galaxy Schema Design

Because the model contains multiple fact tables sharing common dimensions, the architecture follows a Galaxy Schema pattern built from multiple Star Schema structures.

Conceptually:

```text
                  dim_product
                       │
          ┌────────────┼────────────┐
          │            │            │
    fact_sales   fact_inventory   fact_promotion
          │                         │
          │                         │
     dim_customer             dim_campaign
          │
          │
    fact_order_process
```

This architecture allows multiple business processes to coexist without forcing them into a single wide table.

---

# 7. Accumulating Snapshot Fact

The `fact_order_process` table was modeled as an **Accumulating Snapshot Fact**.

It captures the progression of an order through important fulfillment milestones:

```text
Order
  ↓
Invoice
  ↓
Shipment
  ↓
Delivery
  ↓
Payment
```

This structure allows the model to analyze elapsed time between milestones.

Examples include:

- Order-to-invoice duration
- Invoice-to-delivery duration
- Delivery-to-payment duration
- Overall order-to-payment duration

This provides a foundation for identifying fulfillment bottlenecks and process delays.

---

# 8. Bridge Table Design

The relationship between marketing campaigns and products introduces a many-to-many relationship.

Instead of connecting the entities directly, a bridge structure was introduced through:

`fact_promotion_coverage`

Conceptually:

```text
dim_campaign
      │
      │
fact_promotion_coverage
      │
      │
dim_product
```

This provides a controlled relationship between campaigns and products while avoiding ambiguous filtering paths.

---

# 9. Junk Dimension

Low-cardinality order attributes were consolidated into:

`dim_order_flags`

This table contains attributes such as:

- Order channel
- Priority
- Status-related flags

The purpose is to avoid unnecessarily adding multiple small dimensions or repeatedly storing classification fields inside fact tables.

---

# 10. Power Query Transformation

Power Query was used as the primary data preparation layer.

Key transformation activities included:

### Data Cleaning

- Handling missing values
- Standardizing data types
- Removing unwanted records
- Correcting inconsistent values

### Data Normalization

- Splitting fields where necessary
- Reshaping source structures
- Standardizing attributes

### Deduplication

Duplicate records were identified and removed where appropriate to improve model reliability.

### Unpivoting

Wide source structures were reshaped into analytical formats where required.

### Surrogate Keys

Standardized keys were generated to support relationships between fact and dimension tables.

The objective was to move data preparation work into the transformation layer rather than relying on report-level calculations.

---

# 11. Relationship Design

Relationships were designed around the principle of maintaining clear and predictable filter propagation.

Where possible, the model uses:

- One-to-many relationships
- Single-direction filtering
- Shared dimensions
- Explicit bridge structures for many-to-many scenarios

This reduces the likelihood of:

- Circular dependencies
- Ambiguous filter paths
- Unexpected aggregation behavior

The objective was to make the model's filtering behavior predictable for both developers and report users.

---

# 12. Centralized Measures

A dedicated measure table was created:

`_measures`

This provides a centralized location for analytical calculations.

Examples of measure categories include:

- Revenue
- Sales performance
- Inventory metrics
- Fulfillment durations
- Campaign metrics
- Target achievement
- Time intelligence

Centralizing measures reduces duplicated business logic and makes the semantic layer easier to maintain.

---

# 13. Row-Level Security

A dedicated security mapping table was introduced to support regional data access.

Conceptually:

```text
Authenticated User
        ↓
USERPRINCIPALNAME()
        ↓
Security Mapping
        ↓
Assigned Region
        ↓
Authorized Data
```

The model uses dynamic Row-Level Security rather than creating separate roles or reports for every region.

This approach allows one centralized model to support multiple regional stakeholders while controlling which records each user can access.

---

# 14. Data Model Validation

The model was reviewed for several common analytical modeling risks.

### Relationship Validation

Relationships were checked for:

- Correct cardinality
- Appropriate filter direction
- Unexpected many-to-many paths
- Ambiguous relationships

### Data Validation

The model was reviewed for:

- Duplicate keys
- Missing relationships
- Unexpected blanks
- Inconsistent data types
- Invalid lookup values

### Measure Validation

Key measures were compared against expected source values to verify that aggregations behaved correctly across different dimensions and filters.

---

# 15. Model Optimization

Several modeling principles were applied to improve the model's efficiency and maintainability.

### Reduce unnecessary columns

Fields that were not required for analysis were excluded from the analytical model.

### Prefer numeric keys

Surrogate keys were used where appropriate instead of relying heavily on descriptive text fields.

### Centralize business logic

Reusable measures were placed in the `_measures` table rather than recreating the same calculations across reports.

### Separate transformation from visualization

Data preparation was handled primarily through Power Query, while DAX was used for analytical calculations.

---

# 16. Final Architecture

The resulting model can be summarized as:

```text
                    MULTI-DOMAIN MODEL
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
      SALES             INVENTORY          MARKETING
        │                  │                  │
   fact_sales       fact_inventory    fact_campaign_spend
        │                                   │
        │                            fact_promotion_coverage
        │                                   │
        └───────────────┬───────────────────┘
                        │
                CONFORMED DIMENSIONS
                        │
        ┌───────────────┼────────────────┐
        │               │                │
     Customer        Product          Geography
        │               │                │
        └───────────────┼────────────────┘
                        │
                  FULFILLMENT
                        │
                fact_order_process
                        │
                        ↓
                 GOVERNED ANALYTICS
                        │
                  Dynamic RLS
```

The final architecture provides a centralized analytical foundation capable of supporting multiple business processes while maintaining consistent dimensions, reusable measures, and controlled regional access.

---

# 17. Key Learning

The main lesson from this project was that effective Power BI development starts **before the dashboard**.

A well-designed semantic model determines:

- How reliably KPIs can be calculated
- How easily different business domains can be connected
- How predictable filtering will be
- How scalable future reporting will be
- How effectively access can be governed

The dashboard is ultimately the visible layer.

**The data model is the foundation.**
