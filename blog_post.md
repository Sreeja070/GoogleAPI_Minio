
# Leveraging Google Places API for Restaurant Data and Storing It in MinIO

In today’s digital world, location data is the backbone of many cutting-edge apps, from travel planners to restaurant finders and even e-commerce platforms. **Google Places API** is a robust tool that opens the door to an entire world of place-related data, empowering developers to enrich their applications with dynamic location intelligence. 

The Google Places API is a service provided by Google that allows applications to access detailed information about places, such as businesses, landmarks, and other points of interest.

In this blog we will see how to do step by step process to get the data using Google Places API and then store it in the **Minio** (Open source data storage/data-lake)


## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Step 1: Get the Google Places API Key](#step-1-get-the-google-places-api-key)
3. [Step 2: Get MinIO Credentials](#step-2-get-minio-credentials)
4. [Step 3: Create and Execute the Python Script](#step-3-create-and-execute-the-python-script)
5. [Advantages of Using Google Places API and MinIO](#advantages-of-using-google-places-api-and-minio)
6. [Alternatives to Google Places API](#alternatives-to-google-places-api)
7. [References](#references)

## Prerequisites
Before we start, ensure you have the following in place:
- Google Cloud Platform (GCP) account
- MinIO set up (either locally or in the cloud)
- Python installed on your system

We’ll use three configuration files in this project:
1. **Google_Places_API_config.py** – for Google Places API configuration
2. **MinIO_config.py** – for MinIO client configuration
3. **Main.py** – the main Python script to run the process

## Step 1: Get the Google Places API Key
First, sign in to your GCP console and create a new project. Once your project is ready:

1. Enable the Google Places API: Navigate to **APIs & Services > Enable APIs and Services**, and search for "Places API."
2. Generate an API Key: Go to **APIs & Services > Credentials** and create a new API key.




We will use this API key in the **Google_Places_API_config.py** file.

## Step 2: Get MinIO Credentials
MinIO credentials can be obtained by logging into your MinIO console. Create a bucket and an object path (if it doesn’t exist, the script will create it automatically). Below is a snippet of how you would structure your MinIO config file:

```python
# MinIO_config.py
import minio

def get_minio_client():
    client = minio.Minio(
        "play.min.io",
        access_key="your-access-key",
        secret_key="your-secret-key",
        secure=True
    )
    return client
```

## Step 3: Create and Execute the Python Script
In the **Main.py** file, we’ll cover the following steps:

### 1. Importing Libraries & Initializing Clients
```python
import json
import time
from io import BytesIO
from minio.error import S3Error
from config.minio_config import get_minio_client
from config.google_api_config import get_gmaps_client

# Initialize clients
gmaps = get_gmaps_client()
minio_client = get_minio_client()
```

### 2. Fetching Nearby Places Using Google Places API
- API request made via `gmaps.places_nearby()` to retrieve restaurant data based on keyword (e.g., "Indian") and location.
- Detailed restaurant information is returned, including name, address, and operational status.
- Full implementation available in the GitHub repository.
```python
places_result = gmaps.places_nearby(location=(latitude, longitude), radius=1500, keyword="Indian")
```

### 3. Filtering Results & Handling Pagination
```python
while 'next_page_token' in places_result:
    time.sleep(2)  # Ensures we wait before fetching the next page
    places_result = gmaps.places_nearby(page_token=places_result['next_page_token'])
```
- Script extracts relevant details (name, rating, address) from each page of results.
- Pagination handled via `next_page_token` to ensure all data is collected.
- Short delay (`time.sleep(2)`) between requests for subsequent pages.

### 4. Uploading Data to MinIO
- Filtered results serialized to JSON and uploaded to MinIO using `BytesIO`.
- Script auto-creates the bucket if it doesn’t exist before upload.
- Data can be stored in other formats like PARQUET with minor code adjustments.
```python
json_data = json.dumps(filtered_results).encode('utf-8')
data_stream = BytesIO(json_data)

try:
    minio_client.put_object('my-bucket', 'restaurants.json', data_stream, len(json_data))
    print("Data uploaded successfully.")
except S3Error as err:
    print(f"Error: {err}")
```

### File Structure
```plaintext
GoogleAPI/
│
├── Main.py                      # Main Python script
│
└── Config/
    ├── MinIO_config.py          # MinIO configuration
    └── Google_API_Config.py     # Google API configuration
```

To execute, navigate to the project folder and run the **Main.py** file:

```bash
python3 Main.py
```

## Advantages of Using Google Places API and MinIO
1. **Access to rich, real-time location data** – Google Places API provides detailed and up-to-date information about businesses and other points of interest.
2. **Flexibility in data filtering** – Filter data to keep only the attributes that matter most for your use case.
3. **Scalable and cost-efficient storage** – MinIO offers scalable object storage, ideal for large datasets.

## Alternatives to Google Places API
While Google Places API is a powerful tool, there are more affordable alternatives available:
- **TomTom Search API**: $0.5 per 1,000 requests (40x cheaper than Google)
- **Mapbox**: Free for the first 100,000 requests, then $0.75 per 1,000 requests
- **OpenStreetMap**: Free, though with some limitations

However, these alternatives might not offer the same level of detail or completeness as Google Places API. Another option is the **SERP API**, which offers easier integration and competitive pricing.

## References
- [Google Places API Documentation](https://developers.google.com/maps/documentation/places/web-service/overview)
- [MinIO Documentation](https://min.io/docs/minio/windows/index.html)
- [GitHub Repository for Code](https://github.com/Sreeja070/GoogleAPI_Minio)
- [Alternatives to Google Places API](https://medium.com/@paulotaylor/google-places-api-alternatives-a-guide-to-save-you-big-money-with-genai-774605e04769)
