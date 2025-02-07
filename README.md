# Auto-application-Script
Created a Scripted to autoapply to linkedin and Indeed jobs

import time
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import (
    StaleElementReferenceException,
    TimeoutException,
    NoSuchElementException
)

# ----------------- LINKEDIN AUTO APPLY ----------------- #

def linkedin_auto_apply():
    print("\n=== LinkedIn Remote Auto-Apply ===")
    linkedin_email = input("Enter LinkedIn email: ")
    linkedin_password = input("Enter LinkedIn password: ")
    search_query = input("Enter job search query for LinkedIn (e.g., 'Software Engineer'): ")

    # Set up Chrome options (uncomment headless mode if desired)
    chrome_options = Options()
    # chrome_options.add_argument("--headless")
    driver = webdriver.Chrome(options=chrome_options)
    wait = WebDriverWait(driver, 15)

    try:
        # Log in to LinkedIn
        driver.get("https://www.linkedin.com/login")
        wait.until(EC.presence_of_element_located((By.ID, "username"))).send_keys(linkedin_email)
        driver.find_element(By.ID, "password").send_keys(linkedin_password)
        driver.find_element(By.XPATH, "//button[@type='submit']").click()
        time.sleep(3)
        print("Logged in to LinkedIn successfully!")

        # Navigate to remote job search page (f_WT=2 filters for remote jobs)
        search_url = f"https://www.linkedin.com/jobs/search/?keywords={search_query.replace(' ', '%20')}&f_WT=2"
        driver.get(search_url)
        time.sleep(3)
        print("Searching for remote jobs on LinkedIn...")

        # Fetch job cards
        job_cards = driver.find_elements(By.CLASS_NAME, "job-card-container--clickable")
        num_jobs = len(job_cards)
        print(f"Found {num_jobs} remote job listings on LinkedIn.")

        for i in range(num_jobs):
            try:
                # Re-fetch job cards to reduce stale references
                job_cards = driver.find_elements(By.CLASS_NAME, "job-card-container--clickable")
                if i >= len(job_cards):
                    break
                job = job_cards[i]
                driver.execute_script("arguments[0].scrollIntoView();", job)
                job.click()
                time.sleep(2)

                # Look for the "Easy Apply" button using robust selectors
                try:
                    apply_button = wait.until(
                        EC.element_to_be_clickable(
                            (By.XPATH, "//button[contains(@aria-label, 'Easy Apply') and contains(@class, 'jobs-apply-button')]")
                        )
                    )
                except TimeoutException:
                    try:
                        apply_button = wait.until(
                            EC.element_to_be_clickable(
                                (By.XPATH, "//button[span[contains(text(),'Easy Apply')]]")
                            )
                        )
                    except TimeoutException:
                        print(f"Job #{i+1}: No 'Easy Apply' button found; skipping this job.")
                        continue

                apply_button.click()
                print(f"Job #{i+1}: Clicked 'Easy Apply' button.")
                time.sleep(3)  # Allow the application modal to load

                # --- Application Modal Flow ---
                # Loop: Click the "Next" button repeatedly if available
                while True:
                    try:
                        next_button = wait.until(
                            EC.element_to_be_clickable((By.XPATH, "//button[@data-easy-apply-next-button]"))
                        )
                        next_button.click()
                        print(f"Job #{i+1}: Clicked 'Next' button.")
                        time.sleep(2)
                    except TimeoutException:
                        # If no Next button is found, try clicking the "Review" button
                        try:
                            review_button = wait.until(
                                EC.element_to_be_clickable((By.XPATH, "//button[contains(@aria-label, 'Review')]"))
                            )
                            review_button.click()
                            print(f"Job #{i+1}: Clicked 'Review' button.")
                            time.sleep(2)
                        except TimeoutException:
                            print(f"Job #{i+1}: No 'Next' or 'Review' button found.")
                        break

                # Scroll down if necessary and click the "Submit application" button
                try:
                    submit_button = wait.until(
                        EC.element_to_be_clickable((By.XPATH, "//button[contains(@aria-label, 'Submit application')]"))
                    )
                    driver.execute_script("arguments[0].scrollIntoView(true);", submit_button)
                    submit_button.click()
                    print(f"Job #{i+1}: Clicked 'Submit application' button.")
                    time.sleep(2)
                except TimeoutException:
                    print(f"Job #{i+1}: 'Submit application' button not found; manual review may be required.")

                # Optionally, click the "Done" button to complete the process
                try:
                    done_button = wait.until(
                        EC.element_to_be_clickable((By.XPATH, "//button[contains(@aria-label, 'Done')]"))
                    )
                    done_button.click()
                    print(f"Job #{i+1}: Clicked 'Done' button.")
                    time.sleep(2)
                except TimeoutException:
                    print(f"Job #{i+1}: 'Done' button not found. Moving on.")

            except StaleElementReferenceException:
                print(f"Job #{i+1}: Stale element reference encountered. Skipping this job.")
            except Exception as e:
                print(f"Job #{i+1}: Error processing job: {e}")

    except Exception as e:
        print("An error occurred during LinkedIn automation:", e)
    finally:
        driver.quit()
        print("LinkedIn browser closed.\n")

