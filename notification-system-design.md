# Notification System Design

## Stage 1

### Overview

This document outlines the REST API specification for the Campus Notification Service. The service enables students to receive timely updates regarding placement drives, campus events, examination results, and other academic announcements.

The API follows RESTful principles, exchanges data in JSON format, and uses standard HTTP methods and status codes. Real-time notification delivery is achieved through Socket.IO over WebSocket connections.

---

# Notification Resource

```json
{
  "id": "uuid",
  "type": "Placement",
  "title": "Placement Drive",
  "message": "TCS Corporation is hiring.",
  "isRead": false,
  "createdAt": "2026-04-22T17:51:18Z"
}
```

---

# Standard HTTP Headers

### Request Headers

```
Content-Type: application/json
Accept: application/json
```

### Response Headers

```
Content-Type: application/json
```

---

# 1. Get All Notifications

### Endpoint

```
GET /api/notifications
```

### Description

Returns a paginated list of notifications associated with the authenticated student.

### Query Parameters

| Parameter | Type    | Required | Description                                                 |
| --------- | ------- | -------- | ----------------------------------------------------------- |
| page      | Integer | No       | Page number for pagination                                  |
| limit     | Integer | No       | Maximum number of records per page                          |
| type      | String  | No       | Filter notifications by category (Placement, Event, Result) |

### Sample Request

```
GET /api/notifications?page=1&limit=10&type=Placement
```

### Success Response (200 OK)

```json
{
  "success": true,
  "page": 1,
  "limit": 10,
  "total": 250,
  "notifications": [
    {
      "id": "1",
      "type": "Placement",
      "title": "Placement Drive",
      "message": "Amazon hiring",
      "isRead": false,
      "createdAt": "2026-04-22T17:51:18Z"
    }
  ]
}
```

---

# 2. Get a Single Notification

### Endpoint

```
GET /api/notifications/{id}
```

### Description

Retrieves the details of a specific notification using its unique identifier.

### Example Request

```
GET /api/notifications/123
```

### Success Response

```json
{
  "success": true,
  "notification": {
    "id": "123",
    "type": "Result",
    "title": "Semester Result",
    "message": "Results published",
    "isRead": false,
    "createdAt": "2026-04-22T17:51:18Z"
  }
}
```

---

# 3. Create a Notification

### Endpoint

```
POST /api/notifications
```

### Description

Creates and stores a new notification that can be delivered to the intended students.

### Request Body

```json
{
  "type": "Placement",
  "title": "Placement Drive",
  "message": "Microsoft is hiring."
}
```

### Success Response (201 Created)

```json
{
  "success": true,
  "message": "Notification created successfully.",
  "notificationId": "12345"
}
```

---

# 4. Mark a Notification as Read

### Endpoint

```
PATCH /api/notifications/{id}/read
```

### Description

Updates the read status of a specific notification.

### Success Response

```json
{
  "success": true,
  "message": "Notification status updated successfully."
}
```

---

# 5. Mark All Notifications as Read

### Endpoint

```
PATCH /api/notifications/read-all
```

### Description

Marks every notification belonging to the authenticated student as read.

### Success Response

```json
{
  "success": true,
  "message": "All notifications have been marked as read."
}
```

---

# 6. Delete a Notification

### Endpoint

```
DELETE /api/notifications/{id}
```

### Description

Removes a notification from the system.

### Success Response

```json
{
  "success": true,
  "message": "Notification removed successfully."
}
```

---

# HTTP Status Codes

| Status Code | Description                    |
| ----------- | ------------------------------ |
| 200         | Request completed successfully |
| 201         | Resource created successfully  |
| 400         | Invalid request or parameters  |
| 404         | Notification not found         |
| 500         | Internal server error          |

---

# Error Response Format

```json
{
  "success": false,
  "message": "Notification not found."
}
```

---

# Real-Time Notification Delivery

### Communication Protocol

Socket.IO (WebSocket)

### Notification Workflow

