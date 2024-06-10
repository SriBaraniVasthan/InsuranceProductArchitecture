# Architecture Challenge for an insurance company

**Problem Statement:** https://github.com/SriBaraniVasthan/InsuranceProductArchitecture/blob/main/Paperless.pdf

**High-Level Architecture for "Paperless" Project in a Layered Architecture:**
![MyProductChallenge](https://github.com/SriBaraniVasthan/InsuranceProductArchitecture/assets/63550126/d8587ebc-b7ad-4534-ba36-be29b10799a8)


User access flow ensures a secure environment for accessing documents within MyProduct's Paperless project. It leverages various components to manage user authentication, authorization, document storage, access control, and user preferences, providing a seamless paperless experience for customers and staff. It promotes modularity and clear separation of concerns between different layers.

**AWS is chosen for some of the reasons below**
1. The system needs to handle potential increases in customer base and document volume with auto scaling capabilities.
2. Secure storage, access control for sensitive policy documents, adherence to relevant data privacy are mandatory for Security and Compliance regulations.

# Presentation Layer:
**MyProduct (Customer Portal):** Client Mobile, Tablet, and Web- Customer logs in to MyProduct using their registered username and password. Customers can view, download, and print policy documents, update paperless preferences, and receive notifications for new documents.
**MyContact (Internal Portal):**  Staff logs in to MyContact using their corporate credentials managed by AWS IAM to access customer information and potentially some policy documents.

**Firewall:** AWS WAF positioned between the public internet and MyProduct's private network (the Paperless system). It filters traffic based on pre-defined rules, allowing only authorized connections, also protects specific web-based attacks like SQL injection and cross-site scripting (XSS).

**Content delivery Network:** AWS cloudfront caches static content like images, files on edge locations around the world, MyProduct users will experience faster loading times for customer portal.

# API Layer:
**API Gateway:** This acts as a central hub for managing APIs, offering functionalities like authentication, authorization, client source identification, and routing for users accessing MyProduct and MyContact.  It performs additional checks based on user roles or claims within a JWT (JSON Web Token) for specific functionalities. If the authorization check fails, the user is denied access, and an appropriate error message is displayed. It can scale to handle traffic spikes, ensuring a smooth user experience even during peak usage periods. 

# Business Logic Layer:
**Message Broker:** When a new document becomes available in the system (e.g., uploaded to S3 storage), the AWS Simple Queue Service on the Paperless system can publish a message through the message broker. This triggers real-time notifications for MyProduct or custom services for document processing. The message broker decouples document upload from notification delivery. The Paperless system can publish the message and continue processing, while the message broker ensures notifications are delivered to subscribed consumers asynchronously. It can handle high volumes of messages efficiently and scales independently of other system components.

#API services:
Manages customer preferences, document access control, notification generation, and document retrieval from the document storage system.
1. **Preference Management microservice:** It stores user preferences related to paperless document delivery. This could include Preferred document delivery method (paper vs. electronic) for different document types (e.g., statements, policy updates) and Communication preferences (email vs. SMS) for notification of new documents.
2. **DocumentCreation microservice:** They are used to automate generation of various documents based on templates and data sources. For example, they can create invoices, contracts, or reports by merging pre-defined templates with customer data.

# Data Layer:
**DynamoDB:** Customer preferences for receiving notifications (email, SMS) would be stored within the Dynamo NoSQL data store associated with the Paperless project. A massive user base with millions of users is supported by DynamoDB's scalability, flexible schema and high write throughput.

**AWS S3 Storage:** AWS S3 scales seamlessly to accommodate any volume of documents, regardless of size making it cost-effective for growing document archives. S3 storage stores policy documents in a secure and accessible format, features like server-side encryption at rest and in transit can be used to protect sensitive policy documents.
S3 offers different storage classes optimized for various access needs. S3 Standard can be used for frequently accessed documents and S3 Glacier for infrequently accessed data, and S3 Glacier Deep Archive for long-term archival are chosen at the lowest cost.

**AWS Key Management Service:** Integrating AWS KMS with S3, MyProduct can implement a robust security encryption for their document storage and the system ensures the user has the decryption password permissions for the document's encryption key.

**AWS Simple Notification Service:** AWS SNS facilitates sending notifications to customers about new documents based on preference. Sending a message to an SQS (Simple Queue Service) queue triggers an AWS Lambda serverless function configured to publish the notification to the SNS topic. Lambda functions might experience a slight delay during the first execution after a period of inactivity with cold start, but it can execute event-driven tasks with variable times effectively scaling demands.

**AWS CloudWatch:** CloudWatch collects and track various metrics related to the infrastructure and services. It can track metrics from backend services involved in document storage, retrieval, and notification, such as resource utilization (CPU, memory), database performance, and Lambda function execution times. Additinally,  it monitors performance metrics like latency, page load times, and error rates. It also set up alarms based on defined thresholds for critical metrics. If a metric exceeds or falls below the threshold (e.g., high API latency), CloudWatch can trigger notifications via SNS (Simple Notification Service) to alert team for investigation and potential intervention.
