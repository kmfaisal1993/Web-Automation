# Web-Automation
1. Write 20 Test Cases from different modules (including CRUD operation).
2. For web Automation you can use the selenium/cypress library.
3. Try to use POM/BDD framework.
4. Generate Allure report.

Code:
import os
import pandas as pd
import pytest
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager
import allure

# Get the directory of the Excel file using openpyxl
excel_file_path = 'C:/Users/Hi/Desktop/Test2/CRUD.xlsx'
excel_directory = os.path.dirname(excel_file_path)

# Create a fixture for the browser driver
@pytest.fixture
def driver_fixture():
    options = Options()
    options.add_experimental_option("detach", True)
    options.add_argument("--incognito")

    driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=options)
    yield driver
    driver.quit()

# Read test cases from the Excel sheet using pandas
df = pd.read_excel(excel_file_path)

# Check if the column headers 'Test Case ID' and 'Steps' exist in the DataFrame
if 'Test Case ID' not in df.columns or 'Steps' not in df.columns:
    print("Error: 'Test Case ID' or 'Steps' column not found in the Excel sheet.")
else:
    # Create a list to store test cases
    test_cases = []

    # Iterate through the test cases in the Excel sheet
    for index, row in df.iterrows():
        test_case_id = row['Test Case ID']
        test_description = row['Steps']

        test_cases.append((test_case_id, test_description))

    # Define the pytest test function using the @pytest.mark.parametrize decorator
    @pytest.mark.parametrize("test_case_id, test_description", test_cases)
    @allure.title("Run Test Case: {test_case_id}")
    def test_run_test_case(driver_fixture, test_case_id, test_description):
        driver = driver_fixture
        try:
            # Navigate to the website
            driver.get("https://hishabee.business/")

            allure.attach(driver.get_screenshot_as_png(), name=f'Screenshot_Test_{test_case_id}', attachment_type=allure.attachment_type.PNG)
        except Exception as e:
            # Handle any exceptions or errors
            print(f"Test {test_case_id} failed: {e}")

allure_results_directory = os.path.join(excel_directory, 'allure-results')
os.makedirs(allure_results_directory, exist_ok=True)
pytest.main(['-v', '--alluredir', allure_results_directory])
print(f"Allure report will be saved in: {allure_results_directory}")

# Run in Terminal to start test
# pytest --alluredir=C:\Users\Hi\Desktop\Test2\allure-results Assignment2.py
# npm install --save-dev allure-commandline
# npx allure serve allure-results

