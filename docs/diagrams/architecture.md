## Product Choice
Telegram
https://web.telegram.org/k/
Telegram is a secure, cloud-based messaging app focused on speed and privacy. It offers end-to-end encrypted chats, large group capacities, and cross-platform sync across all your devices.

## Main components
![Telegram Component Diagram](../diagrams/out/telegram/component-diagram/Component%20Diagram.svg)

[Telegram Component Diagram Code](../diagrams/src/telegram/component-diagram.puml)

### MTProto Gateway (DC Entry)
Acts as the main entry point for client connections. It receives encrypted client requests and routes them to internal Telegram services.

### Auth & Session Service
Handles user authentication, login verification, and manages user sessions across devices.

### Message Handling Service
Processes sending, receiving, and storing messages between users, groups, and chats.

### Media & File Service
Manages uploading, downloading, and storage of media files such as images, videos, and documents.

### Notification/Updates Service
Responsible for delivering real-time updates and notifications about new messages and events to users.

## Data flow

![Telegram Sequence Diagram](../diagrams/out/telegram/sequence-diagram/Sequence%20Diagram.svg)

[Telegram Sequence Diagram Code](../diagrams/src/telegram/sequence-diagram.puml)

### Uploading and sending a media message

When a user selects a photo or media file to send, the client application uploads encrypted file parts to the MTProto Gateway. The gateway forwards file data to the Media Service, which stores file chunks in the Distributed File System and saves file metadata in the database.

After the file upload is completed, the client sends a message request containing the file reference. The request is validated by the Auth Service and processed by the Message Service, which stores the message in the Sharded Chat Database and updates message sequence data.

Once the message is stored, the Message Service publishes an event through the Kafka Event Bus. The Push Service consumes the event, checks notification settings, and sends push notifications through external push providers.

When the recipient opens the application, the client retrieves message updates through the MTProto Gateway. The Media Service provides the media content, and the client displays the received file.

## Deployment

![Telegram Deployment Diagram](../diagrams/out/telegram/deployment-diagram/Deployment%20Diagram.svg)

[Telegram Deployment Diagram Code](../diagrams/src/telegram/deployment-diagram.puml)

Telegram client applications, including mobile apps, desktop apps, and web clients, are deployed on user devices such as smartphones and personal computers. These clients connect to Telegram infrastructure over the internet using the MTProto protocol or HTTPS for bot integrations.

Edge connection components such as MTProto Gateways and Bot API Frontend servers are deployed in Telegram data centers and act as entry points for client and bot traffic. Core backend services including authentication, messaging, channel management, media processing, and push notification services are deployed in containerized environments within compute clusters.

Data and middleware components such as state caching systems, distributed file storage, and sharded chat databases are deployed in dedicated storage and memory clusters to ensure high availability and performance. Event-driven communication between services is handled through an internal event bus cluster.

External ecosystem services such as SMS providers and push notification platforms are integrated through secure HTTP or SMPP communication channels to deliver verification messages and user notifications.

## Assumptions

- I assume Telegram uses load balancing across MTProto gateways to distribute user traffic between multiple data centers and ensure high availability.

- I assume Telegram's distributed file storage system uses data replication and chunk-based storage to improve media delivery performance and fault tolerance.

## Open questions

- How does Telegram synchronize message history and media data between multiple user devices in real time?

- How does Telegram handle failover and data consistency between geographically distributed data centers?