# ----------------- INDEED AUTO APPLY ----------------- #

def indeed_auto_apply():
    print("\n=== Indeed Remote Auto-Apply ===")
    indeed_email = input("Enter Indeed email: ")
    indeed_password = input("Enter Indeed password: ")
    search_query = input("Enter job search query for Indeed (e.g., 'Software Engineer'): ")
    location = "remote"  # Force remote jobs

    chrome_options = Options()
    # chrome_options.add_argument("--headless")
    driver = webdriver.Chrome(options=chrome_options)
    wait = WebDriverWait(driver, 15)

    try:
        driver.get("https://secure.indeed.com/account/login")
        wait.until(EC.presence_of_element_located((By.ID, "login-email-input"))).send_keys(indeed_email)
        driver.find_element(By.ID, "login-password-input").send_keys(indeed_password)
        driver.find_element(By.XPATH, "//button[@type='submit']").click()
        time.sleep(5)
        print("Logged in to Indeed successfully!")

        search_url = f"https://www.indeed.com/jobs?q={search_query.replace(' ', '+')}&l={location}"
        driver.get(search_url)
        wait.until(EC.presence_of_all_elements_located((By.CSS_SELECTOR, "div.job_seen_beacon")))
        time.sleep(2)

        job_cards = driver.find_elements(By.CSS_SELECTOR, "div.job_seen_beacon")
        num_jobs = len(job_cards)
        print(f"Found {num_jobs} remote jobs for '{search_query}' on Indeed.")

        for i in range(num_jobs):
            try:
                job_cards = driver.find_elements(By.CSS_SELECTOR, "div.job_seen_beacon")
                if i >= len(job_cards):
                    break
                job = job_cards[i]
                driver.execute_script("arguments[0].scrollIntoView(true);", job)
                job.click()
                time.sleep(2)

                try:
                    apply_button = wait.until(
                        EC.element_to_be_clickable((By.XPATH, "//button[contains(text(), 'Apply Now')]"))
                    )
                    apply_button.click()
                    print(f"Job #{i+1}: Clicked 'Apply Now'.")
                    time.sleep(3)
                except (NoSuchElementException, TimeoutException):
                    print(f"Job #{i+1}: 'Apply Now' button not found; skipping this job.")

            except StaleElementReferenceException:
                print(f"Job #{i+1}: Stale element reference encountered. Skipping this job.")
            except Exception as e:
                print(f"Job #{i+1}: Error processing job: {e}")

    except Exception as e:
        print("An error occurred during Indeed automation:", e)
    finally:
        driver.quit()
        print("Indeed browser closed.\n")

# ----------------- MAIN MENU ----------------- #

if __name__ == "__main__":
    print("Which platform do you want to run? (linkedin/indeed)")
    choice = input("Enter your choice: ").strip().lower()

    if choice == "linkedin":
        linkedin_auto_apply()
    elif choice == "indeed":
        indeed_auto_apply()
    else:
        print("Invalid option. Please run the script again and choose either 'linkedin' or 'indeed'.")
