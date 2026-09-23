# Airline Booking Platform

## Overview

This project is a customer-facing airline booking platform that integrates with the airline's core reservation system (Sabre) through APIs.
The platform allows customers to:

1. Search for available flights
2. Select a flight and fare
3. Enter passenger details
4. Review their booking
5. Make payment
6. Confirm the booking
7. Receive a booking reference

The core reservation system remains the system of record for airline bookings, while this platform provides the customer-facing booking experience and coordinates the API calls required to complete a reservation.

**********************************************************************************************************************************************************************************************************************

# Architecture and Technology Stack

## Architecture Overview

The platform uses a layered architecture where the customer-facing application communicates with Sabre through a backend integration layer.

<div align="center">
<pre>
Customer
   |
   v
React Frontend
   |
   v
Backend API
   |
                              +---------> Payment Gateway
   |
                                   +---------> Application Database
   |
   v
Sabre Integration Layer
   |
   v
Sabre

</pre>
</div>

Sabre remains the **system of record for airline bookings**, while the platform manages the customer journey, booking workflow, payments, validation, and error handling.

## Tech Stack

<div align="center">
<pre>
   ________________________________________________________
| Layer             | Technology                           |
| ----------------- | ------------------------------------ |
| Frontend          | React + TypeScript                   |
| Backend           | .NET/ C#                             |
| API Documentation | Swagger / OpenAPI                    |
| Database          | PostgreSQL                           |
| Cache             | Redis                                |
| API Gateway       | Azure API Management                 |
| WAF/CDN           | Azure Front Door                     |
| Secrets           | Azure Key Vault                      |
| Monitoring        | Azure Monitor + Application Insights |
| Source Control    | Azure Repos                          |
| CI/CD             | Azure DevOps                         |
| API Testing       | Postman                              |
| E2E Testing       | Playwright                           |
____________________________________________________________
</pre>
</div>

## Backend Responsibilities

The backend handles:

* Sabre authentication and token management
* Flight availability search
* Passenger and booking management
* Payment orchestration
* Booking confirmation
* Error handling and reconciliation
* Logging and monitoring

The frontend should **not communicate directly with Sabre**.

<div align="center">
<pre>
Frontend -> Backend -> Sabre
</pre>
</div>

## Security

The platform should use:

* HTTPS
* WAF
* API rate limiting
* Secure secrets management
* Input validation
* Encryption
* Audit logging
* SAST and DAST
* Vulnerability testing


## Deployment Environments

<div align="center">
<pre>
Development
   |
   v
  SIT
   |
   v
  UAT
   |
   v
Production
</pre>
</div>

Each environment should use separate credentials, configuration, databases, and external-system endpoints.

*****************************************************************************************************************************************************************************************************************************
#Booking Flow

The current API collection demonstrates the following basic booking flow:


<div align="center">
<pre>
Authentication
    |
    v
Flight Availability Search
    |
    v
Flight Selection
    |
    v
Passenger and Contact Details
    |
    v
Create Trip/Booking State
    |
    v
Retrieve Booking State
    |
    v
Payment
    |
    v
Commit Booking
    |
    v
Booking Confirmation
</pre>
</div>

The frontend should not call Sabre directly. All core-system interactions should go through our backend/integration layer, which will handle authentication, token management, request/response transformation, booking state and error handling.
The basic flow is:
1.	The customer searches for a flight using origin, destination, travel date and passenger information.
2.	Our backend calls the Sabre availability API and returns a simplified list of flights and fares to the frontend.
3.	When the customer selects a flight, we retain the Sabre journeyKey and fareAvailabilityKey within the booking session.
4.	The customer enters passenger and contact information, which the backend sends to the Sabre Trip API together with the selected journey and fare.
5.	We retrieve the current booking state from Sabre so that the customer can review the flight, passenger details, fare, taxes and total amount before payment.
6.	The customer initiates payment through the required payment gateway.
7.	Once payment succeeds, the corresponding payment is recorded against the Sabre booking transaction.
8.	The backend then calls the Sabre Commit API to finalise the reservation.
9.	If the commit is successful, we capture the confirmed booking/PNR and return it to the customer.
10.	If payment succeeds but the Sabre commit fails, the transaction must not simply be treated as failed. It should be flagged for reconciliation because the customer may already have been charged.
Sabre remains the system of record for the airline reservation. Our platform owns the customer experience and orchestrates the APIs required to create the booking.

***************************************************************************************************************************************************************************************************************************************************