1. The student logs into the application.
2. The client establishes a Socket.IO connection with the server.
3. The server maintains an active socket session for the authenticated student.
4. An administrator creates a new notification.
5. The notification is saved to the database.
6. The server immediately emits the notification to the intended recipient(s).
7. Connected students receive the notification instantly without refreshing the application.

### Socket Event

```
new-notification
```

### Event Payload

```json
{
  "id": "123",
  "type": "Placement",
  "title": "Placement Drive",
  "message": "Amazon hiring",
  "createdAt": "2026-04-22T17:51:18Z"
}
```

---

# API Reference

| Method | Endpoint                     | Description                    |
| ------ | ---------------------------- | ------------------------------ |
| GET    | /api/notifications           | Retrieve all notifications     |
| GET    | /api/notifications/{id}      | Retrieve a notification by ID  |
| POST   | /api/notifications           | Create a new notification      |
| PATCH  | /api/notifications/{id}/read | Mark a notification as read    |
| PATCH  | /api/notifications/read-all  | Mark all notifications as read |
| DELETE | /api/notifications/{id}      | Delete a notification          |

---

# API Design Guidelines

* Resource URLs use plural nouns to maintain REST consistency.
* HTTP methods follow RESTful conventions.
* All request and response bodies are transmitted in JSON format.
* Every notification is identified using a UUID.
* Timestamps follow the ISO 8601 date-time standard.
* Responses follow a consistent structure to simplify frontend integration and error handling.

# Stage 2

# Database Design

## Database Selection

For the Campus Notification Platform, **MongoDB** is selected as the primary database for storing notification-related data.

### Why MongoDB?

MongoDB is an ideal choice for notification systems because it provides:

* A document-oriented storage model using BSON (JSON-like documents).
* High-speed write operations, making it suitable for applications that generate notifications frequently.
* Horizontal scalability through sharding, allowing the system to handle future growth efficiently.
* A flexible schema that supports adding new fields without modifying existing documents.
* Native integration with Node.js applications using the Mongoose ODM.

---

# Database Collections

## students

```json
{
  "_id": ObjectId,
  "name": "GUDE GOPI KRISHNA",
  "email": "23pa1a4543@vishnu.edu.in",
  "createdAt": ISODate("2026-04-22T17:51:18Z")
}
```

---

## notifications

```json
{
  "_id": ObjectId,
  "studentId": ObjectId,
  "type": "Placement",
  "title": "Placement Drive",
  "message": "Microsoft is hiring",
  "isRead": false,
  "createdAt": ISODate("2026-04-22T17:51:18Z")
}
```

---

# Data Relationship

Each student may receive multiple notifications throughout the lifecycle of the application.

```text
Student
   │
   ├── Notification
   ├── Notification
   └── Notification
```

The relationship between the collections is maintained using the `studentId` field stored within each notification document.

---

# Indexing Strategy

To optimize database performance, the following indexes should be created.

```javascript
db.notifications.createIndex({ studentId: 1 })

db.notifications.createIndex({ studentId: 1, isRead: 1 })

db.notifications.createIndex({ type: 1 })

db.notifications.createIndex({ createdAt: -1 })
```

These indexes improve query efficiency by enabling MongoDB to quickly locate matching documents instead of performing full collection scans.

---

# Challenges with Large Datasets

As the number of notifications increases, the application may encounter the following challenges:

* Query execution becomes slower when indexes are missing.
* Retrieving large volumes of notifications increases memory usage.
* Sorting notifications by creation date requires additional processing.
* Overall query latency increases as the collection grows.

---

# Performance Optimization

The following strategies help maintain efficient database performance as the system scales:

* Create indexes on frequently queried and filtered fields.
* Use `skip()` and `limit()` to implement pagination and reduce the number of records retrieved.
* Fetch only the required fields by using projections.
* Archive older notifications into a separate collection to keep the primary collection lightweight.
* Enable sharding when notification traffic becomes significantly large.
* Utilize Redis caching for frequently accessed notifications to minimize database load.

