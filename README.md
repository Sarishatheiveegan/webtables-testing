
# Selenium Web Tables Testing

## Overview

This automates web table testing using **Python and Selenium WebDriver**. It demonstrates how to locate HTML tables, extract employee details, search records, retrieve email addresses, identify the highest Due amount, verify website links, and count table rows.

**Practice Website:** https://assertqa.com/practice/webtables

## Technologies Used

* Python
* Selenium WebDriver
* Google Chrome
* Visual Studio Code

## Test Cases

| Test Case | Task                                          | Expected Result                             |
| --------- | --------------------------------------------- | ------------------------------------------- |
| TC01      | Print all column headings                     | All column headings are displayed           |
| TC02      | Print the first data row                      | First employee record is displayed          |
| TC03      | Print the last data row                       | Last employee record is displayed           |
| TC04      | Search for an employee by last name           | Matching employee record is displayed       |
| TC05      | Extract all email addresses                   | Email addresses are printed                 |
| TC06      | Find the employee with the highest Due amount | Employee with the highest Due is identified |
| TC07      | Verify that a particular website link exists  | Pass or fail result is reported             |
| TC08      | Count data rows excluding the header          | Total number of data rows is displayed      |

##  Concepts Practiced

* Launching a browser using Selenium WebDriver
* Locating web elements using `By.TAG_NAME`, `By.CSS_SELECTOR`, and `By.XPATH`
* Finding table headers and data rows
* Extracting cell values using `.text`
* Searching employee records using conditional statements
* Extracting email addresses
* Comparing numerical values to identify the highest amount
* Verifying hyperlinks using the `href` attribute
* Counting table rows without including the header
* Using explicit waits with `WebDriverWait`
* Handling browser closure using `try`, `finally`, and `driver.quit()`

