# README.md

## K2ViewAgent Java Program

### Overview
The `K2ViewAgent` Java program is designed to manage and process HTTP requests and responses. The program consists of two main classes: `K2ViewAgent` and `AgentSender`.

### Classes

#### 1. K2ViewAgent
This is the main class responsible for running the program. It manages the polling of URLs and the processing of requests.

##### Key Components:
- **Logger:** Utilizes `slf4j` for logging.
- **ConcurrentLinkedQueue:** Holds the requests to be processed.
- **Threads:** Uses two threads, one for managing and another for working on the requests.

#### 2. AgentSender
This class provides an asynchronous mechanism for sending HTTP requests and receiving their responses. It uses `LinkedBlockingQueue` to store incoming requests and an `ExecutorService` to handle outgoing requests.

##### Key Components:
- **Request and Response Records:** Represents HTTP requests sent to the server and HTTP responses received from the server.
- **Blocking Queues:** Stores incoming requests and outgoing responses.
- **Executor Services:** Handles incoming requests and outgoing responses.
- **HttpClient:** Used to send HTTP requests.

### Workflow
1. **K2ViewAgent** starts two threads:
   - **Manager Thread:** Reads a list of URLs and adds them to the `requestsQueue`.
   - **Worker Thread:** Polls the queue and sends the URLs using `AgentSender`.

2. **AgentSender** sends HTTP requests and manages the responses asynchronously, storing incoming requests in one queue and outgoing responses in another.

3. The program continuously retrieves the next request from the incoming queue and sends it as an HTTP request to the remote server. The response is sent to the outgoing queue.

### Usage
To run the program, execute the `main` method in the `K2ViewAgent` class with the appropriate command-line arguments.

### Dependencies
- `com.google.gson.Gson`
- `java.net.http.HttpClient`
- `org.slf4j.Logger`

### Configuration
- The polling interval for the `K2ViewAgent` program is set to 60 seconds.
- The `K2VIEW_MANAGER_URL` environment variable should be set with the manager URL.

### Error Handling
- The program handles various exceptions and logs errors appropriately using the `Logger`.
- In case of an error while sending a request in `AgentSender`, an error response is added to the outgoing queue.

### Notes
- Ensure that all dependencies are correctly imported and the environment variable is properly set before running the program.
- This README provides a high-level overview of the program's structure and functionality, for detailed implementation refer to the code.

### License
TBD