---

# MongoDB Operations

## Retrieve Notifications

```javascript
db.notifications
.find({ studentId: ObjectId(studentId) })
.sort({ createdAt: -1 })
.skip(0)
.limit(10)
```

This query retrieves a paginated list of notifications for a specific student, ordered from the most recent to the oldest.

---

## Retrieve a Single Notification

```javascript
db.notifications.findOne({
    _id: ObjectId(notificationId)
})
```

This operation fetches a notification using its unique identifier.

---

## Create a Notification

```javascript
db.notifications.insertOne({
    studentId: ObjectId(studentId),
    type: "Placement",
    title: "Placement Drive",
    message: "Microsoft is hiring",
    isRead: false,
    createdAt: new Date()
})
```

This operation inserts a new notification into the collection with the current timestamp.

---

## Mark a Notification as Read

```javascript
db.notifications.updateOne(
    {
        _id: ObjectId(notificationId)
    },
    {
        $set: {
            isRead: true
        }
    }
)
```

This operation updates the read status of a single notification.

---

## Mark All Notifications as Read

```javascript
db.notifications.updateMany(
    {
        studentId: ObjectId(studentId)
    },
    {
        $set: {
            isRead: true
        }
    }
)
```

This operation marks every notification belonging to the specified student as read.

---

## Delete a Notification

```javascript
db.notifications.deleteOne({
    _id: ObjectId(notificationId)
})
```

This operation removes a notification document from the collection using its unique identifier.

# Stage 3

# Query Evaluation and Optimization

## Existing Query

```sql
SELECT *
FROM notifications
WHERE student_id = 1042
AND is_read = FALSE
ORDER BY created_at ASC;
```

### Query Analysis

The query is functionally correct. It retrieves all unread notifications for the student with ID **1042** and sorts them in ascending order based on the notification creation time.

Although the query produces the expected results, its performance may degrade significantly as the notification table grows to millions of records.

---

# Performance Considerations

## 1. Selecting All Columns

Using `SELECT *` causes the database to return every column from the matching rows.

In most applications, the user interface displays only a few attributes such as the notification title, message, category, and creation time. Retrieving unnecessary columns increases:

* Disk I/O
* Memory usage
* Network bandwidth
* Overall query execution time

---

## 2. Missing Indexes

If the filtering columns are not indexed, the database performs a full table scan to locate matching notifications.

As the size of the notifications table increases, this sequential scan becomes increasingly expensive and leads to slower response times.

---

## 3. Sorting Cost

The query sorts the results using the `created_at` column.

Without an index that supports the sorting order, the database must perform an additional sorting operation after filtering the records, increasing execution time for large datasets.

---

# Optimized Query

Instead of retrieving every column, fetch only the fields required by the application.

```sql
SELECT
    id,
    title,
    message,
    type,
    created_at
FROM notifications
WHERE student_id = 1042
AND is_read = FALSE
ORDER BY created_at ASC;
```

By limiting the selected columns, the database reads and transfers less data, resulting in improved query performance and reduced resource utilization.

---

# Recommended Index

A composite index that matches both the filtering conditions and sorting order provides the most effective optimization.

```sql
CREATE INDEX idx_notifications_student_read_created
ON notifications(student_id, is_read, created_at);
```

This index enables the database to:

* Quickly locate notifications for a specific student.
* Efficiently filter unread notifications.
* Return results in chronological order without requiring an additional sort operation.

As a result, both lookup time and query execution are significantly improved.

---

# Time Complexity Analysis

## Without an Index

Without appropriate indexing, the database performs a full table scan before sorting the results.

* Searching: **O(n)**
* Sorting: **O(n log n)**

As the number of notifications grows, query performance decreases considerably.

---

## With the Composite Index

Using the recommended composite index allows the database to navigate directly to the required records.

* Index Lookup: **O(log n)**
* Retrieving Matching Records: Proportional to the number of matching rows

Since the index already maintains the required order, an additional sorting step is usually unnecessary.

