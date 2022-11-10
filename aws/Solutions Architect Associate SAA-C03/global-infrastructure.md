# Global Infrastructure

## Regions

- A region is a cluster of data centers.
- Most services are scoped to a specific region
- How to Choose an AWS Region
  - **Compliance** with data governance and legal requirements
    - data never leaves a region without explicit permissions
  - **Proximity** to customers
    - reduce latency
  - **Available Services** within a region
    - new services/features are not availble in all regions
  - **Pricing**
    - pricing varies region to refion and is transparent on the pricing page

## Availability Zones

- each region has many availablity zones
- usually 3 zones per region
  - min 2 and max 6
- Each AZ is one or more discrete data centers

## Data Centers

- redundant power, networking, and connectivity
- seperated to be isolated from disasters
- connected with high bandwidth, ultra-low latency networking

## Edge Locations / Points of Presence

- 216 Points of Presence (205 Edge locations & 11 Regional Caches) in 84 cities across 42 countries
- allows lower latency to customers