## **Scrape-Clean-Store-Social-Media-Data**
## **Project Overview**
This project focuses on scraping publicly available data from social media platforms, cleaning the data, and storing it in a well-structured relational database. The goal is to build a system that automates the collection, processing, and storage of social media data for analysis and reporting.

## **Features**
- **Data Scraping**: Extract publicly accessible user profiles, posts, comments, and hashtags.
- **Data Cleaning**: Remove inconsistencies and standardize data before storing.
- **Data Storage**: Use an MS SQL Server database to store the cleaned data following a structured schema based on the ER diagram.
- **Automation**: Enable periodic scraping and data updates.

## **Technologies Used**
- **Programming Language**: Python
- **Libraries**: 
  - `BeautifulSoup` and `requests` for web scraping.
  - `pyodbc` for connecting Python to MS SQL Server.
- **Database**: MS SQL Server
- **Optional Visualization**: Tableau, Power BI, or Python Dash for insights.

## **System Requirements**
- **Python**: Version 3.7 or above.
- **MS SQL Server**: Installed and configured with necessary permissions.
- **Libraries**:
  ```bash
  pip install beautifulsoup4 requests pyodbc
  ```
- **Optional Tools**:
  - Postman or social media APIs for structured data access.
  - A text editor or IDE for Python scripting (e.g., VS Code, PyCharm).

## **Database Design**
The database schema is based on the following ER diagram:

### **Entities and Relationships**
- **USER**: Stores user information.
- **POST**: Contains post details like captions and locations.
- **COMMENTS**: Includes comments on posts.
- **HASHTAGS**: Tracks hashtags used in posts.
- **FOLLOWERS**: Captures relationships between users.

Let's break down the entities and their relationships again for clarity:

### Entities:
1. **USER**  
   - Attributes: `user_id` (Primary Key), `username`, `email`

2. **POST**  
   - Attributes: `post_id` (Primary Key), `caption`, `location`, `user_id_fk` (Foreign Key referencing USER)

3. **COMMENTS**  
   - Attributes: `comment_id` (Primary Key), `content`, `user_id_fk` (Foreign Key referencing USER), `post_id_fk` (Foreign Key referencing POST)

4. **HASHTAGS**  
   - Attributes: `hashtag_id` (Primary Key), `hashtag_text`, `post_id_fk` (Foreign Key referencing POST)

5. **FOLLOWERS**  
   - Attributes: `follower_id` (Primary Key), `user_id_fk` (Foreign Key referencing USER as the follower), `following_user_id_fk` (Foreign Key referencing USER as the one being followed)

### Relationships:
- **USER** `creates` **POST**  
- **POST** `has` **COMMENTS**  
- **POST** `includes` **HASHTAGS**  
- **USER** `follows` **USER** (self-referential via FOLLOWERS table)

### **SQL Script for Schema Creation**
```sql
-- Create the database
CREATE DATABASE SocialMediaDB;

-- Switch to the database
USE SocialMediaDB;

-- Create USER table
CREATE TABLE [USER] (
    user_id INT PRIMARY KEY IDENTITY(1,1),
    username NVARCHAR(50) NOT NULL,
    email NVARCHAR(100) NOT NULL UNIQUE,
    bio NVARCHAR(255),
    profile_photo NVARCHAR(255),
    created_at DATETIME DEFAULT GETDATE()
);

-- Create POST table
CREATE TABLE POST (
    post_id INT PRIMARY KEY IDENTITY(1,1),
    user_id INT FOREIGN KEY REFERENCES [USER](user_id),
    caption NVARCHAR(500),
    location NVARCHAR(255),
    created_at DATETIME DEFAULT GETDATE()
);

-- Create COMMENTS table
CREATE TABLE COMMENTS (
    comment_id INT PRIMARY KEY IDENTITY(1,1),
    post_id INT FOREIGN KEY REFERENCES POST(post_id),
    user_id INT FOREIGN KEY REFERENCES [USER](user_id),
    comment_text NVARCHAR(500),
    created_at DATETIME DEFAULT GETDATE()
);

-- Create HASHTAGS table
CREATE TABLE HASHTAGS (
    hashtag_id INT PRIMARY KEY IDENTITY(1,1),
    hashtag_name NVARCHAR(50) NOT NULL UNIQUE
);

-- Create POST_TAGS table
CREATE TABLE POST_TAGS (
    post_id INT FOREIGN KEY REFERENCES POST(post_id),
    hashtag_id INT FOREIGN KEY REFERENCES HASHTAGS(hashtag_id),
    PRIMARY KEY (post_id, hashtag_id)
);

-- Create FOLLOWERS table
CREATE TABLE FOLLOWERS (
    follower_id INT FOREIGN KEY REFERENCES [USER](user_id),
    followed_id INT FOREIGN KEY REFERENCES [USER](user_id),
    followed_at DATETIME DEFAULT GETDATE(),
    PRIMARY KEY (follower_id, followed_id)
);
```

