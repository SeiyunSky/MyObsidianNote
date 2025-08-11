## **LSTM Malware Detector Test Document**

### **1. Project Overview**

This document provides a comprehensive test plan for the LSTM (Long Short-Term Memory) based malware detector project, available at freesilly/LSTM-based-malware-detector. This project utilizes a deep learning model to differentiate between malware and benign software by analyzing API call sequences. The purpose of this testing is to ensure the functional correctness of individual modules, the stability of the system, and the performance of the model.

### **2. Test Scope**

#### **2.1. In Scope**

- **Unit Testing:** Independent testing of core functions, including data loading, data preprocessing, and model creation.
    
- **Integration Testing:** Testing the interaction between different modules, such as the main program (main.py) calling utility functions (utils.py) and the model (model.py).
    
- **System Testing:** End-to-end testing of the entire workflow, including model training, model evaluation, and single-file prediction.
    
- **Performance Testing:** Evaluating key model metrics such as accuracy, precision, and recall.
    

#### **2.2. Out of Scope**

- Internal functional testing of third-party libraries (e.g., TensorFlow, Scikit-learn, Pandas).
    
- Exhaustive testing for operating system or hardware compatibility.
    
- Non-functional testing, such as stress testing and security penetration testing.
    

### **3. Test Environment**

Based on the project's requirements.txt file, testing should be conducted in an environment that meets the following specifications:

- **Operating System:** Linux / Windows / macOS
    
- **Python Version:** 3.8+
    
- **Core Dependencies:**
    
    - tensorflow==2.4.1
        
    - scikit-learn==0.24.2
        
    - numpy==1.19.5
        
    - pandas==1.2.4
        
    - matplotlib==3.3.4
        

### **4. Test Strategy & Cases**

#### **4.1. Unit Testing**

Unit tests are designed to verify the correctness of individual functions or components.

|              |                |                                                                          |                                                    |                                                                                                              |
| ------------ | -------------- | ------------------------------------------------------------------------ | -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Test Case ID | Module to Test | Test Description                                                         | Input                                              | Expected Output                                                                                              |
| **UT-001**   | utils.py       | Test if the load_data function can successfully load a CSV dataset.      | A valid CSV file path.                             | Returns a non-empty Pandas DataFrame.                                                                        |
| **UT-002**   | utils.py       | Test if the data_preprocess function correctly processes the data.       | A Pandas DataFrame containing features and labels. | Returns normalized and reshaped NumPy arrays (X, y).                                                         |
| **UT-003**   | model.py       | Test if the create_model function can successfully build the LSTM model. | The shape of the input data (input_shape).         | Returns a compiled TensorFlow Keras model instance. The model's layer structure should match the definition. |

#### **4.2. Integration Testing**

Integration tests verify that different modules work together as expected when combined.

|              |                    |                                                                                                |                                |                                                                         |                                                                                                                 |
| ------------ | ------------------ | ---------------------------------------------------------------------------------------------- | ------------------------------ | ----------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| Test Case ID | Modules            | Test Description                                                                               | Pre-conditions                 | Steps                                                                   | Expected Output                                                                                                 |
| **IT-001**   | main.py + utils.py | Verify that in training mode, main.py successfully calls utils.py to load and preprocess data. | The training dataset is ready. | Run python main.py --mode train and pause before model training begins. | The program runs without errors and successfully prints log messages related to data loading and preprocessing. |
| **IT-002**   | main.py + model.py | Verify that in training mode, main.py successfully calls model.py to create the model.         | The training dataset is ready. | Run python main.py --mode train and pause after the model is compiled.  | The program runs without errors and successfully creates a Keras model instance.                                |

#### **4.3. System Testing**

System tests evaluate the complete, integrated system from an end-user perspective.

|   |   |   |   |   |
|---|---|---|---|---|
|Test Case ID|Test Scenario|Steps|Input|Expected Output|
|**ST-001**|**End-to-End Model Training**|1. Prepare the training dataset train.csv.<br>2. Execute the command python main.py --mode train.|Training data containing API call sequences and labels.|1. The program runs to completion without crashes or errors.<br>2. A model file malware_detector.h5 is generated in the project root directory.<br>3. Training logs are generated in the logs/ directory.<br>4. The training loss and accuracy are displayed on the screen.|
|**ST-002**|**End-to-End Model Evaluation**|1. Ensure the malware_detector.h5 model file exists.<br>2. Prepare the test dataset test.csv.<br>3. Execute the command python main.py --mode test.|Test data with API call sequences and labels, and a pre-trained model.|1. The program runs to completion without crashes or errors.<br>2. Model evaluation metrics, including Accuracy, Precision, Recall, and F1-score, are printed to the screen.|
|**ST-003**|**Single File Prediction**|1. Ensure the malware_detector.h5 model file exists.<br>2. Prepare a CSV file for prediction (e.g., predict_file.csv).<br>3. Execute python main.py --mode predict --file predict_file.csv.|A CSV file with an API call sequence, and a pre-trained model.|1. The program runs to completion without crashes or errors.<br>2. The prediction result, such as "Prediction: Malicious" or "Prediction: Benign", is printed to the screen.|
|**ST-004**|**Invalid Argument Handling**|Execute the command python main.py --mode invalid_mode.|An invalid argument for mode.|The program should handle the error gracefully by displaying a help message or an error prompt instead of crashing.|
|**ST-005**|**Missing File Handling**|Execute the prediction command without the --file argument: python main.py --mode predict.|mode set to predict but the file argument is missing.|The program should prompt the user that a file path is required.|

### **5. Defect Reporting**

Any defects or issues discovered during testing should be reported on the project's GitHub Issues page. A good bug report should include:

- **Title:** A clear and concise summary of the issue.
    
- **Steps to Reproduce:** A detailed, step-by-step description of how to reproduce the problem.
    
- **Expected Behavior:** A description of what should have happened.
    
- **Actual Behavior:** A description of what actually happened, including any error messages or logs.
    
- **Test Environment:** Information about the OS, Python version, and library versions used during the test.