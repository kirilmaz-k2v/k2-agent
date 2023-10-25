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
This project is licensed under the MIT License - see the [LICENSE](LICENSE.txt) file for details.

## Contact
For any inquiries or issues, please open an issue in the repository.

---

Feel free to modify or add any sections as needed to better suit the project's requirements or documentation standards.