---

# Should Every Column Be Indexed?

No.

Although indexes improve read performance, indexing every column is not considered a good database design practice.

Creating excessive indexes introduces several drawbacks:

* Increased storage requirements.
* Slower `INSERT`, `UPDATE`, and `DELETE` operations because every index must also be updated.
* Many indexes may remain unused by the query optimizer.
* Higher maintenance and backup costs.

A better approach is to create indexes only on columns that are frequently used for filtering, sorting, searching, or joining.

---

# SQL Query

Retrieve the IDs of students who received **Placement** notifications during the last seven days.

```sql
SELECT DISTINCT student_id
FROM notifications
WHERE type = 'Placement'
AND created_at >= NOW() - INTERVAL 7 DAY;
```

---

# Recommended Index for the Above Query

The following composite index improves the performance of the query.

```sql
CREATE INDEX idx_notifications_type_created
ON notifications(type, created_at);
```

This index allows the database to efficiently:

* Filter notifications by the **Placement** category.
* Retrieve only notifications created within the last seven days.
* Minimize the number of records scanned during query execution.

---

# Summary

* The original query is functionally correct but may become inefficient on large datasets.
* Selecting only the required columns reduces memory usage and network overhead.
* Composite indexes significantly improve filtering and sorting performance.
* Proper indexing reduces query execution time from a full table scan to efficient index lookups.
* Avoid indexing every column; instead, create indexes based on common filtering and sorting patterns.
* Optimized queries and indexing strategies ensure better scalability as the notification system grows.

# Stage 4

# Performance Enhancement Strategy

As the notification platform scales to support a growing number of users and notifications, relying exclusively on the database for every request can lead to higher response times and increased backend load. To improve overall system performance, I would integrate **Redis** as a caching layer and **Socket.IO** for real-time notification delivery.

---

# Why Use Caching?

Caching plays a crucial role in improving application performance by storing frequently accessed data in memory.

Instead of querying the database for every request, the application can retrieve cached data from Redis, resulting in much faster response times.

Redis is particularly suitable for this purpose because it is an in-memory key-value store capable of handling high-speed read and write operations with very low latency.

---

# Cache Retrieval Workflow

The notification retrieval process follows the steps below:

1. A user requests their notification list.
2. The application checks Redis for the requested data.
3. If the data exists (**Cache Hit**), Redis immediately returns the notifications.
4. If the data is unavailable (**Cache Miss**), the application retrieves the notifications from the MySQL database.
5. The retrieved data is stored in Redis before being returned to the client.

Serving frequently requested data directly from memory significantly reduces database queries and improves application responsiveness.

---

# Cache Synchronization

To ensure users always receive the latest information, the cache must remain consistent with the database.

Whenever a notification is created, updated, marked as read, or deleted, the corresponding cached data should either be refreshed or invalidated.

A common cache invalidation strategy is:

1. Apply the required changes to the MySQL database.
2. Remove the affected user's cached notifications from Redis.
3. During the next request, fetch the updated data from MySQL and repopulate the Redis cache.

This strategy keeps the cache synchronized with the database while maintaining high performance.

---

# Real-Time Notification Delivery

To deliver notifications instantly without requiring users to refresh the application, the system uses **Socket.IO**, which enables bidirectional communication over WebSocket connections.

### Notification Workflow

1. The user logs into the application.
2. The client establishes a persistent Socket.IO connection with the server.
3. When a new notification is created, it is first stored in the MySQL database.
4. The corresponding Redis cache for the user is invalidated.
5. The server emits a `new-notification` event through Socket.IO.
6. Connected clients receive the notification immediately and update the user interface in real time.

---

# Benefits of Using Redis and Socket.IO Together

Although Redis and Socket.IO serve different purposes, they complement each other to improve the overall system architecture.

* **Redis** minimizes database access by serving frequently requested data directly from memory.
* **Socket.IO** enables the server to push notifications instantly through persistent client-server connections.

Using both technologies together provides several benefits:

* Faster response times
* Reduced database workload
* Improved scalability for high-traffic environments
* Instant notification delivery without page refreshes
* Enhanced user experience through real-time updates

---

# Summary

Integrating Redis and Socket.IO significantly improves the performance and scalability of the notification platform. Redis accelerates data retrieval through in-memory caching, while Socket.IO ensures users receive notifications instantly. Together, these technologies reduce backend load, improve response times, and deliver a seamless real-time experience as the application continues to grow.

# Stage 5

# Asynchronous Notification Processing

To improve the scalability and reliability of the notification platform, I would introduce a message broker such as **RabbitMQ** between the Notification API and the notification delivery service.

Instead of sending notifications immediately after an API request, the application first stores the notification in the MySQL database and then publishes a message to RabbitMQ. Background worker processes consume messages from the queue and handle notification delivery independently, allowing the API to respond without waiting for the notification to be delivered.

---

# Notification Processing Workflow

The notification delivery process consists of the following steps:

1. Save the notification in the MySQL database.
2. Publish a notification message to RabbitMQ.
3. A background worker retrieves the message from the queue.
4. The worker delivers the notification to the intended user through Socket.IO.
5. If the delivery fails, the worker retries the operation based on a predefined retry policy.
6. Messages that continue to fail after all retry attempts are moved to a **Dead Letter Queue (DLQ)** for further analysis or manual processing.

By separating notification delivery from the API, client requests are completed quickly while notification processing continues asynchronously in the background.

---

# Benefits of Asynchronous Processing

Using RabbitMQ as a message broker offers several architectural advantages:

* API responses are faster because notification delivery occurs asynchronously.
* Background workers can be scaled independently to handle increasing notification traffic.
* Retry mechanisms improve the reliability of notification delivery.
* Failed messages are preserved in a Dead Letter Queue instead of being lost.
* The message queue absorbs sudden traffic spikes, preventing excessive load on the backend services.

---

# System Architecture

```text id="wz3m9r"
Client
   │
   ▼
Notification API
   │
   ▼
MySQL Database
   │
   ▼
RabbitMQ Exchange / Queue
   │
   ▼
Background Worker
   │
   ▼
Socket.IO Server
   │
   ▼
Connected Users
```

---

# Summary

Integrating RabbitMQ enables the notification platform to process notifications asynchronously, resulting in faster API responses, improved fault tolerance, and better scalability. Background workers manage notification delivery independently, while retry mechanisms and the Dead Letter Queue ensure reliable message handling even when delivery failures occur.

# Stage 6

# Priority-Based Notification Display

To enhance the usability of the notification dashboard, notifications should be displayed based on their priority instead of relying solely on their creation time. This approach ensures that the most important updates are presented first, allowing students to quickly access critical information.

---

# Notification Priority Levels

Notifications are displayed according to the following priority order:

1. **Placement**
2. **Result**
3. **Event**

Notifications with a higher priority are always shown before those with a lower priority.

---

# SQL Query

```sql id="6n0vxm"
SELECT *
FROM notifications
WHERE student_id = ?
ORDER BY
    CASE
        WHEN type = 'Placement' THEN 1
        WHEN type = 'Result' THEN 2
        WHEN type = 'Event' THEN 3
    END,
    created_at DESC;
```

The query first sorts notifications based on their assigned priority and then orders notifications within the same category from the newest to the oldest.

---

# Benefits of Priority-Based Sorting

Implementing priority-based ordering provides several advantages:

* Placement notifications are always displayed at the top of the notification list.
* Important academic updates, such as examination results, receive higher priority than general event announcements.
* Notifications within each category are sorted in descending order of creation time, making the latest updates easier to identify.
* Students can quickly access critical information without manually searching through all notifications.

---

# Summary

Priority-based notification sorting improves the overall user experience by displaying the most important updates first. By combining category-based prioritization with chronological ordering, the notification system presents information in a clear, organized, and user-friendly manner, ensuring that students never miss essential updates.