## CODE
```

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()
driver.maximize_window()
wait = WebDriverWait(driver, 15)

URL = "https://assertqa.com/practice/webtables"


def get_table():
    return driver.find_element(By.TAG_NAME, "table")


def get_headers():
    table = get_table()
    return [
        th.text.strip()
        for th in table.find_elements(By.CSS_SELECTOR, "thead th")
    ]


def get_data_rows():
    table = get_table()
    return table.find_elements(By.CSS_SELECTOR, "tbody tr")


def get_column_index(headers, name):
    for i, header in enumerate(headers):
        if header.strip().lower() == name.lower():
            return i
    return -1


def get_all_records():
    """Collect records across all pagination pages."""
    records = []
    visited_pages = set()

    while True:
        headers = get_headers()
        rows = get_data_rows()

        # Prevent accidentally revisiting the same page.
        page_data = tuple(row.text for row in rows)
        if page_data in visited_pages:
            break
        visited_pages.add(page_data)

        for row in rows:
            cells = row.find_elements(By.CSS_SELECTOR, "td")
            if cells:
                records.append(
                    [cell.text.strip() for cell in cells]
                )

        # Find the next-page button.
        next_buttons = driver.find_elements(
            By.XPATH,
            "//button[contains(translate(normalize-space(.), "
            "'ABCDEFGHIJKLMNOPQRSTUVWXYZ', 'abcdefghijklmnopqrstuvwxyz'), "
            "'next')]"
        )

        if not next_buttons:
            break

        next_button = next_buttons[-1]

        if (
            not next_button.is_displayed()
            or not next_button.is_enabled()
            or next_button.get_attribute("disabled") is not None
        ):
            break

        old_page = page_data
        next_button.click()

        try:
            wait.until(
                lambda d: tuple(
                    row.text for row in get_data_rows()
                ) != old_page
            )
        except Exception:
            break

    return headers, records


def report(tc, description, passed):
    status = "PASS" if passed else "FAIL"
    print(f"\n{tc} - {description}: {status}")


try:
    driver.get(URL)

    wait.until(
        EC.presence_of_element_located((By.TAG_NAME, "table"))
    )

    headers, records = get_all_records()

    # TC01: Print all column headings
    print("\nTC01 - ALL COLUMN HEADINGS")
    for heading in headers:
        print(heading)

    report("TC01", "Print all column headings", len(headers) > 0)

    # TC02: Print the first data row
    print("\nTC02 - FIRST DATA ROW")
    if records:
        print(dict(zip(headers, records[0])))
    report("TC02", "Print first data row", len(records) > 0)

    # TC03: Print the last data row
    print("\nTC03 - LAST DATA ROW")
    if records:
        print(dict(zip(headers, records[-1])))
    report("TC03", "Print last data row", len(records) > 0)

    # TC04: Search for an employee by last name
    print("\nTC04 - SEARCH BY LAST NAME")
    last_name_index = get_column_index(headers, "Last Name")
    search_name = "Smith"

    matches = []
    if last_name_index != -1:
        matches = [
            dict(zip(headers, record))
            for record in records
            if record[last_name_index].lower() == search_name.lower()
        ]

    if matches:
        for employee in matches:
            print("Matching employee:", employee)
    else:
        print("No employee found with last name:", search_name)

    report("TC04", "Search employee by last name", bool(matches))

    # TC05: Extract all email addresses
    print("\nTC05 - ALL EMAIL ADDRESSES")
    email_index = get_column_index(headers, "Email")
    emails = []

    if email_index != -1:
        emails = [
            record[email_index]
            for record in records
            if email_index < len(record) and record[email_index]
        ]

    for email in emails:
        print(email)

    report("TC05", "Extract all email addresses", bool(emails))

    # TC06: Find employee with highest Due amount
    print("\nTC06 - HIGHEST DUE AMOUNT")
    due_index = get_column_index(headers, "Due")

    if due_index == -1:
        print("Cannot perform: the table has no Due column.")
        report("TC06", "Find employee with highest Due", False)
    else:
        due_records = []

        for record in records:
            if due_index < len(record):
                amount = record[due_index].replace("$", "").replace(",", "").strip()
                try:
                    due_records.append((float(amount), record))
                except ValueError:
                    continue

        if due_records:
            highest_amount, employee = max(
                due_records, key=lambda item: item[0]
            )
            print("Employee:", dict(zip(headers, employee)))
            print("Highest Due:", highest_amount)
            report("TC06", "Find employee with highest Due", True)
        else:
            print("No valid Due amounts found.")
            report("TC06", "Find employee with highest Due", False)

    # TC07: Verify that a particular website link exists
    print("\nTC07 - VERIFY WEBSITE LINK")
    website_index = get_column_index(headers, "Web Site")
    expected_website = "http://www.jsmith.com"

    if website_index == -1:
        print("Cannot perform: the table has no Web Site column.")
        report("TC07", "Verify website link exists", False)
    else:
        links_found = []

        for row in get_table().find_elements(
            By.CSS_SELECTOR, "tbody tr"
        ):
            cells = row.find_elements(By.CSS_SELECTOR, "td")

            if website_index < len(cells):
                links = cells[website_index].find_elements(
                    By.TAG_NAME, "a"
                )

                for link in links:
                    if expected_website in (
                        link.get_attribute("href") or ""
                    ):
                        links_found.append(link)

        print(
            "Website link found."
            if links_found
            else "Website link not found."
        )
        report(
            "TC07",
            "Verify website link exists",
            bool(links_found)
        )

    # TC08: Count data rows without counting the header
    print("\nTC08 - COUNT DATA ROWS")
    print("Total data rows:", len(records))
    report("TC08", "Count data rows excluding header", len(records) > 0)

finally:
    input("\nPress Enter to close the browser...")
    driver.quit()
```

## Execution and Results

<img width="1917" height="1065" alt="image" src="https://github.com/user-attachments/assets/4c6b63c7-aa35-4c27-8af4-913d8e3900ff" />
<img width="1917" height="1070" alt="image" src="https://github.com/user-attachments/assets/1651448b-a70a-4dcd-b45f-6e573036c5b8" />

## github link

https://github.com/Sarishatheiveegan/webtables-testing.git




