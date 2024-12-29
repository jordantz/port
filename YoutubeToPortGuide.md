# Ingest Data From Youtube To Port
This guide takes under 10 minutes to complete, and aims to demonstrate the ease with which data from external platforms can be retrieved, ingested, and displayed in Port.

> Prerequisite: [Port's GitHub app](https://github.com/apps/getport-io) is installed for the relevant repository.


## The goal of this guide
In this guide, we will create a platform specific blueprint, webhook, and workflow action, which will streamline data from Youtube into Port.

Port's flexibility ensures that the retrieved data can be easily modeled, viewed, and cataloged to fit any of the user's distinct needs.

### Create Blueprint

1. Head to the [Builder ](https://app.getport.io/settings/data-model) page of your portal.
2. Select `+ Blueprint` and create a blueprint with your desired name. For simplicity, we chose **Youtube Data**.
3. Add any property that you want to be displayed and determine how to display it. Our example blueprint includes: ![image](https://i.imgur.com/Txl5nfp.png) 
###### The blueprint is completely customizable and allows for future changes to match the dynamic nature of the information you require. More on how to utilize blueprints [here](https://docs.getport.io/build-your-software-catalog/customize-integrations/configure-data-model/setup-blueprint/).

<details>
  <summary>Blueprint JSON (click to expand)</summary>
  <pre><code>
    {
    "identifier": "youtube_data_blueprint",
    "description": "This blueprint displays the retrieved data from the chosen Youtube Source in Port",
    "title": "Youtube Data",
    "icon": "Actions",
    "schema": {
      "properties": {
        "youtube_account_owner": {
          "type": "string",
          "description": "The account which uploaded the Youtube video",
          "title": "Account Owner",
          "icon": "User"
        },
        "youtube_views_count": {
          "type": "number",
          "description": "The number of views of the Youtube video",
          "title": "Views Count",
          "icon": "Infinity"
        },
        "youtube_comments_count": {
          "type": "number",
          "description": "The number of comments on the Youtube video",
          "title": "Comments Count",
          "icon": "DefaultBlueprint"
        },
        "youtube_video_url": {
          "type": "string",
          "description": "The URL of the Youtube Video",
          "title": "Video URL",
          "icon": "Url",
          "format": "url"
        },
        "youtube_upload_date": {
          "type": "string",
          "description": "The date on which the video was uploaded",
          "title": "Upload Date",
          "icon": "Calendar",
          "format": "date-time"
        }
      },
      "required": []
    },
    "mirrorProperties": {},
    "calculationProperties": {},
    "aggregationProperties": {},
    "relations": {}
  }
  </code></pre>
</details>

### Create Webhook
1. Head to the [Data Sources](https://app.getport.io/settings/data-sources) page of your portal.
2. Select `+ Data source` in the top-right corner, navigate to the webhook tab, and click on `Custom integration`.
3. Pick a title that describes the webook's responsibility. For example: **Youtube Webhook**.
4. Scroll to the `Map the data from the external system into Port` section of the Mapping tab and edit it to match your blueprint.
<details>
  <summary>Webhook JSON (click to expand)</summary>
  <pre><code>
[
  {
    "blueprint": "youtube_data_blueprint",
    "operation": "create",
    "filter": ".body.entity.title != null",
    "entity": {
      "identifier": ".body.entity.identifier",
      "title": ".body.entity.title",
      "properties": {
        "youtube_upload_date": ".body.entity.properties.youtube_upload_date",
        "youtube_account_owner": ".body.entity.properties.youtube_account_owner",
        "youtube_views_count": ".body.entity.properties.youtube_views_count",
        "youtube_comments_count": ".body.entity.properties.youtube_comments_count",
        "youtube_video_url": ".body.entity.properties.youtube_video_url"
      }
    }
  }
]
  </code></pre>
</details>

5. Save the URL of the webhook, which can be found in the `Configure Webhook event` section of the Mapping tab, as it will be needed for the Workflow step.
###### There are multiple data sources already integrated into Port. Learn more [here](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/#available-plug--play-integrations). Don't see the one you need? You can always [add your own](https://docs.getport.io/build-your-software-catalog/custom-integration/)! 

### Create Workflow Action
1. Head to the [Self Service](https://app.getport.io/self-serve) page of your portal.
2. Select `+ Action` in the top-right corner, and pick a title that best describes the action you will create. **Get Data From Youtube** has a nice ring to it, doesn't it?
3. In the `Basic Details` tab, choose the operation that this action will perform, and the blueprint associated with it. For this guide's purpose, we will choose the **Create** operation, and the blueprint **Youtube Data** that we just created.
4.  In the `User Form` tab, select `+ Input`, and add a required URL input.
5. Fill the `Backend` tab of the action. Port supports multiple invocation types to match your preferred Git provider. In this guide, we will demonstrate GitHub: ![image](https://i.imgur.com/iElwYZR.png)
edit the organization and repository names to match your own.
6. Change the permissions as necessary. 
###### You can do just about anything with a Port Action! Learn [how](https://docs.getport.io/actions-and-automations/create-self-service-experiences/).
<details>
  <summary>Workflow Action JSON (click to expand)</summary>
  <pre><code>
{
  "identifier": "get_data_from_youtube",
  "title": "Get data from Youtube",
  "description": "Fetches data from the given Youtube source and ingests it to Port",
  "trigger": {
    "type": "self-service",
    "operation": "CREATE",
    "userInputs": {
      "properties": {
        "youtube_url": {
          "title": "Youtube URL",
          "description": "Youtube Source Link for a video or playlist",
          "icon": "Link",
          "type": "string",
          "format": "url"
        }
      },
      "required": [
        "youtube_url"
      ],
      "order": [
        "youtube_url"
      ]
    },
    "blueprintIdentifier": "youtube_data_blueprint"
  },
  "invocationMethod": {
    "type": "GITHUB",
    "org": "jordantz",
    "repo": "port",
    "workflow": "fetch_youtube_data.yml",
    "workflowInputs": {
      "": "",
      "youtube_url": ""
    },
    "reportWorkflowStatus": true
  },
  "requiredApproval": false
}
  </code></pre>
</details>

### Create Workflow
1. Create a workflow file in your git repository under the folder `.github/workflows`. Ensure that:
- The `filename` matches the one you provided in the Port action. In our case, **fetch_youtube_data.yml**.
- The `input identifier` recieved from port matches the one in the workflow file.
- The `payload properties` correlate with the blueprint you constructed.
- The `port_url` equals the webhook URL that was copied in the **Create Webhook** step.
<details>
  <summary>GitHub Workflow (click to expand)</summary>
  <pre><code>
name: Fetch YouTube Data and Ingest into Port

on:
  workflow_dispatch:
    inputs:
     youtube_url:
        description: "the desired YouTube Source's URL"
        required: true
        type: string

jobs:
  fetch-and-ingest:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.9'

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install requests google-api-python-client

    - name: Fetch Youtube Data and Ingest into Port
      env:
        YOUTUBE_API_KEY: $ # Youtube API key, acquired from Google Cloud Console
        PORT_API_KEY: $       # Port API key, acquired from Port
      run: |
        python <<EOF
        import os
        import requests
        from urllib.parse import urlparse, parse_qs
        from googleapiclient.discovery import build

        # Inputs and API keys
        youtube_url = "$"
        youtube_api_key = os.environ['YOUTUBE_API_KEY']
        port_api_key = os.environ['PORT_API_KEY']
        port_url = "https://ingest.getport.io/OxySO1wS0HYHaEJ2"  # Port Webhook specific URL

        # Parse the input URL
        query_params = parse_qs(urlparse(youtube_url).query)
        playlist_id = query_params.get("list", [None])[0]
        single_video_id = query_params.get("v", [None])[0]
        if not (playlist_id or single_video_id):
            raise ValueError("Invalid URL. Provide a valid YouTube playlist or video link.")

        # Initialize YouTube API client
        youtube = build('youtube', 'v3', developerKey=youtube_api_key)

        # Headers for Port API request
        headers = {
            "Content-Type": "application/json",
            "Authorization": port_api_key
        }

        # Function to ingest video data into Port
        def ingest_video(video_id):
            # Fetch video details
            video_response = youtube.videos().list(
                part="snippet,statistics",
                id=video_id
            ).execute()

            if not video_response.get("items"):
                raise ValueError(f"Video with ID {video_id} not found.")

            video_data = video_response["items"][0]
            snippet = video_data["snippet"]
            stats = video_data["statistics"]

            # Prepare the payload
            payload = {
                "identifier": video_id,
                "title": snippet["title"],
                "icon": "Default",
                "entity": {
                    "identifier": video_id,
                    "title": snippet["title"],
                    "properties": {
                        "youtube_account_owner": snippet["channelTitle"],
                        "youtube_upload_date": snippet["publishedAt"],
                        "youtube_views_count": int(stats.get("viewCount", 0)),
                        "youtube_comments_count": int(stats.get("commentCount", 0)),
                        "youtube_video_url": f"https://www.youtube.com/watch?v={video_id}"
                    }
                }
            }

            # Log the Payload
            print(payload)

            # Ingest into Port
            response = requests.post(port_url, json=payload, headers=headers)
            
            # Log response status and content
            print(f"Status Code: {response.status_code}")
            print(f"Response Headers: {response.headers}")
            print(f"Response Text: {response.text}")

            if response.status_code not in [200, 202]:
                raise Exception(f"Failed to ingest video {snippet['title']}")
            
        # If it's a playlist, fetch all videos
        if playlist_id:
            next_page_token = None
            while True:
                playlist_response = youtube.playlistItems().list(
                    part="snippet",
                    playlistId=playlist_id,
                    maxResults=50,
                    pageToken=next_page_token
                ).execute()

                for item in playlist_response.get("items", []):
                    video_id = item["snippet"]["resourceId"]["videoId"]
                    ingest_video(video_id)

                # Check if more pages exist
                next_page_token = playlist_response.get("nextPageToken")
                if not next_page_token:
                    break
        elif single_video_id:  # If it's a single video
            ingest_video(single_video_id)
        EOF
  </code></pre>
</details>

### Add API Keys
1. Head to your Port application, click on the `...` in the top-right corner, then select Credentials. Click to `Generate API token` and save the generated key. Note that this token expires after an hour.
2. Head to Google Cloud Console, log in, and click the `Select a Project` button. From there, choose `New Project`, and give it a fitting name such as `Youtube to Port`.
3. Go to `APIs & Services > Library` in the left-hand menu,
search for `YouTube Data API v3`, and enable it.
4.  Go to `APIs & Services > Credentials` in the left-hand menu, and
choose `CREATE CREDENTIALS > API key`.  Save the generated key.
5. Head to your GitHub repository, click on `Settings > Secrets and variables > Actions`, and create `Repository secrets` with the generated keys. Their names should be **PORT_API_KEY** and **YOUTUBE_API_KEY** respectively.

### Execute
You're all done! What's left is to head back to Port, and click on the `Create` button in the `Self-Service Hub`. You'll be prompted to supply a youtube URL before executing the action and voilà! The data from the provided link will be displayed under the corresponding blueprint section in the `Catalog` tab.

Now all the data you requested pertaining to the provided link is neatly shown in Port. 

But *wait*, there's more that can be done! 

You can add visualizations to make select information stand out, and ensure that your users see what is most relevant to them - and to you. Port offers a variety of possible widgets to match your every need. These can be found under the [dashboard](https://docs.getport.io/customize-pages-dashboards-and-plugins/dashboards/) category.

Curious to see some examples? We've got you covered!
![image](https://i.imgur.com/XWHhjdE.png)
![image](https://i.imgur.com/7xQ7t6m.png)
This, and much more, awaits you in Port.

