    # K2ViewAgent Java Project

    ## Overview
    The K2ViewAgent Java Project consists of two main classes, `K2ViewAgent` and `AgentSender`, designed to manage and process HTTP requests asynchronously.

    ## K2ViewAgent Class
    ### Description
    The `K2ViewAgent` class serves as the main entry point of the program, managing the flow of HTTP requests and responses.

    ### Features
    - **Manager Thread**: Reads a list of URLs at a specified interval and adds them to a concurrent queue.
    - **Worker Thread**: Polls the queue, sends HTTP GET requests, and handles responses asynchronously.

    ### Usage
    To run the program, execute the `main` method. Ensure that the required environment variables and dependencies are properly configured.

    ## AgentSender Class
    ### Description
    The `AgentSender` class provides asynchronous mechanisms for sending HTTP requests and receiving responses, utilizing concurrent queues and executor services.

    ### Features
    - **Asynchronous Request Handling**: Sends HTTP requests and receives responses asynchronously.
    - **Concurrent Queues**: Utilizes `LinkedBlockingQueue` for managing incoming requests and outgoing responses.
    - **Executor Services**: Employs `ExecutorService` for handling request and response processing.

    ### Usage
    Create an instance of `AgentSender`, and use the `send` method to add requests to the queue. Use the `receive` or `receiveAsync` methods to retrieve responses.

    ## Getting Started
    1. Clone the repository.
    2. Ensure Java and required libraries are installed.
    3. Configure environment variables as needed.
    4. Run the `K2ViewAgent` class.

    ## Dependencies
    - Google Gson: For JSON parsing.
    - SLF4J: For logging.
    - Java HTTP Client: For sending HTTP requests.

    ## Environment Variables
    - `K2VIEW_MANAGER_URL`: URL for fetching the list of URLs to process.

    ## Contributing
    Contributions are welcome. Please follow the standard Java coding conventions and ensure proper testing before submitting pull requests.

    ## License
    This project is licensed under the Apache License - see the [LICENSE](https://www.apache.org/licenses/LICENSE-2.0) file for details.

    ## Contact
    For any inquiries or issues, please open an issue in the repository.

    ---


    This Java code consists of two classes: `K2ViewAgent` and `AgentSender`.

    ### K2ViewAgent Class:
    This class is the main class for the K2ViewAgent program. It has two main threads:
    1. **Manager Thread**: It reads a list of URLs and adds them to a `ConcurrentLinkedQueue` named `requestsQueue` at a specified polling interval.
    2. **Worker Thread**: It polls the `requestsQueue` and sends HTTP GET requests to the URLs using an instance of `AgentSender`. It then receives the responses asynchronously and prints them.

    #### Methods:
    - `main(String[] args)`: The entry point for the program, starts the Manager and Worker threads.
    - `getInboxMessages()`: Gets a list of requests from the manager's inbox via REST API.

    ### AgentSender Class:
    This class provides an asynchronous mechanism for sending HTTP requests and receiving their responses. It uses `LinkedBlockingQueue` to store incoming requests and outgoing responses and `ExecutorService` to handle incoming requests and outgoing responses.

    #### Inner Classes:
    - **Request**: Represents an HTTP request that is sent to the server.
    - **Response**: Represents an HTTP response received from the server.

    #### Methods:
    - `AgentSender(int maxQueueSize)`: Constructor that initializes the `AgentSender` with a specified maximum queue size.
    - `send(Request request)`: Adds a request to the `incoming` queue.
    - `receive(long timeout, TimeUnit timeUnit)`: Polls the `outgoing` queue to retrieve responses synchronously.
    - `receiveAsync(long timeout, TimeUnit timeUnit)`: Asynchronously retrieves the responses from the outgoing queue.
    - `run()`: Continuously retrieves the next request from the incoming queue and sends it as an HTTP request to the remote server.
    - `getMail()`: Retrieves the next mail request from the incoming queue.
    - `sendMail(Request request)`: Sends an HTTP request to the given URL.
    - `getHttpRequest(Request request)`: Constructs an HttpRequest object from the given request object.
    - `sendResponse(String id, HttpResponse<String> response)`: Adds a Response object to the outgoing queue.
    - `sendErrorResponse(String id, Exception e)`: Adds an error Response object to the outgoing queue.
    - `getHeaders(Request request)`: Extracts the headers from the given request object.
    - `close()`: Closes the `AgentSender` by setting the running flag to false, shutting down the executors, and joining the worker thread.

    ### Observations:
    - The `K2ViewAgent` class is responsible for managing and processing HTTP requests, and the `AgentSender` class is responsible for sending and receiving HTTP requests and responses.
    - The `AgentSender` class uses asynchronous programming to handle HTTP requests and responses efficiently.
    - The `K2ViewAgent` class uses concurrent data structures and multithreading to process multiple requests simultaneously.

    If you need a more detailed explanation of any part of the code or have any specific questions, feel free to ask!