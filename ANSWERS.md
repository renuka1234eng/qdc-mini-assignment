# ANSWERS.md

## 1. In the current NestJS backend, orders are stored as an in-memory array in `OrdersService`. If this were evolved into a production QDC system with thousands of orders and concurrent users, what changes would you make to the data access layer and service design, and why?

The current implementation stores orders in an in-memory array, which is suitable only for a small demonstration application. In a production QDC environment, data must persist across server restarts and support thousands of orders and concurrent users. I would replace the in-memory storage with a relational database such as PostgreSQL and introduce a dedicated repository or data access layer. This would separate business logic from persistence logic and improve maintainability, scalability, and testing.

The `OrdersService` should focus on business rules while repositories handle database interactions. Frequently queried fields such as order ID, customer ID, status, and creation date should be indexed. Pagination, filtering, and sorting should be implemented to avoid loading large datasets into memory. For performance optimization, caching solutions such as Redis could be introduced. Transactions should also be used for operations that update multiple entities to ensure consistency and prevent race conditions during concurrent updates.

## 2. The `GET /api/orders/:id` endpoint currently returns either an `Order` or an `{ error: string }` object. What are the tradeoffs of this approach, and how would you improve error handling and API design for a real-world POS/ordering API?

Returning either an `Order` object or an error object creates an inconsistent API contract because clients must inspect the response body to determine whether the request succeeded. This increases frontend complexity and makes error handling less predictable. Additionally, returning an error object inside a successful HTTP response can make monitoring and debugging more difficult.

In a real-world POS API, I would use proper HTTP status codes and NestJS exceptions. For example, when an order does not exist, the application should throw a `NotFoundException` and return an HTTP 404 response. Validation failures should return HTTP 400, authorization failures should return HTTP 401 or 403, and unexpected errors should return HTTP 500. A standardized error response containing fields such as `statusCode`, `message`, `timestamp`, and `path` would improve consistency and make API consumption easier.

## 3. The React frontend calls the backend directly with `fetch('http://localhost:3001/api/orders')` inside `useEffect`. If this UI needed to grow into a more complex dashboard (filters, pagination, multiple API calls), how would you structure the frontend code differently to keep it maintainable?

While the current implementation is acceptable for a small application, it becomes difficult to maintain as the number of API calls and features increases. I would move all API communication into dedicated service modules such as `ordersApi.ts` rather than placing fetch logic directly inside components. This keeps components focused on rendering and user interaction.

I would also introduce custom hooks such as `useOrders()` to encapsulate data-fetching logic and state management. Libraries such as React Query or TanStack Query would provide caching, automatic refetching, pagination support, loading states, and error handling. As the application grows, organizing code into feature-based folders and creating reusable UI components would improve maintainability, reduce duplication, and make future enhancements easier to implement.

## 4. Orders in QDC involve multiple business workflows (garment processing, delivery, billing, prepaid packages). Given the current `Order` and `Garment` types in `OrdersService`, what potential edge cases or missing fields do you see, and how would you evolve the domain model to better support real-world laundry operations?

The current domain model is intentionally minimal and does not capture many important aspects of a real laundry management system. Orders do not contain billing information, payment status, customer contact details, pickup information, delivery schedules, pricing details, or employee assignments. Garments also lack attributes such as garment category, special cleaning instructions, stain descriptions, damage notes, and barcode identifiers.

To support real-world operations, I would introduce additional entities such as Customer, Invoice, Payment, Delivery, and Package. Orders should include timestamps for major workflow stages such as received, cleaning started, ready for pickup, and delivered. Status transitions should be validated to prevent invalid workflow changes. Audit logs should track who performed important actions. These additions would provide better traceability, reporting capabilities, and support for advanced features such as prepaid subscriptions and home delivery services.

## 5. Imagine the `OrdersService` methods were initially generated by an AI tool and then lightly edited. What specific risks do you see in relying on AI-generated code in this backend and frontend, and what review/debugging practices would you apply before shipping features to production?

AI-generated code can improve development speed, but it should never be trusted without review. Generated code may contain subtle logic errors, incomplete edge-case handling, security vulnerabilities, inefficient implementations, or incorrect assumptions about business requirements. It may compile successfully while still producing incorrect behavior in production.

Before releasing features, I would perform thorough code reviews and verify that the implementation satisfies all requirements. Unit tests should validate service logic, integration tests should verify API behavior, and frontend tests should confirm user workflows. Static analysis tools, linting, and TypeScript checks should be part of the development process. AI-generated code should be treated as a draft that requires careful human validation before deployment.

## 6. Suppose the QDC dashboard needs to show a real-time view of garments moving through statuses (e.g., a board that updates when a garment becomes `ready`). With the current REST endpoints and architecture, how would you approach adding near real-time updates, and what tradeoffs would you consider?

The simplest approach would be periodic polling, where the frontend requests updated order data every few seconds. Polling is easy to implement using the existing REST endpoints, but it generates unnecessary network traffic and introduces delays between updates and user visibility.

For a more scalable solution, I would introduce WebSockets using NestJS gateways. Whenever a garment status changes, the backend could emit an event to connected clients so that dashboards update immediately. Another possible solution is Server-Sent Events (SSE), which works well when communication is primarily from server to client. The main tradeoff is between simplicity and responsiveness. Polling is easy to implement but less efficient, while WebSockets provide a better user experience at the cost of additional infrastructure, connection management, and operational complexity.
