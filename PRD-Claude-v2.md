# Product Requirements Document (PRD)

**Product:** Mechanic Shop Management System
**Version:** 1.0
**Date:** <>

## Table of Contents

1. [Product Overview](#1-product-overview)
2. [Problem Statement](#2-problem-statement)
3. [Goals & Objectives](#3-goals--objectives)
4. [User Roles & Permissions](#4-user-roles--permissions)
5. [Functional Requirements](#5-functional-requirements)
6. [Non-Functional Requirements](#6-non-functional-requirements)
7. [User Stories](#7-user-stories)
8. [Out of Scope](#8-out-of-scope)
9. [Success Metrics](#9-success-metrics)
10. [Assumptions & Dependencies](#10-assumptions--dependencies)
11. [Constraints](#11-constraints)
12. [Acceptance Criteria Summary](#12-acceptance-criteria-summary)
13. [Glossary](#13-glossary)

---

## 1. Product Overview

The Mechanic Shop Management System is a workshop operations platform designed to manage customers, vehicles, repair tasks, work orders, scheduling, labor assignment, and operational monitoring. The system streamlines daily operations for automotive repair shops by providing a centralized platform for tracking work, preventing scheduling conflicts, and maintaining service quality through standardized repair procedures.

## 2. Problem Statement

Mechanic shops currently lack a centralized system to:
- Track customer and vehicle information efficiently
- Manage the complete lifecycle of repair jobs
- Prevent scheduling conflicts and double-booking of technicians
- Monitor operational performance
- Enforce business rules consistently

This system aims to digitize these operations and provide a single source of truth for workshop management.

## 3. Goals & Objectives

- Digitize workshop operations and eliminate paper-based processes
- Reduce scheduling conflicts and double-booking errors
- Improve visibility into work order status for both staff and management
- Enable data-driven operational decisions through real-time metrics
- Standardize repair processes through reusable task templates
- Increase operational efficiency and customer satisfaction

## 4. User Roles & Permissions

| Role | Permissions |
|---|---|
| Manager | Full system access: manage customers, vehicles, repair tasks, work orders, scheduling, and view all reports |
| Labor (Technician) | Limited access: view assigned work orders, update work order status to InProgress or Completed |

## 5. Functional Requirements

### 5.1 Customer & Vehicle Management

As a Manager, I need to:
- Register new customers with contact information
- Update customer details (name, phone, email, address)
- Delete customer records that have no associated work orders
- Add vehicles to customer accounts (make, model, year, VIN)
- Update vehicle information
- Remove vehicles that have no active work orders

**Acceptance Criteria:**
- Cannot delete a customer with existing work orders
- Cannot delete a vehicle with active work orders
- VIN must be unique in the system
- All contact information fields must be validated

### 5.2 Repair Task Catalog

As a Manager, I need to:
- Create repair task templates with name, description, estimated cost, and estimated duration
- Add required parts to repair task templates
- Update repair task details and associated parts
- Delete repair task templates from the catalog

**Business Rules:**
- Repair tasks are reusable templates, not unique instances
- The same task template can be used in multiple work orders
- Deleting a task template does not affect existing work orders that reference it

**Acceptance Criteria:**
- Task templates must have a unique name
- Estimated cost and duration must be positive values
- Can add, update, or remove parts from a task template

### 5.3 Work Order Management

As a Manager, I need to:
- Create new work orders for specific vehicles
- Add repair tasks to work orders (from the task catalog)
- Assign technicians to work orders
- Change scheduled time and service bay assignments
- Delete work orders
- View work order summary (basic information)
- View complete work order details (all tasks, parts, labor, costs)
- Update work order status through its lifecycle

As a Technician, I need to:
- View my assigned work orders
- Mark work orders as InProgress when I start working
- Mark work orders as Completed when I finish

**Work Order Lifecycle:**

```
                 ┌──────────────┐
                 │  Scheduled   │
                 └──────┬───────┘
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
      ┌──────────────┐    ┌──────────────┐
      │  InProgress  │───▶│  Cancelled   │
      └──────┬───────┘    └──────────────┘
             │
             ▼
      ┌──────────────┐
      │  Completed   │
      └──────────────┘
```

**Valid Transitions:**
- Scheduled → InProgress (when technician starts work)
- Scheduled → Cancelled (if customer no-shows or cancels)
- InProgress → Completed (when work is finished)
- InProgress → Cancelled (if customer requests cancellation mid-job)

**Business Rules:**
- Cannot create work order for non-existent vehicle
- Cannot assign technician to overlapping time slots
- Cannot delete work order once it's InProgress
- Work orders automatically cancelled if customer no-shows (15 minutes past scheduled time)
- Only Managers can create, delete, and reschedule work orders
- Technicians can only update status of their assigned work orders

**Acceptance Criteria:**
- New work orders default to Scheduled status
- Invalid state transitions are rejected with clear error messages
- Auto-cancellation triggers within 15 minutes of scheduled time
- Work order displays total estimated cost (sum of all tasks)
- Work order displays total estimated duration

### 5.4 Scheduling

As a Manager, I need to:
- Schedule work orders for specific dates, times, and service bays
- View daily schedule by date
- View schedule by technician
- Reschedule existing work orders
- Cancel scheduled work orders

**Important Constraint:**
- All scheduling operations (create, modify, delete) must be performed from the Schedule Page only
- Other pages can display schedule information but cannot modify it
- This ensures a single point of control and prevents scheduling conflicts

**Business Rules:**
- Cannot schedule overlapping work orders for the same technician
- Cannot schedule overlapping work orders for the same service bay
- Cannot remove a work order from schedule once it's InProgress
- Schedule must show availability gaps for optimal planning

**Acceptance Criteria:**
- Daily schedule view shows all work orders for selected date
- Technician view shows only that technician's assignments
- Visual indicators for scheduling conflicts (if any slip through)
- Can drag-and-drop to reschedule (UI requirement)

### 5.5 Labor Management

As a Manager, I need to:
- View list of all technicians
- See technician availability for scheduling
- Assign technicians to work orders
- Reassign work orders if needed

**Acceptance Criteria:**
- Cannot double-book a technician
- System prevents overlapping assignments
- Technicians can only see their own assigned work orders

### 5.6 Dashboard & Reporting

As a Manager, I need to:
- View work order statistics by date
- See total work orders (all states)
- See count of completed work orders
- See count of in-progress work orders
- See count of cancelled work orders

**Future Enhancement (Out of Current Scope):**
- Revenue tracking
- Technician productivity metrics
- Customer history reports
- Parts usage reports

**Acceptance Criteria:**
- Dashboard updates in real-time or near-real-time
- Can filter statistics by date range
- Statistics are accurate and match work order data

### 5.7 Authentication & Authorization

As a User, I need to:
- Log in with username and password
- Remain logged in for a reasonable session duration
- Log out when finished

**Security Requirements:**
- All pages except login require authentication
- Managers can access all features
- Technicians can only access their assigned work orders
- Passwords must be securely stored (never in plain text)
- Sessions must expire after period of inactivity
- Failed login attempts should be rate-limited

**Acceptance Criteria:**
- Unauthenticated users redirected to login page
- Invalid credentials show clear error message
- Users see only features they have permission to access
- Session persists across page refreshes
- Logout clears session completely

## 6. Non-Functional Requirements

**Performance**
- Page load time < 2 seconds under normal conditions
- API response time < 500ms for standard queries
- System supports at least 10 concurrent users without degradation
- Database queries optimized to prevent slow operations

**Availability**
- System available during business hours (6 AM - 8 PM)
- Planned maintenance communicated in advance
- Data backed up daily

**Usability**
- Intuitive interface requiring minimal training
- Clear error messages that guide users to resolution
- Responsive design (works on desktop and tablet)
- Consistent navigation across all pages

**Security**
- All sensitive data encrypted in transit (HTTPS)
- Passwords must meet minimum complexity requirements
- User sessions expire after 30 minutes of inactivity
- Audit log of all data modifications (who, what, when)

**Data Integrity**
- All database operations must be transactional
- System prevents invalid data entry through validation
- Referential integrity maintained (e.g., can't delete customer with work orders)
- Data recovery possible from daily backups

**Maintainability**
- Code must follow consistent style guidelines
- All APIs documented with clear descriptions
- Business logic separated from data access
- Automated tests for critical workflows

## 7. User Stories

### Epic: Work Order Management

**Story 1: Create Work Order**
- As a Manager
- I want to create a work order for a customer's vehicle
- So that I can schedule and track repair work

Acceptance Criteria:
- Can select customer and their vehicle from dropdown
- Can add multiple repair tasks from catalog
- Can assign technician and service bay
- Can set scheduled date and time
- System validates no scheduling conflicts
- Work order created in Scheduled status

**Story 2: Start Work on Vehicle**
- As a Technician
- I want to mark my assigned work order as InProgress
- So that everyone knows I'm actively working on it

Acceptance Criteria:
- Can only update work orders assigned to me
- Can only transition from Scheduled to InProgress
- Timestamp recorded when status changes
- Manager can see updated status immediately

**Story 3: Handle Customer No-Show**
- As a Manager
- I want work orders automatically cancelled if customer doesn't arrive
- So that I don't waste time waiting and can reschedule the bay

Acceptance Criteria:
- Work order auto-cancelled 15 minutes after scheduled time
- Only applies to work orders still in Scheduled status
- Cancelled work orders marked with "Auto-cancelled: No-show"
- Manager notified of auto-cancellation

### Epic: Scheduling

**Story 4: Prevent Double-Booking**
- As a Manager
- I want the system to prevent overlapping assignments
- So that I don't accidentally double-book technicians or service bays

Acceptance Criteria:
- Cannot assign technician to overlapping time slots
- Cannot assign same service bay to overlapping time slots
- Clear error message shown when conflict detected
- Suggests alternative times or technicians

## 8. Out of Scope

The following features are explicitly not included in this version:

**Phase 1 Exclusions**
- Parts inventory tracking and stock management
- Financial transactions and payment processing
- Customer communication (email/SMS notifications)
- Multiple shop locations
- Mobile application
- Customer self-service portal
- Third-party integrations (QuickBooks, parts suppliers, etc.)

**Rationale**

These features are excluded from the initial release to focus on core workflow management and scheduling capabilities. Advanced features like inventory management and payment processing will be considered for Phase 2 based on user feedback and business needs.

## 9. Success Metrics

**Operational Metrics (6 months post-launch)**
- Reduction in scheduling conflicts by 90%
- Work order tracking accuracy at 100%
- User adoption rate > 80% within first month
- Average time to create work order < 2 minutes
- Customer appointment no-show rate reduced by 30% (due to auto-cancellation feature)
- Technician utilization rate visibility increased to 100%

**User Satisfaction Metrics**
- Manager satisfaction score > 4/5
- Technician ease-of-use score > 4/5
- System uptime > 99% during business hours

## 10. Assumptions & Dependencies

**Assumptions**
- Single shop location with 2-4 service bays
- 3-10 technicians on staff
- Customers pre-registered before work orders created
- Internet connectivity available during business hours
- Users have basic computer literacy

**Dependencies**
- Web server for hosting application
- Database server for data storage
- Modern web browser (Chrome, Firefox, Safari, Edge)
- No integration with external systems required

## 11. Constraints

**Technical Constraints**
- Must be web-based application accessible via standard browsers
- Must support modern web browsers (Chrome, Firefox, Safari, Edge - no IE11)
- Must use RESTful API architecture for future extensibility

**Business Constraints**
- Initial release targets single-shop operation only
- No multi-tenancy required in Phase 1
- English language only
- US date/time formats

**Resource Constraints**
- Development budget: [To be defined]
- Timeline: [To be defined]
- Team size: [To be defined]

## 12. Acceptance Criteria Summary

**Must Have (P0)**
- User authentication and role-based access
- Customer and vehicle CRUD operations
- Repair task catalog management
- Work order lifecycle management
- Scheduling with conflict prevention
- Auto-cancellation of no-shows
- Basic dashboard with statistics

**Should Have (P1)**
- Audit logging of changes
- Advanced search and filtering
- Data export capabilities

**Could Have (P2)**
- Reporting and analytics
- Email notifications
- Calendar view of schedule

**Won't Have (This Version)**
- Payment processing
- Inventory management
- Multi-location support
- Mobile app

## 13. Glossary

| Term | Definition |
|---|---|
| Work Order | A job request to perform repair work on a specific vehicle |
| Repair Task | A reusable template defining a standard service procedure |
| Labor | A technician who performs repair work |
| Service Bay | A physical location where repair work is performed |
| Schedule | The allocation of work orders to specific times and service bays |
| No-Show | When a customer fails to arrive for their scheduled appointment |

---
**Document Version:** 1.0
**Last Updated:** <>
**Document Owner:** Product Management Team
**Stakeholders:** Business Operations, Engineering, Shop Management
