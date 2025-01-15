# Pizza Retail Analytics Pipeline

A comprehensive project to process, analyze, and visualize pizza retail data using Kafka, Flink SQL, Elasticsearch, and Kibana. This project enables efficient data management, real-time analytics, and actionable managerial insights.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Steps for Data Input and Processing](#steps-for-data-input-and-processing)
3. [Technologies Used](#technologies-used)
4. [Objectives](#Objectives)
5. [Managerial Insights](#managerial-insights)
6. [How to Use the Project](#how-to-use-the-project)
7. [License](#license)

---

## Introduction

This project is designed to streamline data processing by integrating Kafka topics, Flink SQL transformations, and Elasticsearch-based analytics to deliver actionable managerial insights. The steps include transforming raw data into JSON format, feeding it into Kafka, processing it via Flink SQL, and exporting it to Elasticsearch for further visualization.

---

## Steps for Data Input and Processing

### 1. Transforming Data to JSON Format
- **Script**: Use the `convert.py` script to transform raw data into JSON key-value pairs.
- **Command**:
  ```bash
  python $HOME/Documents/fake/convert.py
  ```
- **Input**: Provide the full path of the JSON file (e.g., `xyz.json`).

### 2. Feeding Data into Kafka
- **Start Docker**: Ensure Docker is running.
- **Command**: Use `gen_sample.sh` to pipe the JSON data into a Kafka topic (e.g., `mytest`).
  ```bash
  ./start_flink_nodatagen.sh
  ./gen_sample.sh /home/ashok/Documents/gendata/rev_sample.json | kafkacat -b localhost:9092 -t mytest -K: -P
  ```
- **Validation**: Open a new terminal and use the consumer script to check data flow:
  ```bash
  ./consumer.sh mytest
  ```

### 3. SQL Table Creation and Data Processing

#### Table Definitions
- **Orders Table**:
  ```sql
  CREATE TABLE orders (
      order_details_id BIGINT,
      order_id BIGINT,
      pizza_id STRING,
      quantity INT
  ) WITH (
      'connector' = 'kafka',
      'topic' = 'details',
      'scan.startup.mode' = 'earliest-offset',
      'properties.bootstrap.servers' = 'kafka:9094',
      'format' = 'json'
  );
  ```

- **Pizza Orders Date-Time Table**:
  ```sql
  CREATE TABLE pizza_orders_date_time (
      id BIGINT,
      order_id BIGINT,
      `date` STRING,
      `time` STRING
  ) WITH (
      'connector' = 'kafka',
      'topic' = 'orders_1',
      'scan.startup.mode' = 'earliest-offset',
      'properties.bootstrap.servers' = 'kafka:9094',
      'format' = 'json'
  );
  ```

- **Pizzas Table**:
  ```sql
  CREATE TABLE pizzas (
      id BIGINT,
      pizza_id STRING,
      pizza_type_id STRING,
      size STRING,
      price DECIMAL(4, 2)
  ) WITH (
      'connector' = 'kafka',
      'topic' = 'pizzas',
      'scan.startup.mode' = 'earliest-offset',
      'properties.bootstrap.servers' = 'kafka:9094',
      'format' = 'json'
  );
  ```

- **Pizza Types Table**:
  ```sql
  CREATE TABLE pizza_types (
      id BIGINT,
      pizza_type_id STRING,
      name STRING,
      category STRING,
      ingredients STRING
  ) WITH (
      'connector' = 'kafka',
      'topic' = 'pizza_types',
      'scan.startup.mode' = 'earliest-offset',
      'properties.bootstrap.servers' = 'kafka:9094',
      'format' = 'json'
  );
  ```

#### Merged View
- **Create View**:
  ```sql
  CREATE VIEW merged_pizza_orders AS
  SELECT 
      o.order_details_id,
      o.order_id,
      o.pizza_id,
      o.quantity,
      p.price,
      p.size,
      pt.name AS pizza_name,
      pt.category AS pizza_category,
      pt.ingredients AS pizza_ingredients,
      pdt.`date`,
      pdt.`time`
  FROM orders o
  JOIN pizza_orders_date_time pdt ON o.order_id = pdt.order_id
  JOIN pizzas p ON o.pizza_id = p.pizza_id
  JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id;
  ```

#### Export to Elasticsearch
- **Create Elasticsearch Table**:
  ```sql
  CREATE TABLE pizza_order_summary_es (
      order_details_id BIGINT,
      order_id BIGINT,
      pizza_id STRING,
      quantity INT,
      price DECIMAL(4, 2),
      size STRING,
      pizza_name STRING,
      pizza_category STRING,
      pizza_ingredients STRING,
      `date` STRING,
      `time` STRING,
      proctime TIMESTAMP(3)
  ) WITH (
      'connector' = 'elasticsearch-7',
      'hosts' = 'http://elasticsearch:9200',
      'index' = 'pizza_order_summary',
      'document-id.key-delimiter' = '-',
      'format' = 'json',
      'sink.bulk-flush.max-actions' = '1'
  );
  ```

- **Insert Data**:
  ```sql
  INSERT INTO pizza_order_summary_es
  SELECT 
      o.order_details_id,
      o.order_id,
      o.pizza_id,
      o.quantity,
      p.price,
      p.size,
      pt.name AS pizza_name,
      pt.category AS pizza_category,
      pt.ingredients AS pizza_ingredients,
      pdt.`date`,
      pdt.`time`,
      PROCTIME() AS proctime
  FROM orders o
  JOIN pizza_orders_date_time pdt ON o.order_id = pdt.order_id
  JOIN pizzas p ON o.pizza_id = p.pizza_id
  JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id;
  ```

---

## Technologies Used

- **Programming Languages**: Python, SQL
- **Data Streaming**: Kafka
- **Data Processing**: Apache Flink
- **Analytics and Visualization**: Elasticsearch and Kibana

---
## Objectives

The primary objective of this project is to analyze and interpret pizza retail sales data to uncover actionable managerial insights, enabling businesses to optimize operations, improve customer satisfaction, and achieve sustainable growth. Specifically, the project aims to:

1. **Enhance Sales Performance**:
   - Monitor sales trends to identify areas of growth and concern.
   - Leverage seasonality insights to optimize inventory, staffing, and promotional strategies.
   - Understand product-level performance to prioritize high-performing categories like classic pizzas.

2. **Understand Customer Preferences**:
   - Analyze customer preferences for pizza sizes and categories to tailor offerings.
   - Determine pricing sensitivity and develop competitive pricing strategies to maximize revenue.

3. **Improve Operational Efficiency**:
   - Use ingredient consumption data to improve inventory management and reduce waste.
   - Optimize order fulfillment processes based on customer ordering patterns.

4. **Adapt to Competitive Dynamics**:
   - Identify factors contributing to declining sales and devise strategies to counteract competitive pressures.
   - Stay informed about customer needs and industry trends through continuous market research.

5. **Support Data-Driven Decision-Making**:
   - Provide a robust data pipeline using tools like Kafka, Flink, Elasticsearch, and Kibana to enable real-time analytics and visualization.
   - Equip decision-makers with insights to make informed business decisions, from marketing to supply chain management. 

This objective is supported by a scalable analytics pipeline and detailed managerial recommendations derived from comprehensive data analysis.

---

## Dashboard 
![1](https://github.com/user-attachments/assets/8e41ad22-8814-4809-8287-4e63fd93e7e3)

![2](https://github.com/user-attachments/assets/80a5c750-b645-4ff6-af65-41dd1713118d)
file:///home/ashok/Pictures/3.jpg

## Managerial Insights

### Overall Insights & Implications

#### Sales Performance & Trends
- **Sales Volume**: The overall sales volume is significant, indicating a strong market presence. However, the downward trend in sales across categories is a concerning factor. 
- **Sales Seasonality**: The "Pizza Sales Over Time" chart reveals potential seasonality patterns in sales. Identifying these patterns allows for proactive planning of inventory, staffing, and promotional activities during peak and off-peak periods.
- **Pizza Type Popularity**: Classic pizzas consistently outsell other types, suggesting a strong customer base for this category. This information can guide product development and marketing efforts.

#### Customer Preferences & Behavior
- **Pizza Size Preference**: Customers exhibit a preference for Large (L) and Medium (M) sizes, with a significant proportion opting for Large. This insight is crucial for inventory management and production planning.
- **Pricing Sensitivity**: The observation that larger sizes generally command higher prices suggests that customers are willing to pay a premium for larger quantities. However, it's important to monitor price elasticity to ensure that pricing strategies remain competitive.

#### Operational Efficiency
- **Ingredient Usage**: The information on ingredient usage is critical for inventory management and cost control. Optimizing ingredient ordering and storage can significantly impact profitability.
- **Order Fulfillment**: Analyzing average pizza quantity sold per order can inform order fulfillment processes and kitchen operations.

#### Competitive Landscape
- The downward trend in sales across categories may indicate increased competition from other food delivery services or local restaurants. Conducting market research to understand competitive pressures is crucial.

### Recommendations
- **Address Sales Decline**: Investigate the factors contributing to the decline in sales, such as increased competition, pricing issues, or changes in customer preferences. Implement strategies to address these factors, such as targeted promotions, competitive pricing, and menu innovation.
- **Leverage Customer Preferences**: Tailor product offerings and marketing campaigns to cater to customer preferences for size and pizza types.
- **Optimize Operations**: Streamline inventory management, order fulfillment, and production processes to improve efficiency and reduce costs.
- **Market Research & Analysis**: Conduct ongoing market research to monitor competitive trends, customer preferences, and industry dynamics. This information will be vital for making informed business decisions.
- **Data-Driven Decision Making**: Continuously analyze sales data to identify trends, patterns, and areas for improvement. Utilize data-driven insights to inform all aspects of the business, from product development to marketing and operations.

Overall, the dashboard provides valuable insights into the pizza retail business. By leveraging these insights and implementing data-driven strategies, the business can improve its performance, enhance customer satisfaction, and achieve sustainable growth.

---
## How to Use the Project

1. Clone the repository.
2. Follow the steps to process data and integrate it into the pipeline.
3. Access Kibana for visual insights.

---

## Contributing

Contributions are welcome! Please follow the contributing guidelines to submit issues or pull requests.

---

## License

This project is licensed under the MIT License. See the LICENSE file in the repository for more details.
