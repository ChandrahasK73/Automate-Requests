# LinkedIn Connection Automation

This script automates the process of sending connection requests on LinkedIn using Selenium WebDriver. The script reads LinkedIn profile URLs from an Excel file, logs into LinkedIn, and sends connection requests with a personalized note.

## Prerequisites

1. Python 3.x installed on your system.
2. The following Python packages installed:
   - `selenium`
   - `pandas`

You can install the required packages using pip:

```bash
pip install selenium pandas
```

3. Microsoft Edge WebDriver downloaded and the path set correctly in the script.
4. An Excel file (`.xlsx`) containing LinkedIn profile URLs with columns: `name`, `title`, `company_name`, and `person_linkedin_url`.

## Script Explanation

### Imports and Dependencies

```python
import pandas as pd
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.edge.service import Service
from selenium.webdriver.edge.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time
```

### Configuration

- **LinkedIn login credentials**: Your LinkedIn username and password.
- **Excel file path**: Path to the Excel file containing LinkedIn profile URLs.
- **Driver path**: Path to the Microsoft Edge WebDriver.

### Reading LinkedIn Profile URLs

The script reads the LinkedIn profile URLs from an Excel file using the `pandas` library.

```python
excel_file_path = 'path_to_your_excel_file.xlsx'  # Replace with your actual file path
df = pd.read_excel(excel_file_path)
profiles = df['person_linkedin_url'].tolist()
```

### WebDriver Setup

The script sets up the WebDriver for Microsoft Edge in headless mode.

```python
driver_path = 'path_to_your_driver'
service = Service(driver_path)
options = Options()
options.headless = True  # Enable headless mode

driver = webdriver.Edge(service=service, options=options)
wait = WebDriverWait(driver, 10)
```

### Login Function

The `login` function logs into LinkedIn using the provided username and password.

```python
def login(driver, username, password):
    driver.get('https://www.linkedin.com/login')
    time.sleep(2)
    driver.find_element(By.ID, 'username').send_keys(username)
    driver.find_element(By.ID, 'password').send_keys(password)
    driver.find_element(By.XPATH, '//button[@type="submit"]').click()
    time.sleep(5)
```

### Sending Invitation Function

The `send_invitation` function navigates to each profile URL and sends a connection request with a personalized note.

```python
def send_invitation(driver, profile_url, note):
    driver.get(profile_url)
    time.sleep(5)  # Adjust this sleep time as needed
    try:
        driver.execute_script("return document.readyState") == "complete"
        time.sleep(2)

        connect_button = None
        connect_locators = [
            '//button[contains(@aria-label, "Connect")]',
            '//button[contains(text(), "Connect")]',
            '//button[contains(@aria-label, "Connect with")]',
            '//button[contains(@aria-label, "Invite")]',
            '//button[contains(text(), "Invite")]',
        ]

        for locator in connect_locators:
            try:
                connect_button = wait.until(EC.element_to_be_clickable((By.XPATH, locator)))
                if connect_button:
                    print(f"Connect button found with locator: {locator}, clicking...")
                    driver.execute_script("arguments[0].click();", connect_button)
                    time.sleep(2)
                    break
            except:
                print(f"Connect button with locator: {locator} not found, trying next...")

        if not connect_button:
            print(f"Error: Connect button not found for {profile_url}")
            return

        try:
            add_note_button = wait.until(EC.element_to_be_clickable((By.XPATH, '//button[@aria-label="Add a note"]')))
            print("Add note button found, clicking...")
            driver.execute_script("arguments[0].click();", add_note_button)
            time.sleep(2)

            text_area = driver.find_element(By.XPATH, '//textarea[@name="message"]')
            print("Message text area found, entering note...")
            text_area.send_keys(note)

            send_button = driver.find_element(By.XPATH, '//button[@aria-label="Send now"]')
            print("Send button found, clicking...")
            driver.execute_script("arguments[0].click();", send_button)
            time.sleep(2)
        except Exception as e:
            print(f"Add note button not found or error occurred: {e}")
            print("Attempting to send invitation without note...")
            try:
                send_button = driver.find_element(By.XPATH, '//button[@aria-label="Send now"]')
                driver.execute_script("arguments[0].click();", send_button)
                time.sleep(2)
            except Exception as inner_e:
                print(f"Error sending invitation without note: {inner_e}")
    except Exception as e:
        print(f"Error sending invitation to {profile_url}: {e}")
```

### Main Script

The main script logs into LinkedIn and iterates over the list of profile URLs to send connection requests.

```python
login(driver, username, password)
for profile in profiles:
    send_invitation(driver, profile, personal_note)

driver.quit()
```

## Running the Script

1. Update the following variables in the script:
   - `username`: Your LinkedIn username.
   - `password`: Your LinkedIn password.
   - `excel_file_path`: Path to your Excel file.
   - `driver_path`: Path to the Microsoft Edge WebDriver.
   - `personal_note`: Your personalized connection request note.

2. Ensure the required packages are installed:

```bash
pip install selenium pandas
```

3. Run the script:

```bash
python your_script_name.py
```

Replace `your_script_name.py` with the actual name of your script file. The script will log into LinkedIn and send connection requests to the profiles listed in the Excel file with the specified personalized note.