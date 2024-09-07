# Web_scraping_WikiHow_for_training
WebScraping WikiHow using bs4 ,python to get Sub-heading and paragraph to train a models
## **Project Documentation**



This project is designed to scrape articles from Wikihow, extract the title, subheadings, and paragraphs, and store the extracted data in a CSV file.

### **Code Structure**

The code is organized into the following sections:

1. **Import Libraries:**
   - `requests`: For making HTTP requests to Wikihow.
   - `BeautifulSoup`: For parsing the HTML content of the scraped pages.
   - `csv`: For writing the extracted data to a CSV file.

2. **Loop and Scrape:**
   - Iterates over a specified number of articles.
   - Sends an HTTP request to a random Wikihow page.
   - Parses the HTML content using BeautifulSoup.

3. **Extract Data:**
   - Extracts the article title from the `<title>` tag.
   - Finds all `step` divs and extracts subheadings from `<b>` tags.
   - Extracts paragraphs from the remaining text within each `step` div.

4. **Write to CSV:**
   - Writes the extracted data (title, subheadings, and paragraphs) to a CSV file, creating a new row for each subheading and paragraph.

### **Code Snippet**

```python
import requests
from bs4 import BeautifulSoup
import csv

# Loop to scrape multiple articles
for count in range(4000):
    # URL of the Wikihow page to scrape
    url = "[https://www.wikihow.com/Special:Randomizer](https://www.wikihow.com/Special:Randomizer)"

    # Send an HTTP request and receive HTML content
    response = requests.get(url)
    html_content = response.content

    # Parse the HTML content using BeautifulSoup
    soup = BeautifulSoup(html_content, 'html.parser')

    # Extract the article title
    article_title = soup.find('title').text.strip()
    print(f"Article Title (Iteration {count+1}): {article_title}")  # Informative output

    # Extract subheadings and paragraphs using appropriate HTML tags
    subheadings = []
    paragraphs = []

    # Find all 'step' divs and extract subheadings and paragraphs
    steps = soup.find_all('div', class_="step")
    for step in steps:
        # Extract subheadings
        subheading_element = step.find('b')
        if subheading_element:
            subheading_text = subheading_element.get_text().strip().replace('\n', ' ')
            subheading_text = subheading_text.encode('ascii', errors='ignore').decode()
            subheading_text = re.sub(r'\s+', ' ', subheading_text)
            subheadings.append(subheading_text)
            subheading_element.extract()  # Remove subheading element to avoid duplication

        # Remove span tags
        for span_tag in step.find_all("span"):
            span_tag.extract()

        # Extract paragraph text
        paragraph_text = step.get_text().strip().replace("\n", " ").replace("'", "")
        paragraph_text = paragraph_text.encode('ascii', errors='ignore').decode()
        paragraph_text = re.sub(r'\s+', ' ', paragraph_text)
        paragraphs.append(paragraph_text)

    # Write data to CSV only if subheadings were found
    if len(subheadings):
        # Create the CSV file with headers in 'append' mode if it doesn't exist
        with open('wikiHow.csv', mode='a', newline='', encoding='utf-8') as csv_file:
            writer = csv.writer(csv_file)
            # Write headers only on the first iteration (avoid duplicates)
            if count == 0:
                writer.writerow(['title', 'heading', 'paragraph'])
            for i in range(len(subheadings)):
                writer.writerow([article_title, subheadings[i], paragraphs[i]])

print("Scraping completed. Check 'wikiHow.csv' for extracted data.")
