# Santa's Chatbot: A Gemini-powered Christmas Conversation

This repository contains the code for a simple chat application that uses Google's Gemini language model to simulate a conversation with Santa Claus. The application is built using Streamlit and deployed on Google Cloud Run.  This is a sample application to demonstrate a basic Gemini integration.

## Application Description

This Streamlit application provides a fun and interactive chat experience where users can converse with a virtual Santa Claus. The application leverages the Gemini language model to generate Santa's responses, ensuring a jolly and appropriate conversation. The model is carefully instructed to maintain a consistent Santa persona, using child-friendly language, and avoiding inappropriate topics. Robust safety measures are in place to prevent harmful or unsafe content generation.

## Architecture

The application follows a simple client-server architecture:

1. **Client (Frontend):** A Streamlit web application handles user interaction, displaying the chat interface and sending user messages to the backend.
2. **Server (Backend):** A Python script hosted on Google Cloud Run. It receives user messages, interacts with the Gemini API, and sends the generated responses back to the client.
3. **Gemini API:** Google's Gemini Pro language model, accessed via the Vertex AI API, powers the Santa Claus persona.

**Diagram:**

```
+-----------------+     +-----------------+     +-----------------+
| Streamlit Client|---->| Cloud Run Server|---->| Gemini API       |
| (User Interface)|     | (Python Script) |     | (Google Vertex AI)|
+-----------------+     +-----------------+     +-----------------+
```

## Getting Started

Before deploying, ensure you have the following:

1. **Google Cloud Project:**  Create a Google Cloud project and enable the necessary APIs: Vertex AI and Cloud Run.
2. **Service Account:** Create a service account with the `aiplatform.user` role. Download the service account key JSON file (**DO NOT** commit this to the repository).
3. **Environment Variables:** Set the following environment variables:
    * `GOOGLE_APPLICATION_CREDENTIALS`: Path to your service account key file.
    * `PROJECT_ID`: Your Google Cloud project ID.
    * `REGION`: Your preferred Google Cloud region (e.g., `us-central1`).
4. **Python Dependencies:** Install the required Python packages:
    ```bash
    pip install streamlit vertexai
    ```

## Deployment

This section describes deploying the application to Google Cloud Run.  This requires the Google Cloud SDK (`gcloud`).

**1. Build the Docker Image:**

First, you need a `Dockerfile` in the root of your project. A basic example is provided in the repo. You'll likely need to adjust it based on your specific needs.
Then, build the Docker image and push it to Google Container Registry (GCR):

```bash
gcloud builds submit --tag us-central1-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$SERVICE_NAME .
```

* Replace placeholders with your project ID (`$PROJECT_ID`), repository name (`$REPO_NAME`, e.g., `santa-chatbot`), and service name (`$SERVICE_NAME`, e.g., `santa-chat`).

**2. Deploy to Cloud Run:**

Deploy the image to Cloud Run:

```bash
gcloud run deploy $SERVICE_NAME \
    --image us-central1-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$SERVICE_NAME \
    --platform managed \
    --region=$REGION
```

Remember to remove `--allow-unauthenticated` from production deployments and set up appropriate authentication.

**3. Granting Permissions (Vertex AI Access):**

Grant your Cloud Run service account access to the Gemini API:

```bash
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member=serviceAccount:$SERVICE_ACCOUNT_EMAIL \
    --role=roles/aiplatform.user
```

Replace `$SERVICE_ACCOUNT_EMAIL` with your service account's email address.


## Safety Filters and Measures

The application employs multiple safety layers to ensure appropriate conversations:

1. **Gemini's Built-in Safety Filters:** Configured using `safety_settings` in the code.

```python
safety_settings = [
    SafetySetting(
        category=SafetySetting.HarmCategory.HARM_CATEGORY_HATE_SPEECH,
        threshold=SafetySetting.HarmBlockThreshold.BLOCK_LOW_AND_ABOVE
    ),
    SafetySetting(
        category=SafetySetting.HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT,
        threshold=SafetySetting.HarmBlockThreshold.BLOCK_LOW_AND_ABOVE
    ),
    SafetySetting(
        category=SafetySetting.HarmCategory.HARM_CATEGORY_SEXUALLY_EXPLICIT,
        threshold=SafetySetting.HarmBlockThreshold.BLOCK_LOW_AND_ABOVE
    ),
    SafetySetting(
        category=SafetySetting.HarmCategory.HARM_CATEGORY_HARASSMENT,
        threshold=SafetySetting.HarmBlockThreshold.BLOCK_LOW_AND_ABOVE
    ),
]
```

This code snippet configures the safety settings to block content that falls under the categories of hate speech, dangerous content, sexually explicit content, and harassment, even at low levels of detection.  The `BLOCK_LOW_AND_ABOVE` threshold ensures the strictest possible filtering.

**Modifying Safety Settings:**

To adjust the level of safety filtering, you can modify the `threshold` values within each `SafetySetting`. The available options are:

* `BLOCK_NONE`: No filtering.
* `BLOCK_HIGH`: Blocks only content with high confidence of being harmful.
* `BLOCK_MEDIUM_AND_ABOVE`: Blocks medium and high-confidence harmful content.
* `BLOCK_LOW_AND_ABOVE`: Blocks low, medium, and high-confidence harmful content (most restrictive).

You can also add or remove categories as needed, based on your specific requirements and risk tolerance.  Refer to the official Vertex AI documentation for a complete list of available `HarmCategory` options.

2. **Custom Response Handling:** A `try-except` block handles `ResponseValidationError` exceptions.  This provides a fallback response if the safety filters are triggered.

```python
try:
    # ... Gemini API call ...
except ResponseValidationError:
    santa_response = "Ho ho ho! It seems like my elves are a bit worried about that message. Let's keep our conversations focused on the Christmas spirit and all things merry and bright!"
    # ... handle the error ...
```
You can customize the `santa_response` message to better suit your needs.

3. **System Instructions:**  Detailed instructions guide the Gemini model to maintain a consistent Santa Claus persona and avoid inappropriate topics.

```python
system_instructions = """You ARE Santa Claus..."""  # ... (Your existing instructions)
```

These instructions explicitly guide the model to avoid inappropriate topics and maintain the Santa persona.  This preventative approach minimizes the need for extensive post-generation filtering.

## License

Apache 2.0.