## **Implementation Steps**

### **Step 1: Web Scraping**
- Use Python to scrape user profiles, posts, and hashtags from an open social media platform.
- Example Python code for scraping:
```python
import requests
from bs4 import BeautifulSoup

# Target URL (replace with the open platform URL)
url = "https://example.com/public-posts"
response = requests.get(url)
soup = BeautifulSoup(response.text, 'html.parser')

# Example: Extract user profiles
users = soup.find_all('div', class_='user-profile')
for user in users:
    username = user.find('span', class_='username').text
    email = user.find('span', class_='email').text
    print(f"Username: {username}, Email: {email}")
```

### **Step 2: Data Cleaning**
- Use Python to preprocess data:
  - Remove duplicate records.
  - Standardize text fields (e.g., usernames, hashtags).
  - Handle missing or incomplete data.

### **Step 3: Data Storage**
- Use `pyodbc` to connect Python to MS SQL Server and insert the cleaned data into the database:
```python
import pyodbc

# Connect to SQL Server
conn = pyodbc.connect('Driver={SQL Server};'
                      'Server=YOUR_SERVER_NAME;'
                      'Database=SocialMediaDB;'
                      'Trusted_Connection=yes;')
cursor = conn.cursor()

# Insert data into USER table
cursor.execute("INSERT INTO [USER] (username, email, bio, profile_photo) VALUES (?, ?, ?, ?)",
               'example_user', 'user@example.com', 'Bio here', 'https://example.com/profile.jpg')

# Commit and close connection
conn.commit()
conn.close()
```

### **Step 4: Automation**
- Automate periodic scraping using:
  - **Windows Task Scheduler** (Windows).
  - **Cron Jobs** (Linux).
- Use libraries like `schedule` or `APScheduler` for Python-based scheduling.

## **Validation and Testing**
- Query the database to validate data integrity:
```sql
-- List all users
SELECT * FROM [USER];

-- Check posts and their associated hashtags
SELECT P.caption, H.hashtag_name
FROM POST P
JOIN POST_TAGS PT ON P.post_id = PT.post_id
JOIN HASHTAGS H ON PT.hashtag_id = H.hashtag_id;
```

## **Future Enhancements**
- Implement APIs for structured data access.
- Integrate visualization tools like Tableau or Power BI.
- Add logging and error handling for robust scraping.

## **Legal and Ethical Considerations**
- Ensure compliance with platform terms of service.
- Scrape only publicly available data.
- Follow privacy regulations like GDPR or CCPA.

## **File Structure**
Here’s the project file structure:
```
ScrapeCleanStoreSocialMedia/
│
├── README.md         # Documentation
├── schema.sql        # Database schema
├── scrape.py         # Web scraping script
├── clean_data.py     # Data cleaning script
├── store_data.py     # Data insertion script
├── requirements.txt  # Python dependencies
└── LICENSE           # License for the project
```

## **License**
This project is licensed under the [MIT License](https://opensource.org/licenses/MIT).
