# Ingest Data From Youtube To Port
This guide takes under 10 minutes to complete, and aims to demonstrate the ease with which data from external platforms can be retrieved, ingested, and displayed in Port.

> Prerequisite: [Port's GitHub app](https://github.com/apps/getport-io) is installed for the relevant repository.


## The goal of this guide
In this guide, we will create a platform specific blueprint, workflow action, and webhook, which will streamline Youtube data into Port.

Port's flexibility ensures that the retrieved data can be easily modeled, viewed, and cataloged to fit any of the user's distinct needs.

### Create Blueprint

1. Head to the [Builder ](https://app.getport.io/settings/data-model) page of your portal.
2. Choose ![image](https://i.imgur.com/sbKEsJg.png) and create a blueprint with your desired name. For simplicity, we chose **Youtube Data**.
3. Add any property that you want to be displayed and determine how to display it. For example: ![image](https://i.imgur.com/Txl5nfp.png) 
###### The blueprint is versatile and allows for constant editing in order to match the relevant information you require.

The following is a JSON snippet for the created blueprint:
<details>
  <summary>Click to expand</summary>
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
</details>
