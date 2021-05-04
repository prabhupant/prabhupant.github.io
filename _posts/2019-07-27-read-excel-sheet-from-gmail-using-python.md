---
layout: post
title: "Read Excel Sheets from Gmail with Python"
---


Recently, I had to write a Python program to automatically read emails with a specific label from my Gmail account and process the excel sheet attached in that mail daily at a specific time.

I knew that processing the excel sheet will be an easy task with Pandas but the real problem was getting the excel file, especially the second sheet of it. This took me quite some time and finally I was able to find a workaround.
To read emails from Gmail, there are two ways — using [imaplib](https://docs.python.org/3/library/imaplib.html) or using Gmail API.

To use imaplib, just allow it the access (Go to Settings > Forwarding and POP/IMAP > Enable IMAP) and follow this [StackOverflow answer](https://stackoverflow.com/questions/31703739/how-to-read-email-using-python-3).

I wanted to use the official Gmail API because I have never used them before. If you need to make thousands of API calls, then check out the [Usage Limits page](https://developers.google.com/gmail/api/v1/reference/quota). In my case, I needed to make a single API call each day so it’s free.

Follow the instructions in the [Python Quickstart](https://developers.google.com/gmail/api/quickstart/python) page to create a Google Clouds Platform project and download the credentials file. Create the skeleton of the Quickstart as mentioned in the documentation.

After that use the code below to read the excel sheet.


{% highlight python %}
import base64
import os
import pandas as pd
import xlrd
import pickle
import io
import pymysql
import logging
from googleapiclient.discovery import build
from google_auth_oauthlib.flow import InstalledAppFlow
from google.auth.transport.requests import Request
from datetime import datetime, timedelta


SCOPES = ['https://www.googleapis.com/auth/gmail.readonly']


def attach_reader(df):
    """
    To read the excel sheet and extract the data
    """
    client_names = df['Name'].values
    client_age= df['Age'].values
    client_phone = df['client_phone'].values

    return client_names, client_age, client_phone


def email_parser():
    """
    To get the excel sheet in the attachment
    """
    creds = None

    # The file token.pickle stores the user's access and refresh tokens,
    # and is created automatically when the authorization flow completes
    # for the first time.
    if os.path.exists('token.pickle'):
        with open('token.pickle', 'rb') as token:
            creds = pickle.load(token)

    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(
                    'credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)

            # Save credentials for the next run
            with open('token.pickle', 'wb') as token:
                pickle.dump(creds, token)

    service = build('gmail', 'v1', credentials=creds)

    # Getting all emails in the label 'Label_id' by giving the label id
    label_ids=['Label_id']
    response = service.users().messages().list(userId='me', labelIds=label_ids).execute()

    messages = []
    if 'messages' in response:
        messages.extend(response['messages'])

    # Getting the latest email
    message_id = messages[0]['id']
    message = service.users().messages().get(userId='me', id=message_id).execute()

    # to get the attachment ID
    att_id = message['payload']['parts'][-1]['body']['attachmentId']

    attachments = service.users().messages().attachments().get(userId='me', messageId=message_id, id=att_id).execute()

    # Getting the encrypted data of the attachment body
    data = attachments['data']
    data = base64.urlsafe_b64decode(data.encode('UTF-8'))

    # Decoding the decrypted excel data as a bytes string
    decrypted = data
    toread = io.BytesIO()
    toread.write(decrypted)

    # We are passing '1' as the second argument because we need to read the second sheet
    # Then we store the data from the sheet into a Pandas DataFrame
    df = pd.read_excel(toread, 1)

    # Reading the data
    names, ages, phones = attach_reader(df)


if __name__ == '__main__':

    email_parser()
{% endhighlight %}
