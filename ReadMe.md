### **Test Script README**

#### **1. Purpose**
This script (`api_test_client.py`) is designed to test the file upload API of the Flask-based malware detector. It simulates various client requests based on previously defined testing principles to validate the server-side functionality and responses.

#### **2. Prerequisites**
Before running this test script, please ensure the following:

*   **Install Python Dependencies:**
    You will need the `requests` library to send HTTP requests. If you don't have it installed, run:
    ```bash
    pip install requests
    ```

*   **Start the Backend Server:**
    Your Flask application (the one containing the `upload_file` function) must be running locally. You should start it using the standard Flask command:
    ```bash
    python app.py
    ```
    Ensure the server has started successfully and is listening on `localhost:5000`.

#### **3. How to Run the Test**
1.  In the same directory as `api_test_client.py`, create a dummy file for testing. The script will automatically create a file named `dummy_malware_sample.exe`, so no manual creation is needed.
2.  Make sure your Flask server is running.
3.  Open a new terminal and execute the following command:
    ```bash
    python api_test_client.py
    ```

#### **4. Expected Output**
The script will execute three test cases sequentially and print detailed information for each, including the request being sent, the server's status code, and the content returned by the server. You can use this output to verify if your backend API is working as expected.

---