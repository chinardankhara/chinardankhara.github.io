---
layout: page
title: EvoCal
description: Let AI find and schedule your calendar events
img: assets/img/evocal.png
importance: 1
category: 
related_publications: 
---

This was a joint project by me, [Harsh Muriki](https://www.linkedin.com/in/venkata-harsh-muriki/) and [Shriya Edukulla](https://www.linkedin.com/in/shriyaedukulla/) for the AI ATL hackathon organised by Startup Exchange. We noticed that many emails have event information - whether that is formal like a TA mixer at a certain time and place or a casual lunch between club members. Most emails don't attach event invites either due to inertia or because there are too many events in the email to attach invites for all of them. We wanted to create a tool that would automatically detect events in emails and add them to your calendar. 

We used the GMail and Google Calendar APIs to read and write events automatically. Emails were fetched using a timer trigger every hour and plain text was extracted using a BeautifulSoup parser. We used OpenAI's GPT-4 Turbo API to classify the emails as "eventful" or not. We experimented with various models but found only GPT-4 to cross the 95% accuracy threshold we wanted. For the eventful emails, we used GPT-3.5 Turbo with JSON output grammar decoder to get structured output for events details and directly write to the calendar. 

We hosted our frontend on NextJS and all our backend services use a modular architecture with an access point using Flask API that runs on Google Cloud Run. This is because our project is a work in progress and we plan to add integrations for Outlook and iCloud (plus a cleaner UI that exists on Heroku) and a modular architecture is easily extensible. The project is open source and can be found at [EvoCal Github](https://github.com/harshmuriki/EvoCal/tree/main). Below is an abridged example of one of our backend services.

```python

def create_calendar_event(service, event_data):

    for email in event_data:
        for event_ in email:
            event = {
                'summary': event_['name'],
                'location': event_['location'],
                'start': {
                    'dateTime': event_['start_time'],
                    'timeZone': 'EST',
                },
                'end': {
                    'dateTime': event_['end_time'],
                    'timeZone': 'EST',
                },
            }
            event = service.events().insert(calendarId='primary', body=event).execute()


app = Flask(__name__)


@app.route('/calendar_invite', methods=['POST'])
def calendar_data():
    try:
        token = request.json['data']
        data = request.json['body']
        assert isinstance(token, str)
        service = get_calendar_service(token)
        create_calendar_event(service, event_data=data)
        return jsonify({"True"})
    except Exception as e:
        return jsonify({'error': str(e)})


if __name__ == '__main__':
    app.run(port=5002)
    pass

```